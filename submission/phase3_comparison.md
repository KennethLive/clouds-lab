# Phase 3 — Comparison Writeup

Copy this file to `submission/phase3_comparison.md`, fill in every answer slot, and commit it:

```bash
cp phase-3-mapreduce-spark/comparison_template.md submission/phase3_comparison.md
```

**How to fill it in.** Each answer sits between a pair of `<!--answer:...-->` markers. Replace
the `TODO` line with your answer — **leave the markers themselves untouched**, they are how the
grader finds your answers. Anything you write outside the markers is ignored, so add extra
prose, tables or images freely.

Numbers in section 1 must **match `submission/phase3_report.json`** — copy them across, do not
retype from memory; they are checked against it.

---

## 1. Facts from your runs

Numbers only (e.g. `12`, `20.3`, `0.041`).

| | Value |
|---|---|
| Map tasks in the cloud job (`mapreduce.num_splits`) | <!--answer:num_map_tasks-->12<!--/answer--> |
| Reduce tasks in the cloud job (`mapreduce.num_reducers`) | <!--answer:num_reduce_tasks-->4<!--/answer--> |
| Cloud MapReduce wall time, seconds (`mapreduce.cloud_elapsed_s`) | <!--answer:cloud_elapsed_s-->38.6<!--/answer--> |
| Local single-process wall time, seconds (`mapreduce.local_elapsed_s`) | <!--answer:local_elapsed_s-->0.022<!--/answer--> |
| Spark wall time for the `collect`, seconds (`tfidf.spark_elapsed_s`) | <!--answer:spark_elapsed_s-->3.7<!--/answer--> |
| Number of **stages** in the Spark job (read off notebook 2b / 3a) | <!--answer:num_stages-->4<!--/answer--> |
| Task **retries** you counted in your `--chaos` run (0 if none happened) | <!--answer:chaos_retries-->4<!--/answer--> |

**Explain the timings.** The three wall times above cover the same 60 documents, but not the
same job: the single-process run and the cloud MapReduce are *the same word count*; Spark's
time is for *TF-IDF*, which is more work (two `reduceByKey`s and a `join`). Rank the three and
explain what each is spending its time on. Say which comparison is like-for-like, and what the
Spark number — a heavier job, on one machine — tells you about where the cloud job's time goes.
(~80 words)

<!--answer:timing_explanation-->
The local script (0.022s), Spark TF-IDF (3.7s), followed by cloud MapReduce (38.6s). Only the local and cloud runs are comparable for the same workload (same word count). The local script runs Python entirely in memory. Spark performs a more computationally intensive TF-IDF calculation on a single machine, whilst still being more than 10 times faster than the cloud. This shows that the cloud job’s execution time is dominated by the overhead associated with distributed processing: HTTP/OIDC calls, Cloud Run cold starts, GCS I/O and retries.
<!--/answer-->

**What the counts tell you.** From your stage count: how many shuffles did the Spark job have,
and which operations in `spark_tfidf.py` caused them? From your retry count: roughly what
fraction of the 16 task attempts was retried, and is that what `--chaos 0.3` should produce?
(~50 words)

<!--answer:counts_interpretation-->
Four stages correspond to three shuffle boundaries in spark_tfidf.py, caused by `reduceByKey` in `term_freq`, `reduceByKey` in `doc_freq` and `join` in `build_tfidf_rdd'. During the chaos run, there were 4 retries out of 16 tasks (i.e. 25%), which is consistent with --chaos 0.3 (approximately 30% expected failures).
<!--/answer-->

---

## 2. Spark: transformations, actions, lineage

**Transformations vs actions.** Go through `spark_tfidf.py` and the notebook's Spark cells.
Name each RDD operation and say whether it is a transformation or an action. What happened
when you ran `build_tfidf_rdd` — and what happened when you ran `collect`? (~60 words)

<!--answer:transformations_vs_actions-->
In spark_tfidf.py, `flatMap`, `map`, `reduceByKey` and `join` are transformations (they do not perform any calculations). Only `collect()` is an action. When I ran `build_tfidf_rdd`, Spark did not perform any calculations: it simply prepared the execution plan. It was the call to `collect()` that actually triggered the calculations and returned the 5,975 results.
<!--/answer-->

**Lineage and recovery.** Using the lineage you printed in 2b and the timeline in 3a: if the
worker holding one output partition of the *first* `reduceByKey` died, what would Spark have
to recompute, and what could it reuse? Name the stage or shuffle boundary where the recovery
starts. (~50 words)

<!--answer:lineage_recovery-->
If a worker loses a partition from the first `reduceByKey` (ShuffledRDD[3]), Spark recalculates only that lost partition. To reconstruct it, it starts again from the initial data (ParallelCollectionRDD[0], Stage 0) and re-runs the `flatMap` up to that first shuffle boundary (Stage 1). All other intact partitions are reused directly.
<!--/answer-->

---

## 3. Your cloud MapReduce, mapped onto Hadoop

Name the GCP service or file that played each role (one line each):

- **JobTracker** (schedules tasks, waits for the map phase, retries failures):
<!--answer:role_jobtracker-->Cloud Workflows<!--/answer-->
- **Workers** (run your map and reduce code):
  <!--answer:role_workers-->Cloud RUn functions<!--/answer-->
- **HDFS** (holds inputs, shuffle files, outputs):
  <!--answer:role_hdfs-->Cloud Storage<!--/answer-->
- **Partitioner** (decides which reducer owns a key):
  <!--answer:role_partitioner-->partition() function<!--/answer-->

**The barrier.** There is no explicit "wait for all mappers" step in `workflow.yaml`. Where,
exactly, is the barrier between the map and reduce phases — which construct provides it?
(~30 words)

<!--answer:barrier_location-->
The boundary between `map_phase` and `reduce_phase` is implicit: in `workflow.yaml`, the `parallel` block automatically waits for all Map tasks to complete before moving on to the next stage.
<!--/answer-->

**Idempotence.** The workflow retries a task that failed. Why is that safe with the worker as
written, and what would go wrong if `run_map` *appended* its pairs to the shuffle file instead
of overwriting it? (~60 words)

<!--answer:idempotence-->
Rerunning a task is safe because each *worker* overwrites its *shuffle* (or output) file in Cloud Storage: running the same task twice therefore produces exactly the same result. If `run_map` appended the data instead of overwriting it, a task that failed and was then rerun would write duplicate words, which would skew the final count.
<!--/answer-->

**The chaos run.** What did you see in the execution log and in Logs Explorer when you ran
with `--chaos 0.3`? Did the job still succeed, and was the final answer still identical to the
local reference? (~50 words)

<!--answer:chaos_observations-->
With `--chaos 0.3`, I saw four HTTP `503` errors (the simulated crashes) on `mr-worker` in Logs Explorer, which were automatically retried by Cloud Workflows. Despite these failures, the job was successful (`SUCCEEDED`) and the final result was exactly the same as the local reference (`identical: True`, 2,221 words).
<!--/answer-->
