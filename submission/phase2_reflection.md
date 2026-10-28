# Phase 2 — Reflection: where does parallelism stop helping?

Copy this file to `submission/phase2_reflection.md`, fill in every answer slot, and commit it:

```bash
cp phase-2-tiny-gpt/reflection_template.md submission/phase2_reflection.md
```

**How to fill it in.** Each answer sits between a pair of `<!--answer:...-->` markers. Replace
the `TODO` line with your answer — **leave the markers themselves untouched**, they are how the
grader finds your answers. Anything you write outside the markers is ignored, so add extra
prose, tables or your threading plot freely.

Numbers in section 1 must **match `submission/phase2_report.json`** — copy them across, do not
retype from memory; they are checked against it.

---

## 1. Your measurements

| | Value |
|---|---|
| Naive attention, ms (`attention.naive_ms`) | <!--answer:attn_naive_ms-->702.123<!--/answer--> |
| Vectorized attention, ms (`attention.vectorized_ms`) | <!--answer:attn_vectorized_ms-->1.093<!--/answer--> |
| CPU training, ms for 100 steps (`devices.cpu_ms`) | <!--answer:cpu_ms-->447124.9<!--/answer--> |
| GPU training, ms for 100 steps (`devices.gpu_ms`) | <!--answer:gpu_ms-->12762.5<!--/answer--> |
| Final training loss (`training.final_loss`) | <!--answer:final_loss-->1.4905<!--/answer--> |

**The thread sweep.** Give your four timings from `threads` — 1, 2, 4 and 8 threads — and say
which count was fastest. (verbatim: the numbers, in order)

<!--answer:thread_sweep-->
1 thread: 2359.631 ms
2 threads: 2244.31 ms
4 threads: 2608.841 ms
8 threads: 2498.574 ms
The fastest count was 2 threads.
<!--/answer-->

---

## 2. Intra-node parallelism, three ways

**Why vectorizing wins.** Both attention implementations do the same arithmetic and produce the
same result (the unit test checks that). Explain where the naive version's time actually goes,
and what the vectorized one does differently — in terms of the machine, not the Python.
(~70 words)

<!--answer:vectorization_why-->
The naive version takes longer because the processor reads, decodes and executes each arithmetic instruction sequentially, one by one. The vectorised version utilises SIMD. Instead of processing elements one by one, the processor loads an entire array of data into very wide hardware registers and executes a single mathematical instruction simultaneously across the whole block. Vectorisation is more efficient because it replaces repetitive loops with direct hardware parallelism, as we have seen in class.
<!--/answer-->

**Where the thread curve flattens, and why.** Using your sweep: more threads stopped helping —
or started hurting — at some point. Say where, and give the reason. Your Colab runtime's vCPU
count is part of the answer; so is what the threads have to coordinate on. (~80 words)

<!--answer:threading_plateau-->
The curve levelled off and then deteriorated beyond 2 threads, as the Google Colab environment only has 2 virtual cores. The architecture of multi-core processors divides the physical processor but requires threads to use shared memory. When more threads are launched than there are physical cores, the threads must coordinate to access the limited hardware resources. This coordination, particularly where there are shared variables, results in a massive software overhead. As illustrated in the course, this contention overhead can make multithreaded execution slower than serial execution.
<!--/answer-->

**What made the GPU faster.** The model, the data and the step count were identical; only the
device changed. Say what about a transformer training step suits a GPU, and name one workload
that would *not* see this speed-up. (~70 words)

<!--answer:gpu_why-->
Training a Transformer model relies on massive matrix multiplications. The GPU is faster at this because its architecture is different. Whilst the CPU devotes a large proportion of its transistors to caches and complex logic, the GPU uses its transistors to align thousands of simple computing cores. The GPU uses the SIMT model, executing thousands of threads in a synchronised manner on independent data.
A strictly sequential task, for example, would not benefit from any acceleration on a GPU.
<!--/answer-->

---

## 3. The question

**Where does parallelism stop helping, and why?** Pull the three experiments together. Your
answer should distinguish the *kinds* of limit you met — work that cannot be divided, hardware
you do not have, coordination that costs more than it saves, data that has to move — and say
which one bound each experiment. One concrete number from section 1 per claim. (~150 words)

<!--answer:parallelism_limit-->
Parallelism ceases to be useful when the costs of coordination exceed the computational gains, or when faced with hardware limitations.
Firstly, in the case of indivisible work: the SIMD experiment bypasses sequential, instruction-by-instruction processing, reducing the time from 702.123 ms to 1.093 ms. However, the final text generation remains sequential and indivisible.
Secondly, coordination costs more than it yields when hardware resources are limited: using 4 threads (2,608.841 ms) instead of 2 (2,244.31 ms) on a constrained CPU slows down execution due to access conflicts and the mandatory synchronisation associated with shared memory and mutexes.
Finally, regarding the data that needs to be moved: although the GPU performs better than the CPU thanks to its multiple cores suited to matrix multiplication, it is ultimately limited by the slow transfer of data between the host’s memory and the device’s memory.
<!--/answer-->

**What you would measure next.** One experiment this phase did not run that would sharpen the
answer, and what result would change your mind. (~50 words)

<!--answer:next_experiment-->
I would measure the exact time taken for the memory transfer between the host and the GPU. If this data transfer takes longer than the matrix calculation on the GPU, this would prove that memory bandwidth is the real bottleneck, not the computation itself.
<!--/answer-->
