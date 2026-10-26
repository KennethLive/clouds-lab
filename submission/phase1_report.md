# Phase 1 — Comparison Report

Copy this file to `submission/phase1_report.md`, fill in every answer slot, and commit it:

```bash
cp phase-1-echo-bot/report_template.md submission/phase1_report.md
```

**How to fill it in.** Each answer sits between a pair of `<!--answer:...-->` markers. Replace
the `TODO` line with your answer — **leave the markers themselves untouched**, they are how the
grader finds your answers. Anything you write outside the markers is ignored, so add extra
prose, tables or images freely.

Numbers must **match `submission/phase1_report.json`** exactly — that file is produced by
`measure.py` and is what your figures are checked against. Copy them across; do not retype from
memory.

---

## 1. Setup facts

**Your image size**, in MB, from `gcloud artifacts docker images list` (number only, e.g. `142`):

<!--answer:image_size_mb-->
48
<!--/answer-->

**The custom VPC and subnet you created** (names only):

<!--answer:vpc_name-->
echo-net
<!--/answer-->

<!--answer:subnet_name-->
echo-subnet
<!--/answer-->

---

## 2. Measurements

Copy these from `submission/phase1_report.json`. Cold is `cold_ms`; warm median is
`stats.median`. Numbers only, in milliseconds (e.g. `812.4`).

| Platform | Cold (ms) | Warm median (ms) |
|---|---|---|
| IaaS VM | <!--answer:vm_cold_ms-->251.8<!--/answer--> | <!--answer:vm_warm_median_ms-->210.8<!--/answer--> |
| Cloud Run | <!--answer:cloudrun_cold_ms-->2264.7<!--/answer--> | <!--answer:cloudrun_warm_median_ms-->633.4<!--/answer--> |
| Cloud Function | <!--answer:function_cold_ms-->1678.4<!--/answer--> | <!--answer:function_warm_median_ms-->642.5<!--/answer--> |

Include your histogram or plot of the warm distribution below (an image, or the ASCII chart
`measure.py` prints). This is read by a human, not the grader:

<!--answer:latency_plot-->
[OK  ] vm        cold=  251.8ms  warm_median=  210.8ms
            #############
[OK  ] cloudrun  cold= 2264.7ms  warm_median=  633.4ms
            #######################################
[OK  ] function  cold= 1678.4ms  warm_median=  642.5ms
            ########################################
<!--/answer-->

---

## 3. Per-platform analysis

**Scaling behaviour.** Imagine requests arriving one at a time, then ten at once, then a
hundred. For each of the three platforms, answer concretely:

- Does anything get created automatically to absorb the load — and if so, *what* exactly?
  (One platform here creates nothing at all.)
- What is the hard limit, and where does it come from — a flag you passed yourself, or the
  size of the machine you picked?

(~80 words. Your `--machine-type=e2-micro` and `--max-instances=3` are both concrete answers
to the second question.)

<!--answer:scaling_behaviour-->
To handle the load  for example, in the event of a DDoS attack, as we saw in class and given that the virtual machine is small (e2-micro), it will not automatically create anything to counter the problem, the limit is set by the VM’s resources. As for Cloud Run, Google will automatically clone the container with the imposed limit (max-instances=3 flag). Finally, Cloud Functions behaves in the same way as Cloud Run, using Google’s default limits.
<!--/answer-->

**Deployment effort.** What did you have to build, configure and maintain to get each of the
three running? (~100 words)

<!--answer:deployment_effort-->
In order of difficulty, the VM comes first because we had to set up the network manually, open the ports, connect via SSH and install Docker, this therefore requires some system administration work beforehand. Next, Cloud Run was simpler than the VM: a single command to deploy and make it public. The administrative work is invisible, but we still had to build the Docker image. Finally, Cloud Functions handled almost everything on its own (Docker, container, etc.), requiring just a single command to specify the directory containing the Python code.
<!--/answer-->

---

## 4. Why do two of your three numbers look alike?

With Cloud Run and your Function both deployed, run `gcloud run services list` and paste the
**complete output** here, unedited:

<!--answer:run_services_list-->
✔
SERVICE: echo
REGION: us-central1
URL: https://echo-956842128527.us-central1.run.app
LAST DEPLOYED BY: service-956842128527@gcf-admin-robot.iam.gserviceaccount.com
LAST DEPLOYED AT: 2026-10-26T10:48:20.754815Z

✔
SERVICE: echo-bot
REGION: us-central1
URL: https://echo-bot-956842128527.us-central1.run.app
LAST DEPLOYED BY: Kenneth.Live-Hip-Kai@eurecom.fr
LAST DEPLOYED AT: 2026-10-26T10:46:05.345009Z
<!--/answer-->

Now explain what that output tells you about how your Function is actually being run, why its
cold start resembles Cloud Run's so closely, and what — if anything — genuinely differs between
the two deployment paths. (~120 words)

<!--answer:convergence_explanation-->
The output of the command shows that our Cloud Function appears directly in the list of Cloud Run services. This reveals that second generation Cloud Functions actually run on the Cloud Run infrastructure. This is why their cold start times are so similar, both platforms use exactly the same technology and container environment. The only real difference lies in the deployment experience. With Cloud Run, we need to write a Dockerfile, build the image and publish it on our own. With Cloud Functions, we simply need to provide the raw source code, and Google builds the container automatically in the background.
<!--/answer-->
