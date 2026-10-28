# Research Notes

## 1. Research Objective

Investigate existing approaches to inference-serving autoscaling, GPU resource efficiency, queue-based scaling, and cold-start mitigation.

The research should establish what existing systems already provide, identify limitations relevant to the project's research question, and guide the experimental design.

## 2. Core Research Questions

- How do queue-based autoscalers respond to sudden workload increases?
- How do worker startup and model initialization delays affect end-to-end latency?
- What trade-offs exist between keeping GPU workers warm and reducing idle GPU time?
- How do Kubernetes HPA and KEDA behave under bursty inference workloads?
- Can a startup-aware scaling policy improve the trade-off between resource efficiency and SLA violations?
- Which metrics are needed to compare autoscaling strategies fairly?

## 3. Existing Technologies to Investigate

### 3.1 Kubernetes Horizontal Pod Autoscaler (HPA)

**Purpose:** Adjust workload replica counts based on configured metrics, commonly CPU or memory utilization.

**Research questions:**
- How quickly does HPA react to workload changes?
- How do metric collection intervals and scaling behavior affect responsiveness?
- How suitable is CPU-based scaling for GPU-bound inference workloads?
- What configuration is needed for custom or external metrics?

**Relevance:** Potential baseline for comparing conventional resource-metric-based autoscaling with queue-based approaches.

Official documentation: https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/

### 3.2 KEDA

**Purpose:** Enable event-driven Kubernetes scaling using metrics from supported event sources, including queue systems.

**Research questions:**
- Which scaler supports the selected queue?
- How is queue depth translated into replica counts?
- How do polling intervals, cooldown periods, minimum replicas, and maximum replicas affect scaling?
- What happens when queue metrics become unavailable?

**Relevance:** Standard queue-based autoscaling baseline.

Official documentation: https://keda.sh/docs/

### 3.3 KServe

**Purpose:** Provide Kubernetes-based model-serving infrastructure with serving and scaling capabilities.

**Research questions:**
- Which autoscaling mechanisms are available in the selected deployment configuration?
- How are model loading, readiness, and scale-to-zero handled?
- What additional infrastructure does KServe require?
- Would KServe simplify deployment or make the experiment harder to isolate?

**Relevance:** A reference for established model-serving architecture and inference workload management.

Official documentation: https://kserve.github.io/website/

### 3.4 Knative Serving

**Purpose:** Provide request-driven scaling for containerized services, including scale-to-zero capabilities.

**Research questions:**
- How does request-driven scaling differ from queue-depth-based scaling?
- What are the cold-start implications of scale-to-zero?
- Which concurrency and minimum-scale settings influence latency?
- Is Knative suitable for the selected worker-processing architecture?

**Relevance:** A reference for serverless scaling and cold-start trade-offs.

Official documentation: https://knative.dev/docs/serving/

### 3.5 NVIDIA Triton Inference Server

**Purpose:** Serve machine-learning models with inference-serving and performance-management capabilities.

**Research questions:**
- How does model loading affect worker startup time?
- What batching and concurrency features affect throughput and latency?
- Which metrics can be collected for inference performance and GPU utilization?
- Could Triton be used as the inference engine for the real-GPU phase?

**Relevance:** Potential inference-serving component for GPU-backed experiments.

Official documentation: https://docs.nvidia.com/deeplearning/triton-inference-server/

### 3.6 NVIDIA Dynamo

**Purpose:** Provide infrastructure for serving and scaling AI inference workloads, particularly generative AI workloads.

**Research questions:**
- Which scheduling and scaling capabilities overlap with this project?
- How does the system account for workload characteristics and inference capacity?
- What hardware and deployment requirements would make a comparison feasible?
- Would using Dynamo support the research or introduce unnecessary complexity?

**Relevance:** A related modern inference-serving system to review before claiming novelty.

Official documentation: https://docs.nvidia.com/dynamo/

**Note:** Verify the capabilities and version-specific behavior of each technology against its current documentation before drawing conclusions.

## 4. Literature Review Record

Use the following template for every paper or technical publication reviewed.

### Research Entry Template

- **Title:**
- **Authors:**
- **Publication year:**
- **Publication venue:**
- **Paper or source URL:**
- **Problem addressed:**
- **Method proposed:**
- **Evaluation environment:**
- **Metrics used:**
- **Main findings:**
- **Limitations:**
- **Relevance to this project:**
- **Ideas to test:**
- **Review status:** Not reviewed / In progress / Reviewed

Record findings in your own words and distinguish the source's claims from your interpretation.

## 5. Comparison Framework

Compare existing approaches using the following dimensions.

| Dimension | Questions to investigate |
|---|---|
| Scaling signal | CPU, memory, requests, queue depth, or other metrics? |
| Startup awareness | Does the approach account for worker initialization delays? |
| Scale-to-zero | Is it supported, and under what conditions? |
| Warm capacity | Can a minimum number of workers be maintained? |
| Workload assumptions | Are workloads steady, bursty, predictable, or variable? |
| Latency evaluation | Are p95/p99 latency and SLA violations measured? |
| Resource efficiency | Is idle GPU time measured directly or estimated? |
| Experimental reproducibility | Are configurations, workloads, and evaluation procedures documented? |
| Operational complexity | What additional services and controllers are required? |
| Hardware requirements | Is real GPU hardware required? |

Do not fill this comparison with assumptions. Add evidence and citations after reviewing each source.

## 6. Preliminary Research Gaps to Investigate

The following are candidate research directions, not confirmed research gaps:

- Whether queue-depth-only scaling reacts too late when GPU workers have long startup times.
- Whether startup-aware decisions improve latency without requiring excessive warm capacity.
- How different workload patterns change the efficiency-versus-latency trade-off.
- How much of observed delay comes from autoscaler reaction versus container and model startup.
- Whether a simpler custom policy can improve a clearly defined metric under controlled conditions.

Before selecting a claimed research gap, review related papers and existing systems to establish what has already been studied.

## 7. Design Decision Log

Record important decisions using this format:

| Decision | Options considered | Selected option | Rationale | Evidence/status |
|---|---|---|---|---|
| Queue technology | Redis / RabbitMQ | Pending | To be determined after reviewing integration needs | Open |
| Inference engine | Mock / real model / Triton | Pending | Depends on prototype stage and available hardware | Open |
| Standard autoscaler | HPA / KEDA | Pending | KEDA is the planned queue-based baseline; final configuration must be verified | Open |
| Custom policy inputs | Queue depth / arrival rate / latency / startup delay | Pending | To be evaluated experimentally | Open |
| GPU environment | Local / cloud / shared GPU | Pending | Depends on resource availability | Open |

Update the table when decisions are made. Record the reason for each choice and any relevant experiment or source.

## 8. Research Integrity

- Do not claim novelty without reviewing related work.
- Do not describe proposed targets as achieved results.
- Do not treat vendor documentation as independent experimental validation.
- Distinguish published findings from project measurements.
- Record contradictory evidence rather than selecting only favorable results.
- Cite original papers and official documentation where possible.
- Separate mock-inference results from real GPU results.
- Report limitations and failed experiments honestly.

## 9. Research Progress Checklist

- [ ] Review Kubernetes HPA documentation.
- [ ] Review KEDA documentation for the selected queue.
- [ ] Review relevant KServe scaling documentation.
- [ ] Review Knative cold-start and scaling behavior.
- [ ] Investigate Triton for potential real-GPU inference.
- [ ] Review NVIDIA Dynamo for overlapping capabilities.
- [ ] Find and review relevant research papers on inference autoscaling and cold starts.
- [ ] Build an evidence-backed comparison table.
- [ ] Identify a defensible research gap.
- [ ] Finalize the custom policy's motivation and evaluation criteria.
- [ ] Update design decisions based on evidence.
