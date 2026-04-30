---
layout: distill
title: "Tensor Parallel + Sequence Parallel — A Deep Dive"
permalink: /blog/tensor-parallel
date: 2026-04-30 15:00 +0200

authors:
  - name: Guanxiong Luo
    url: "https://ggluo.github.io/"

toc:
  - name: Overview
    subsections:
      - name: "Column Parallel and Row Parallel"
      - name: "From Tensor Parallel to Sequence Parallel"
      - name: "Advantages and Limitations"
  - name: Core Implementation
    subsections:
      - name: "Communication Primitives"
      - name: "ColumnParallelLinear"
      - name: "RowParallelLinear"
      - name: "Weight Initialization"
      - name: "Transformer Integration"
      - name: "Training Data Flow"
  - name: "Communication-Computation Overlap"
    subsections:
      - name: "AllGather + GEMM Pipeline"
      - name: "GEMM + ReduceScatter Pipeline"
      - name: "Per-Layer Overlap Strategy"
  - name: Extended Thoughts
  - name: Code Walkthrough
    subsections:
      - name: "TP Linear Layers"
      - name: "Transformer Model"
      - name: "Overlap Implementation"
      - name: "Testing and Profiling"
  - name: Summary
---

## Overview

**Tensor Parallelism (TP)** is one of the core model-parallel strategies for large-scale distributed training. It shards individual module weights across multiple GPUs — each GPU holds a slice of the weight matrix, computes locally, and the partial results are merged through collective communication. The merged result is mathematically equivalent to what a single GPU would produce with the full weights.

TP operates at the module level and targets **Linear layers** — the most parameter-heavy components in a Transformer. This post walks through the full picture: Column/Row Parallel, Sequence Parallel, communication-computation overlap, and a complete toy implementation. The code is open-sourced at [tensor_parallel_toy](https://github.com/ggluo/tensor_parallel_toy), built on the original work by [@xiabingquan](https://github.com/xiabingquan/tensor_parallel).

**Prerequisites**: PyTorch, NCCL collective basics (AllReduce, AllGather, ReduceScatter).

### Notation

| Symbol | Meaning |
| --- | --- |
| $$n$$ | TP degree — number of GPUs in the tensor-parallel group |
| $$s$$ | Sequence length |
| $$b$$ | Batch size |
| $$h$$ | Hidden size |
| $$h'$$ | Linear layer output dimension (may differ from $$h$$, e.g. MLP intermediate dim) |
| $$L$$ | Number of Transformer layers |
| $$nh$$ | Number of attention heads |

Linear layer computation is written as $$Y = X A^T$$ where weight $$A$$ has shape $$[h', h]$$ (PyTorch convention: `[out_features, in_features]`).

### Column Parallel and Row Parallel

TP offers two ways to shard a Linear layer, symmetric in their communication patterns.

**Column Parallel** shards weight $$A$$ along dimension 0 (the output dimension). Each GPU holds $$A_i$$ of shape $$[h'/n, h]$$ and computes $$Y_i = X A_i^T$$ locally. The forward pass needs no communication. However, during backward, computing $$dX$$ requires an AllReduce across the partial $$dX_i$$ from each rank.

| Phase | Weight shape | Input shape | Output shape | Communication |
| --- | --- | --- | --- | --- |
| Forward | $$[h'/n, h]$$ | $$[s, b, h]$$ | $$[s, b, h'/n]$$ | None |
| Backward (dX) | $$[h'/n, h]$$ | grad: $$[s, b, h'/n]$$ | $$[s, b, h]$$ | AllReduce |
| Backward (dW) | — | grad × saved input | $$[h'/n, h]$$ | None |

**Row Parallel** shards along dimension 1 (the input dimension). Each GPU holds $$A_i$$ of shape $$[h', h/n]$$, and the input is correspondingly split to $$X_i$$ of shape $$[s, b, h/n]$$. Forward requires an AllReduce of partial sums. Backward produces $$dX$$ naturally sharded — no communication needed.

| Phase | Weight shape | Input shape | Output shape | Communication |
| --- | --- | --- | --- | --- |
| Forward | $$[h', h/n]$$ | $$[s, b, h/n]$$ | $$[s, b, h']$$ | AllReduce |
| Backward (dX) | $$[h', h/n]$$ | grad: $$[s, b, h']$$ | $$[s, b, h/n]$$ | None |
| Backward (dW) | — | grad × saved input | $$[h', h/n]$$ | None |

**Paired usage**: Column and Row Parallel have complementary communication patterns — one communicates in forward, the other in backward. In Transformers they always appear in pairs: QKV projections (Column) paired with output projection (Row) in Attention; FC1 (Column) paired with FC2 (Row) in the MLP. A Column Parallel output of shape $$[s, b, h'/n]$$ feeds directly into a downstream Row Parallel input of shape $$[s, b, h/n]$$ with no intermediate communication.

### From Tensor Parallel to Sequence Parallel

Column + Row Parallel eliminates redundant communication for Linear layers, but non-TP modules like RMSNorm and Dropout still operate on full $$[s, b, h]$$ tensors replicated on every rank — wasting both activation memory and compute.

**Sequence Parallel (SP)** addresses this: in non-TP regions, the tensor is sharded along the sequence dimension into $$[s/n, b, h]$$, so each rank only handles a portion of the sequence. Communication patterns are adjusted: entering a TP region requires an AllGather to reconstruct the full $$[s, b, h]$$; leaving a TP region uses ReduceScatter to sum-and-distribute back to $$[s/n, b, h]$$.

| Aspect | TP only | TP + SP |
| --- | --- | --- |
| Non-TP tensor shape | $$[s, b, h]$$ (replicated) | $$[s/n, b, h]$$ (unique per rank) |
| Non-TP computation | Redundant on all ranks | $$1/n$$ per rank |
| Non-TP activation memory | Full on every rank | $$1/n$$ per rank |
| Forward comm per TP pair | 1× AllReduce | 1× AllGather + 1× ReduceScatter |
| Total communication | $$2N(n-1)/n$$ | $$2N(n-1)/n$$ (same) |

Total communication volume is unchanged — because AllReduce itself decomposes into ReduceScatter + AllGather. SP's gain is purely in activation memory and compute, cutting both to $$1/n$$ with zero extra communication cost. SP is effectively always-on in practice.

### Advantages and Limitations

**Advantages**:
- **Zero redundancy**: weights are strictly partitioned $$1/n$$, with $$1/n$$ compute per rank. With SP, activations and non-TP compute are equally sharded.
- **Mathematical equivalence**: local results after communication are identical to single-GPU full-weight computation — no approximation error.
- **Deployment locality**: TP typically runs within a node over NVLink, where high bandwidth keeps communication overhead manageable.

**Limitations**:
- TP degree is constrained by hidden size divisibility — typically 2, 4, or 8.
- Each TP Linear pair requires two collectives per forward pass; cross-node deployment makes latency a bottleneck.
- Larger TP degree shrinks per-GPU GEMM size, reducing GPU compute efficiency.

## Core Implementation

### Communication Primitives

TP uses two core primitives, both operating along the sequence dimension (dim 0):

| Primitive | Behavior | Input shape | Output shape | Usage |
| --- | --- | --- | --- | --- |
| AllGather | Collect and concatenate shards | $$[s/n, b, h]$$ | $$[s, b, h]$$ | ColumnParallel forward / RowParallel backward |
| ReduceScatter | Sum full tensors, then scatter | $$[s, b, h]$$ | $$[s/n, b, h]$$ | RowParallel forward / ColumnParallel backward |

These two form a conjugate pair in autograd: AllGather in forward corresponds to ReduceScatter in backward, and vice versa. Understanding this conjugacy is key to implementing TP backpropagation correctly.

### ColumnParallelLinear

Each rank holds weight $$[h'/n, h]$$. The full forward-backward flow:

| Phase | Operation | Input shape | Output shape | Communication |
| --- | --- | --- | --- | --- |
| Forward | AllGather → GEMM | $$[s/n, b, h]$$ | $$[s, b, h'/n]$$ | AllGather |
| Backward (dX) | GEMM → ReduceScatter | grad: $$[s, b, h'/n]$$ | $$[s/n, b, h]$$ | ReduceScatter |
| Backward (dW) | GEMM (with saved full input) | grad × saved input | $$[h'/n, h]$$ | None |

Forward: AllGather reconstructs full input $$[s, b, h]$$, then local GEMM produces $$[s, b, h'/n]$$. The backward saves the full gathered input on the forward pass to reuse when computing dW, avoiding redundant communication.

### RowParallelLinear

Each rank holds weight $$[h, h/n]$$:

| Phase | Operation | Input shape | Output shape | Communication |
| --- | --- | --- | --- | --- |
| Forward | GEMM → ReduceScatter | $$[s, b, h/n]$$ | $$[s/n, b, h]$$ | ReduceScatter |
| Backward (dY) | AllGather → GEMM | grad: $$[s/n, b, h]$$ | $$[s, b, h/n]$$ | AllGather |
| Backward (dW) | GEMM (with AllGathered grad) | grad × saved input | $$[h, h/n]$$ | None |

Forward: local GEMM produces a partial sum $$[s, b, h]$$, then ReduceScatter sums across ranks and scatters to $$[s/n, b, h]$$. Backward: AllGather recovers full grad_output before computing dX and dW.

### Weight Initialization

TP weight init must guarantee that sharded weights, when concatenated, are equivalent to initializing a full weight matrix with the same method. Two approaches:

- **CPU init** (used here): all ranks share the same global RNG seed, generate the full weight matrix once, then slice out their shard. Simple and reliable, but the full matrix must fit in CPU memory. Bit-exact with non-TP init.
- **GPU init**: each rank initializes its shard directly on GPU with separate RNG state. Requires careful RNG isolation between TP and non-TP parameters. Cannot be bit-exact with non-TP init.

### Transformer Integration

A Transformer layer contains Attention and MLP, each using one Column + Row Parallel pair:

- **Attention**: Q, K, V projections use ColumnParallel (each rank handles $$nh/n$$ heads); output projection uses RowParallel.
- **MLP**: FC1 uses ColumnParallel (output dim $$h'/n$$); FC2 uses RowParallel.
- **RMSNorm, Dropout**: in SP region, operate on $$[s/n, b, h]$$ tensors. RMSNorm weights are replicated; after backward their gradients must be AllReduced since each rank only saw $$s/n$$ positions.

Each Transformer layer involves 4 communication calls in forward (2 AllGather + 2 ReduceScatter) and symmetrically 4 in backward.

### Training Data Flow

From a global TP + SP perspective, here is how a token's shape evolves through one Transformer layer on each rank:

```
Input: [s/n, b, h]                    (SP shard along sequence)
  │
  ▼
RMSNorm: [s/n, b, h]                  (SP region, replicated weights)
  │
  ▼
QKV ColumnParallel:                   (TP region)
  AllGather → [s, b, h] → GEMM → [s, b, h/n] (each rank: nh/n heads)
  │
  ▼
Attention (per head): [b, nh/n, s, d] → [b, nh/n, s, d]
  │
  ▼
Out RowParallel:                      (TP region)
  GEMM → partial [s, b, h] → ReduceScatter → [s/n, b, h]
  │
  ▼
RMSNorm: [s/n, b, h]                  (SP region)
  │
  ▼
FC1 ColumnParallel:
  AllGather → [s, b, h] → GEMM → [s, b, h'/n]
  │
  ▼
SiLU: [s, b, h'/n]
  │
  ▼
FC2 RowParallel:
  GEMM → partial [s, b, h] → ReduceScatter → [s/n, b, h]
  │
  ▼
Output: [s/n, b, h]                   (SP shard)
```

## Communication-Computation Overlap

The basic TP implementation runs communication and computation serially: first a full AllGather, then a full GEMM. During AllGather the compute units are idle; during GEMM the communication links are idle. Overlap pipelines these by splitting collectives into chunks and interleaving them with computation on separate CUDA streams.

### AllGather + GEMM Pipeline

ColumnParallel forward requires AllGather followed by GEMM. The overlap strategy decomposes AllGather into $$n$$ rounds of ring P2P exchange — each received chunk immediately launches a GEMM:

Without overlap:
```
AllGather(all)              GEMM(all)
|===========================|===========================|
```

With overlap (n=4):
```
  Recv chunk0   Recv chunk1   Recv chunk2   Recv chunk3
  |============|============|============|
               |============|============|============|
                GEMM c0      GEMM c1      GEMM c2      GEMM c3
```

Implementation: each rank starts with its local shard as the first chunk (GEMM immediately). Simultaneously, it sends its local chunk to the next rank via P2P and receives from the previous. Each received chunk triggers a GEMM and is forwarded to the next rank. After $$n-1$$ rounds, GEMM outputs are concatenated in order. Communication latency is hidden behind GEMM execution time.

### GEMM + ReduceScatter Pipeline

RowParallel forward (and ColumnParallel backward) suffer from serial GEMM then ReduceScatter. The overlap approach: split the GEMM output into chunks along the sequence dimension, and launch per-chunk ReduceScatter as each chunk's GEMM completes:

Without overlap:
```
GEMM(all)                   ReduceScatter(all)
|===========================|===========================|
```

With overlap (4 chunks):
```
  GEMM c0      GEMM c1      GEMM c2      GEMM c3
  |==========|==========|==========|
             |==========|==========|==========|
              RS c0       RS c1       RS c2       RS c3
```

Implementation: split input into `num_chunks` along dim 0. For each chunk, GEMM produces a partial result, then `dist.reduce(dst=i)` sends the sum to rank `i` on a separate communication stream. Unlike the AllGather case, chunk count is not limited by TP degree — it can be tuned to balance pipeline granularity against scheduling overhead.

### Per-Layer Overlap Strategy

| Layer | Forward | Backward (dX) | Backward (dW) |
| --- | --- | --- | --- |
| QKV (ColumnParallel) | AG + GEMM overlap | GEMM + RS overlap | No overlap |
| Output (RowParallel) | GEMM + RS overlap | AG + GEMM overlap | No overlap |
| FC1 (ColumnParallel) | AG + GEMM overlap | GEMM + RS overlap | No overlap |
| FC2 (RowParallel) | GEMM + RS overlap | AG + GEMM overlap | No overlap |

dW never overlaps — it uses the forward-saved full input directly, no communication involved.

## Extended Thoughts

**AllReduce vs AllGather + ReduceScatter**: with SP, an AllReduce is replaced by AllGather + ReduceScatter. But ring AllReduce itself decomposes into one ReduceScatter followed by one AllGather — the total communication volume is identical at $$2N(n-1)/n$$. SP's gain is purely in activation memory, not communication.

**Overlap ceiling**: the benefit of overlap depends on the GEMM-to-communication time ratio. When GEMM dominates, communication is nearly fully hidden. When communication dominates (e.g., slow interconnects), overlap provides limited gain. Within a node over NVLink (~900 GB/s), communication is typically faster than GEMM, making overlap highly effective. In practice, production TP overlap relies on kernel-level implementations (e.g., NVIDIA Transformer Engine) rather than naive PyTorch.

**TP and Context Parallel (Ulysses)**: both TP and Ulysses converge on the same insight for attention — each rank computes only a subset of attention heads. TP achieves this by sharding QKV weights along the head dimension via ColumnParallel; Ulysses uses AlltoAll to redistribute from a sequence-sharded layout to a head-sharded layout, computes attention, then AlltoAll back. In long-sequence training, this head-wise sharding effectively mitigates the $$O(s^2)$$ memory and compute bottleneck of attention.

## Code Walkthrough

The complete implementation is at [tensor_parallel_toy](https://github.com/ggluo/tensor_parallel_toy). Here we walk through the key components.

### TP Linear Layers

The core of TP is `ColumnParallelLinear` and `RowParallelLinear`, each implemented as a custom `autograd.Function` that encapsulates the full forward/backward with communication:

```python
class _ColumnParallelLinearFn(torch.autograd.Function):
    @staticmethod
    def forward(ctx, input_, weight, group):
        # AllGather: [s/n, b, h] -> [s, b, h]
        full_input = _all_gather(input_, group)
        # GEMM: [s, b, h] @ [h'/n, h]^T -> [s, b, h'/n]
        output = F.linear(full_input, weight)
        ctx.save_for_backward(full_input, weight)
        ctx.group = group
        return output

    @staticmethod
    def backward(ctx, grad_output):
        full_input, weight = ctx.saved_tensors
        # dX: GEMM -> ReduceScatter
        grad_input_full = grad_output.matmul(weight)
        grad_input = _reduce_scatter(grad_input_full, ctx.group)
        # dW: reuse forward-saved full_input, no communication
        grad_weight = grad_output.reshape(-1, grad_output.shape[-1]).t().matmul(
            full_input.reshape(-1, full_input.shape[-1])
        )
        return grad_input, grad_weight, None
```

`RowParallelLinear` is symmetric: forward does GEMM then ReduceScatter; backward does AllGather then GEMM. Saving the AllGathered input in forward avoids redundant communication during dW computation.

Weight initialization uses the CPU approach: all ranks share a global RNG seed, draw the full weight matrix from `torch.randn`, then slice out their shard. For non-TP runs, a `PlainLinear` draws from the same RNG in identical order to keep RNG consumption consistent.

### Transformer Model

The model is a standard pre-norm Transformer. `config.use_tp` controls whether TP layers are used:

```python
class Attention(nn.Module):
    def __init__(self, config, layer_idx):
        if config.use_tp:
            self.q_proj = ColumnParallelLinear(h, h, use_overlap=config.use_overlap)
            self.k_proj = ColumnParallelLinear(h, h, use_overlap=config.use_overlap)
            self.v_proj = ColumnParallelLinear(h, h, use_overlap=config.use_overlap)
            self.out_proj = RowParallelLinear(h, h, use_overlap=config.use_overlap)
        else:
            self.q_proj = PlainLinear(h, h)
            ...

    def core_attn(self, q, k, v):
        scores = torch.matmul(q, k.transpose(-2, -1)) * self.scale
        attn_weights = torch.softmax(scores.float(), dim=-1).to(q.dtype)
        return torch.matmul(attn_weights, v)
```

With TP enabled, each rank handles `nh/n` attention heads. RMSNorm weights are replicated across ranks; after backward, `allreduce_replicated_grads()` sums their gradients since each rank only saw `s/n` sequence positions during SP.

### Overlap Implementation

Two overlap modes are implemented on dual CUDA streams:

**AllGather + GEMM** (`_ag_gemm_overlap`): ring P2P exchange with `dist.batch_isend_irecv`. Round 0 computes GEMM on the local chunk immediately. Rounds 1 through $$n-1$$: wait for the next received chunk, launch GEMM on it, then forward it to the next rank. Communication and computation alternate on separate streams with event-based synchronization.

**GEMM + ReduceScatter** (`_gemm_rs_overlap`): splits input into chunks along dim 0. For chunk `i`, GEMM runs on the compute stream, then `dist.reduce(dst=i)` runs on the communication stream. Using `dist.reduce` (not `reduce_scatter`) is deliberate — it scatters at the correct per-rank granularity. All GEMM outputs are kept alive until communication finishes, then freed.

The same `ColumnParallelLinear` / `RowParallelLinear` module classes accept a `use_overlap` flag that switches between basic and overlap autograd Functions internally.

### Testing and Profiling

Tests cover two scenarios:

- **TP vs non-TP**: compares forward outputs. Because ReduceScatter changes floating-point accumulation order (partial matmul + cross-rank sum vs single large matmul), bit-exact matching is not achievable, but forward differences stay within ~$$10^{-4}$$. Gradients are not compared since differences amplify through softmax backward.
- **Overlap vs non-overlap**: computation path is mathematically equivalent; both forward and gradients match at `atol=0, rtol=0`.

The memory profiler compares three configurations (no TP, TP, TP+overlap) across metrics: loss, grad norm, step time, and peak memory. With TP+SP, peak memory is expected to drop to roughly $$1/n$$ of the single-GPU baseline since both weights and activations are sharded.

## Summary

Tensor Parallel is a cornerstone of large-model distributed training. The journey from naive weight sharding to Column+Row paired usage, to Sequence Parallel eliminating non-TP redundancy, to communication-computation overlap squeezing out the last bit of performance — it embodies the engineering philosophy of "make it work, then make it fast."

The core insight is simple: matrix multiplication naturally decomposes along rows or columns. Communication only exists to correctly merge the pieces back together. Once you internalize that, the rest follows.
