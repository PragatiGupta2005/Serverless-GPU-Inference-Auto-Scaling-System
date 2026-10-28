# Testing Strategy

## 1. Testing Principles

- Test individual components before integrating them.
- Automate repeatable checks wherever practical.
- Test successful behavior, failure handling, and boundary conditions.
- Keep unit tests independent of Kubernetes and GPU hardware.
- Use mocks for external dependencies in isolated tests.
- Run integration tests against real services when validating service interactions.
- Record test results and investigate failures before accepting changes.
- Distinguish simulated results from measurements obtained using real GPU hardware.

## 2. Testing Levels

| Test level | Purpose | Examples |
|---|---|---|
| Unit testing | Verify individual functions and classes | Queue-depth calculations, configuration validation |
| API testing | Verify HTTP endpoints and validation | Valid requests, malformed payloads, error responses |
| Integration testing | Verify component interactions | API-to-queue, queue-to-worker |
| Infrastructure testing | Verify deployment components | Kubernetes manifests, KEDA configuration |
| Load testing | Evaluate behavior under different workloads | Bursts, steady traffic, overload |
| Regression testing | Detect previously fixed defects | Duplicate job handling, timeout behavior |
| Experiment validation | Verify measurement quality | Metric completeness, repeatability, result consistency |

## 3. Unit Testing

**Tool:** `pytest`

Unit tests should cover isolated application logic without requiring a running Kubernetes cluster or GPU.

Test the following areas:

### Configuration

- Required configuration values are validated.
- Invalid URLs, timeouts, and replica limits are rejected.
- Defaults are applied correctly.
- Sensitive values are not included in error messages.

### Request Validation

- Valid inference payloads are accepted.
- Missing required fields are rejected.
- Incorrect data types are rejected.
- Oversized or unsupported inputs are handled according to policy.

### Queue and Job Logic

- Jobs receive unique identifiers.
- Job states transition correctly.
- Queue errors are handled.
- Retry limits are respected.
- Duplicate delivery does not incorrectly mark a job as completed more than once.

### Scaling Policy

- Low demand produces the expected scaling decision.
- Increasing queue depth can trigger scale-up.
- Scale-down respects minimum replica limits.
- Scale-up respects maximum replica limits.
- Cooldowns and hysteresis work as configured.
- Missing, invalid, or stale metrics are handled safely.
- Startup delay is incorporated if required by the custom policy.

Test expected behavior at threshold boundaries, not only typical inputs.

## 4. API Testing

Use FastAPI's test client or an equivalent HTTP testing approach.

Verify the following:

- Health and readiness endpoints return the expected responses.
- Valid jobs are accepted.
- Invalid payloads return appropriate HTTP 4xx responses.
- Queue submission failures return suitable service errors.
- Unknown request IDs return the documented response.
- Job status and result endpoints return consistent states.
- Timeouts are handled correctly.
- Internal stack traces and secrets are not exposed to clients.
- Authentication and authorization are enforced when enabled.

For asynchronous request handling, verify that the API returns a request ID only after the job has been successfully accepted by the queue.

## 5. Queue and Worker Integration Testing

Run integration tests using the selected queue technology.

### Queue Tests

- A valid job can be enqueued and retrieved.
- Queue depth changes as jobs are added and processed.
- Unavailable queue services are handled correctly.
- Job acknowledgements remove successfully processed jobs according to the queue's semantics.
- Retry and redelivery behavior matches the configured policy.

### Worker Tests

- A worker can retrieve and process a valid job.
- Successful processing produces the expected result.
- Processing failures are recorded.
- Retry limits are enforced.
- Worker termination during processing does not silently lose a job.
- Readiness checks reflect whether the worker can process jobs.

Use mock inference for basic integration tests so that they can run without GPU hardware.

## 6. Autoscaling Tests

Validate each scaling strategy separately before comparing strategies.

### KEDA Queue-Based Scaling

- KEDA can access the configured queue metrics.
- Scaling activates when configured conditions are met.
- Replica counts remain within configured limits.
- Scale-down behavior follows the configured cooldown and stabilization behavior.
- Missing queue metrics do not result in uncontrolled scaling.

### Custom Scaling Controller

- Inputs are parsed and validated correctly.
- Replica recommendations follow the implemented policy.
- Invalid metrics are handled safely.
- Minimum and maximum replica constraints are enforced.
- Cooldown or hysteresis logic prevents unwanted rapid oscillations.
- The controller does not conflict with another component managing the same replicas.

### Scaling Lifecycle

Verify the full lifecycle:

1. Submit enough jobs to create a backlog.
2. Confirm that the active autoscaler detects demand.
3. Observe the replica-count change.
4. Measure the time until new pods become ready.
5. Confirm that additional workers process jobs.
6. Reduce demand and observe scale-down behavior.

Record actual timing measurements rather than assuming that a scaling request immediately adds usable capacity.

## 7. Kubernetes and Deployment Testing

Where a Kubernetes environment is available, verify:

- Application containers start successfully.
- Required environment variables and secrets are configured.
- Readiness and liveness probes behave as expected.
- Worker pods can access the queue.
- Resource requests and limits are applied.
- KEDA can communicate with the relevant scaling target.
- Pods become ready before receiving work.
- Failed pods are restarted or replaced as configured.
- Deployments remain within the intended replica limits.
- Monitoring services can collect the expected metrics.

GPU-specific tests must additionally verify compatible hardware, GPU resource allocation, device visibility, and model initialization.

## 8. Load Testing

**Possible tools:** Locust or k6.

Run controlled workloads to evaluate queue behavior, latency, throughput, and autoscaling response.

### Workload Profiles

| Profile | Pattern | Purpose |
|---|---|---|
| Steady | Approximately constant request rate | Measure stable performance |
| Burst | Sudden increase in request rate | Evaluate scale-up responsiveness |
| Ramp-up | Gradually increasing request rate | Observe scaling thresholds |
| Ramp-down | Gradually decreasing request rate | Evaluate scale-down behavior |
| Variable | Alternating periods of high and low demand | Examine oscillation and warm-capacity behavior |
| Overload | Demand exceeds available processing capacity | Evaluate queue growth, timeouts, and failure handling |

Each run should record the request rate, duration, concurrency, processing workload, and scaling configuration.

## 9. Performance Metrics

Collect the following metrics during load and experiment tests:

| Metric | Validation purpose |
|---|---|
| p50, p95, and p99 end-to-end latency | Understand request response-time distribution |
| SLA violation rate | Calculate the proportion of requests exceeding the defined latency target |
| Throughput | Measure successfully completed requests per second |
| Queue depth | Measure pending workload |
| Error and timeout rate | Assess reliability |
| Scale-up reaction time | Measure response to increased demand |
| Pod startup/readiness duration | Quantify worker initialization delay |
| Worker replica count | Understand scaling behavior |
| Idle GPU time | Estimate unused GPU capacity in real GPU experiments |
| Cost per 1,000 requests | Compare cost using a documented cost model |

The SLA threshold and measurement methodology must be defined in `docs/EXPERIMENT_PLAN.md` before final comparisons.

## 10. Experiment Reproducibility

For each experiment, record:

- Experiment ID.
- Date and time.
- Git commit or code version.
- Scaling strategy.
- Workload profile and request rate.
- Test duration and warm-up period.
- Model and inference configuration.
- Queue configuration.
- Minimum and maximum worker replicas.
- Relevant autoscaling thresholds.
- Kubernetes and container configuration.
- Hardware details, including GPU type where applicable.
- Collected metrics and output file locations.
- Errors, interruptions, or deviations from the planned procedure.

Use consistent workload definitions and measurement windows when comparing strategies.

Repeat runs when practical to identify variability. Do not compare results obtained under materially different conditions without explaining those differences.

## 11. Mock and Real GPU Testing

### Mock Inference

Use mock inference for:

- API and queue development.
- Worker lifecycle testing.
- Autoscaling logic validation.
- CPU-only CI tests.
- Early load-testing experiments.

Mock workloads may simulate processing delays, but they cannot demonstrate actual GPU utilization, GPU startup behavior, or GPU cost savings.

### Real GPU Inference

When compatible GPU resources are available, validate:

- Model-loading and initialization time.
- GPU utilization and memory consumption.
- Worker readiness after model initialization.
- Actual inference latency and throughput.
- Cold-start behavior during scale-up.
- Resource consumption during idle periods.

Report mock and real GPU measurements separately.

## 12. Test Automation and Execution

Suggested local test command:

```bash
pytest -v
```

To run tests in a specific directory:

```bash
pytest -v tests/unit/
pytest -v tests/integration/
```

These commands assume the corresponding directories exist. Update them to match the final repository structure.

Tests requiring external services should be marked or organized separately so that CPU-only unit tests remain runnable without Kubernetes, Redis/RabbitMQ, or a GPU.

## 13. Suggested Test Organization

```text
tests/
├── unit/
│   ├── test_config.py
│   ├── test_request_validation.py
│   └── test_scaling_policy.py
├── api/
│   └── test_inference_api.py
├── integration/
│   ├── test_queue_worker.py
│   └── test_job_lifecycle.py
├── infrastructure/
│   └── test_deployment_config.py
└── performance/
    └── README.md
```

This is a proposed structure. Create test files as their corresponding components are implemented.

## 14. Test Completion Criteria

- [ ] Relevant unit tests pass.
- [ ] API validation and error-handling tests pass.
- [ ] Queue and worker integration tests pass.
- [ ] Scaling-policy boundary tests pass.
- [ ] Kubernetes and KEDA behavior is verified where the environment permits.
- [ ] Load-test outputs include the required metrics.
- [ ] Experiment configurations and results are saved.
- [ ] Known failures and environmental limitations are documented.
- [ ] Mock results are clearly distinguished from real GPU results.

Do not mark a test category as completed until the tests have actually been executed and reviewed.
