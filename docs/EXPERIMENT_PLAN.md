# Experiment Plan

## 1. Research Question

How much idle GPU time can queue-based scaling remove while keeping SLA violations low, given slow GPU pod start-up?

## 2. Experimental Objectives

- Compare fixed-capacity and autoscaling approaches under consistent workloads.
- Measure the effect of queue-based scaling on request latency and backlog.
- Quantify the impact of worker startup delays on scaling performance.
- Evaluate whether the custom scaling policy improves the trade-off between idle capacity and SLA violations.
- Determine how workload patterns affect scaling behavior.
- Produce reproducible results that support the project's conclusions.

## 3. Experimental Hypotheses

These are hypotheses to evaluate, not established results.

- **H1:** Queue-based autoscaling reduces idle capacity compared with fixed over-provisioning.
- **H2:** Fixed under-provisioning produces more latency violations during sufficiently high demand.
- **H3:** Slow worker startup contributes to latency violations during sudden demand increases.
- **H4:** A startup-aware custom policy may reduce SLA violations compared with standard queue-based scaling, potentially at the cost of additional warm capacity.
- **H5:** The effectiveness of each scaling strategy varies with workload intensity and burst duration.

The experiments must be allowed to reject these hypotheses if the measurements do not support them.

## 4. Experimental Baselines

| ID | Strategy | Configuration | Purpose |
|---|---|---|---|
| B1 | Fixed over-provisioning | A predetermined, relatively high worker count | Establish a high-capacity reference |
| B2 | Fixed under-provisioning | A predetermined, relatively low worker count | Establish a constrained-capacity reference |
| B3 | CPU-based HPA | Scale based on CPU utilization | Evaluate a conventional autoscaling approach |
| B4 | KEDA queue-based scaling | Scale according to queue-related metrics | Establish the queue-based autoscaling baseline |
| B5 | Custom scaling policy | Use the implemented policy and its documented inputs | Evaluate the proposed approach |

The exact replica counts and thresholds must be selected during pilot testing and recorded before the final comparison.

If a baseline cannot be implemented in the available environment, document the limitation rather than presenting it as tested.

## 5. Experimental Variables

### Independent Variables

Variables deliberately changed between experiments:

- Autoscaling strategy.
- Request arrival rate.
- Workload pattern.
- Worker startup delay.
- Minimum and maximum worker replicas.
- Scaling thresholds and cooldown settings.
- Inference workload or model configuration.

### Dependent Variables

Outcomes measured during experiments:

- p50, p95, and p99 end-to-end latency.
- SLA violation rate.
- Throughput.
- Queue depth over time.
- Worker replica count over time.
- Scale-up reaction time.
- Pod startup and readiness duration.
- Idle GPU time, where real GPU measurements are available.
- Estimated cost per 1,000 requests, where a defensible cost model is available.
- Error and timeout rates.

### Controlled Variables

Keep these consistent across comparable runs:

- Request payloads and inference workload.
- Load-generator configuration.
- Experiment duration and warm-up period.
- Hardware and cluster configuration.
- Queue technology and job-handling semantics.
- Model version and inference settings.
- Metric collection and calculation methods.

Any unavoidable differences must be documented.

## 6. Workload Profiles

| Profile | Description | Main evaluation goal |
|---|---|---|
| W1: Steady | Approximately constant request rate | Evaluate stable-state performance |
| W2: Burst | Sudden increase in request rate | Evaluate scale-up response |
| W3: Ramp-up | Gradually increasing demand | Evaluate scaling thresholds |
| W4: Ramp-down | Gradually decreasing demand | Evaluate scale-down and idle capacity |
| W5: Variable | Repeated periods of high and low demand | Evaluate oscillation and warm-capacity behavior |
| W6: Overload | Demand exceeds available processing capacity | Evaluate backlog, timeouts, and failure behavior |

For each workload, record the arrival rate, duration, request count, concurrency, and processing configuration.

Start with small pilot runs to identify safe, meaningful workload levels. Do not select final rates solely because they produce favorable results.

## 7. SLA Definition

The SLA must be specified before final experiments.

Define the following:

- **Latency target:** The maximum permitted end-to-end latency for a request.
- **Violation rule:** A request violates the SLA when its measured end-to-end latency exceeds the target.
- **Violation rate:** The number of violating requests divided by the number of eligible measured requests.
- **Error treatment:** Whether failed and timed-out requests are counted separately, as violations, or both.
- **Measurement window:** The period over which the violation rate is calculated.

For example, a latency target such as 1 second could be evaluated during pilot testing, but it must not be adopted as the final SLA without considering the chosen model, workload, and hardware.

Use the same SLA definition for every strategy being compared.

## 8. Experimental Procedure

Follow this procedure for each experiment.

1. Record the experiment ID, code version, environment, and selected scaling strategy.
2. Confirm that the API, queue, workers, autoscaler, and metrics collection are functioning.
3. Apply the required replica limits and scaling configuration.
4. Start metric collection before sending workload traffic.
5. Run the configured warm-up period.
6. Generate the workload according to its defined profile.
7. Collect request-level measurements and infrastructure metrics.
8. Continue collecting data through the defined measurement window.
9. Stop the workload and allow the system to reach the specified post-run condition.
10. Save results, configuration, logs, and relevant failure details.
11. Validate that the data is complete and the run followed the plan.
12. Repeat the run when practical to assess variability.

Do not change thresholds or other parameters during a measured run unless the experiment explicitly tests dynamic configuration changes.

## 9. Measurement Definitions

### 9.1 SLA Violation Rate

\[
\text{Violation Rate} =
\frac{\text{Requests exceeding the latency target}}
{\text{Eligible measured requests}}
\times 100\%
\]

Document how failed and timed-out requests are treated so that strategies cannot appear better simply because their failures were excluded.

### 9.2 Throughput

\[
\text{Throughput} =
\frac{\text{Successfully completed requests}}
{\text{Measurement duration in seconds}}
\]

Report the measurement window and distinguish successful throughput from incoming request rate.

### 9.3 Scale-Up Reaction Time

Measure the elapsed time between a defined demand trigger and the observed scaling response.

Record separate timestamps where possible for:

- Demand threshold crossing.
- Scaling decision.
- Replica-count change.
- Pod creation.
- Pod readiness.
- First successful job processed by the new capacity.

This separation helps identify whether delays originate in metric detection, controller decisions, container startup, model initialization, or job processing.

### 9.4 Idle GPU Time

For real GPU experiments, define idle GPU time using the available GPU telemetry and the selected utilization threshold.

Record:

- GPU observation interval.
- Utilization source.
- Threshold used to classify a GPU as idle.
- Number of allocated GPU devices.
- Whether the measurement covers all allocated devices or only worker activity.

If device \(i\) is classified as idle for \(T_i\) seconds:

\[
\text{Idle GPU time} = \sum_i T_i
\]

Report both the idle GPU time and the total allocated GPU time over the same measurement window. State the threshold and measurement limitations.

**Important:** Low utilization is an operational proxy for unused capacity, not proof that every low-utilization interval is entirely wasted. Do not calculate real idle GPU time from mock inference alone.

### 9.5 Idle GPU Time Reduction

When comparing against a baseline with nonzero idle GPU time:

\[
\text{Reduction (\%)} =
\frac{I_{\text{baseline}} - I_{\text{strategy}}}
{I_{\text{baseline}}}
\times 100\%
\]

Here, \(I\) represents idle GPU time measured under comparable conditions.

If the baseline has zero idle GPU time, this percentage is undefined and must not be reported.

### 9.6 Cost per 1,000 Requests

Where resource pricing or an explicit cost model is available:

\[
\text{Cost per 1,000 requests} =
\frac{\text{Total measured or estimated cost}}
{\text{Successfully completed requests}}
\times 1000
\]

Document the pricing source, resource billing assumptions, measurement duration, and treatment of failed requests. Label modeled costs as estimates rather than actual cloud bills.

## 10. Cold-Start Experiments

Cold-start experiments investigate how worker initialization affects the response to demand spikes.

Vary the startup delay under controlled conditions where feasible, or measure the actual startup time of the selected deployment.

Compare:

- Time until a scaling decision.
- Time until a new pod is created.
- Time until the pod is ready.
- Time until the new worker completes its first job.
- Queue depth during startup.
- SLA violation rate during the demand increase.

Possible mitigation strategies to evaluate include maintaining a minimum warm-worker count, scaling earlier based on queue growth, and applying a startup-aware scaling policy.

Clearly distinguish deliberately simulated startup delays from delays measured on real GPU infrastructure.

## 11. Repeated Runs and Data Quality

- Use the same configuration for repeated runs of a given experiment.
- Repeat experiments sufficiently to identify meaningful variability.
- Record unsuccessful or interrupted runs instead of silently discarding them.
- Validate timestamp consistency across request and infrastructure metrics.
- Check for missing samples and metric collection gaps.
- Separate warm-up measurements from the main measurement window.
- Report the number of runs and the aggregation method.
- Use medians, ranges, or confidence intervals where appropriate.

Avoid making conclusions from a single favorable run.

## 12. Result Storage

Suggested directory structure:

```text
results/
├── raw/
├── processed/
├── figures/
├── tables/
└── reports/
```

Suggested metadata for each run:

```json
{
  "experiment_id": "EXP-001",
  "strategy": "keda_queue_based",
  "workload_profile": "burst",
  "code_version": "recorded_git_commit",
  "inference_mode": "mock",
  "measurement_duration_seconds": 300,
  "sla_target_seconds": null,
  "status": "planned"
}
```

This JSON is an illustrative schema. Replace placeholder values with actual experiment metadata. Do not use `null` SLA values in a final measured experiment if an SLA violation rate is being reported.

Store large raw datasets outside Git when appropriate. Commit small, useful summaries and reproducibility metadata rather than large generated files by default.

## 13. Analysis and Comparison

For each strategy:

1. Validate data completeness.
2. Calculate latency percentiles and SLA violation rates.
3. Calculate successful throughput and error rates.
4. Plot queue depth and replica count over time.
5. Analyze scaling decisions against workload changes.
6. Measure startup and readiness delays.
7. Calculate idle GPU time only when suitable GPU telemetry exists.
8. Estimate cost only when the cost model is documented.
9. Compare results across repeated runs.
10. Identify trade-offs and explain observed limitations.

The main comparison is the trade-off between capacity efficiency and service quality, not merely which strategy uses the fewest replicas.

A strategy that reduces idle GPU time but substantially increases SLA violations may not be an acceptable improvement.

## 14. Threats to Validity

Potential limitations include:

- Mock inference does not reproduce real GPU execution.
- Different hardware can change startup times and throughput.
- Load-generator saturation can distort results.
- Metric sampling intervals can hide short-lived events.
- Cluster contention can affect measured latency.
- Queue and result-storage overhead can influence end-to-end performance.
- Different scaling thresholds can bias comparisons.
- Small sample sizes can produce unstable percentile estimates.
- Estimated infrastructure costs may not match actual billing.
- CPU-based scaling may behave differently for GPU-bound workloads.

Document these limitations in the final analysis and avoid generalizing beyond the tested environment.

## 15. Experiment Completion Checklist

- [ ] Research question and hypotheses are documented.
- [ ] Baselines are implemented or unavailable baselines are documented.
- [ ] Workload profiles are configured.
- [ ] SLA targets and violation rules are finalized.
- [ ] Metrics are validated before the main runs.
- [ ] Experiment configurations are saved.
- [ ] Repeated runs are completed where practical.
- [ ] Mock and real GPU results are separated.
- [ ] Data quality is checked.
- [ ] Results and plots are saved.
- [ ] Limitations and threats to validity are documented.
- [ ] Conclusions are supported by measured evidence.
