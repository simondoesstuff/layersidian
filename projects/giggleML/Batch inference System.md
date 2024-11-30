# Justification of in-house solution
PyTorch has utilities for GPU management and simple utilities for basic distributed setups, but it is not sufficient out of the box for very large scale operations.
- Multiple batches across multiple GPUs is difficult to do efficiently with DDP. Custom parallelization necessary.
	- Particularly if data pre-processing can also be parallelized -- it can be easily implemented seamlessly.
	- Parallelization without frequent locks requires minding
- Large jobs require buffered IO; best with memmap.
	- Also there's nuance in moving data between the root process and the workers. Must be picklable. If there's one-time initialization (such as with a fasta database), it may be optimal to redundantly re-initialize on each worker.
	- Keep `ulimit`s in mind.
	- Predictive pre-loading into the input buffer before the GPU calls for next batch is ideal
- Variable interval sizes
	- require pre-processing before tokenization to discern the global max size.
	- may require swapping out the embedding model to suit lengths.
- Custom distributed sampler
	- Possibly avoidable? I have a single source of truth input file. This needs to be batched and divided into the workers ideally without overlap. The outputs then must be aggregated in original order.
- 32-bit can't index some very large tensors despite being able to (easily) hold the tensor in memory
- Long computation often needs safe intermediate cancellation (& continuation)
- Python (sub 3.13) lacks competent parallelization tooling
- Fiji runs old linux and security makes some devOps extremely cumbersome (often)
# Next
**Major** IO bottleneck? CPU always active during inference. GPU spikes. GPU memory utilization good. It appears to finish writing to a mem-mapped file because the size stabilizes. It then hangs (for potentially minutes at 70GB GPU mem) before ultimately aggregating to the final file. While hanging, there is no obvious change in (any) file sizes. After, final aggregation happens on the root process almost instantly. At least 90% of compute time is caught up in the bottleneck.