# Environment and Setup Guide

## 1. Prerequisites

Install the following tools as needed for the development stage.

| Tool | Purpose | Required stage |
|---|---|---|
| Python 3.11+ | API, worker, and controller development | Initial development |
| Git | Version control | All stages |
| Docker | Containerizing application components | Containerization |
| Docker Compose | Running local multi-container services | Local integration |
| Kubernetes | Container orchestration | Autoscaling experiments |
| kubectl | Managing Kubernetes resources | Kubernetes deployment |
| Helm | Installing packaged Kubernetes components | If used for deployment |
| Redis or RabbitMQ | Message queue | Queue integration |
| Prometheus | Metrics collection | Observability |
| KEDA | Queue-based autoscaling | Autoscaling experiments |
| Locust or k6 | Load generation | Performance testing |
| GPU-enabled environment | Real GPU inference measurements | GPU experiments |

Python 3.11 or another supported version should be selected and used consistently across development and testing. Confirm compatibility with the dependencies before upgrading Python.

Docker Compose is optional if the local environment uses another orchestration method.

## 2. Recommended Development Environment

- **Operating system:** Windows with WSL2, Linux, or another supported Docker environment.
- **Editor:** Visual Studio Code or an equivalent editor.
- **Virtual environment:** Python `venv`.
- **Application framework:** FastAPI.
- **Testing:** pytest.
- **Container runtime:** Docker.
- **Version control:** Git and GitHub.

Kubernetes, KEDA, and GPU dependencies do not need to be installed for initial unit tests or mock-inference development.

## 3. Repository Setup

Clone the repository:

```bash
git clone https://github.com/PragatiGupta2005/Serverless-GPU-Inference-Auto-Scaling-System.git
cd Serverless-GPU-Inference-Auto-Scaling-System
```

Create and activate a virtual environment.

### Windows PowerShell

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

If PowerShell blocks activation, use a permitted activation method for your environment rather than changing system security settings unnecessarily.

### Linux or macOS

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Verify the Python version:

```bash
python --version
```

## 4. Install Dependencies

Install dependencies from the project's dependency file once it has been created.

For a `requirements.txt` setup:

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

If the dependency file does not yet exist, install only the packages required for the current development task.

Initial application dependencies may include:

- `fastapi`
- `uvicorn`
- `pydantic`
- `pytest`
- `httpx`

Add queue clients, Prometheus instrumentation, and other packages when the corresponding components are implemented.

Keep dependency versions consistent and update the dependency file whenever packages are added or removed.

## 5. Environment Variables

Use a local `.env` file for development-specific configuration when the application uses a compatible settings loader.

Maintain `.env.example` as a template containing placeholders and non-sensitive defaults only.

Illustrative configuration:

```dotenv
APP_ENV=development
LOG_LEVEL=INFO

API_HOST=127.0.0.1
API_PORT=8000

QUEUE_BACKEND=redis
QUEUE_URL=redis://localhost:6379/0

METRICS_ENABLED=true
PROMETHEUS_PORT=9090

MIN_REPLICAS=1
MAX_REPLICAS=5
```

These variables are proposed examples, not confirmed application settings. Use them only after the corresponding configuration is implemented.

Rules:

- Never commit real credentials.
- Add `.env` to `.gitignore`.
- Validate required variables during startup.
- Keep development and experiment settings separate.
- Avoid maintaining duplicate configuration systems for the same component.
- Use the existing configuration structure when adding new settings.

## 6. Run the API Locally

Once the FastAPI application and its entry point have been implemented, run it using the actual module path.

For example, if the entry point is `service.main:app`:

```bash
uvicorn service.main:app --reload
```

Open the API documentation at:

`http://127.0.0.1:8000/docs`

The module path above is an example. Update it to match the actual repository structure.

Do not use this command until the specified Python module and FastAPI application object exist.

## 7. Local Queue Setup

Choose one queue technology for the initial implementation.

### Option A: Redis

If Redis is selected and Docker is available:

```bash
docker run --name inference-redis -p 6379:6379 -d redis:7
```

Check the container:

```bash
docker ps
```

Stop it when finished:

```bash
docker stop inference-redis
```

Remove it when it is no longer needed:

```bash
docker rm inference-redis
```

The command uses a local development configuration. Do not expose an unauthenticated Redis instance to an untrusted network.

### Option B: RabbitMQ

If RabbitMQ is selected instead, use an appropriate RabbitMQ image and configure the application with the correct connection settings.

Do not run both queue systems unless there is a specific experimental requirement.

## 8. Docker and Containerization

When Dockerfiles are implemented, build the application image using the relevant build context.

Example:

```bash
docker build -t gpu-inference-app:dev .
```

The command assumes a Dockerfile exists in the repository root. If the Dockerfile is stored in a subdirectory, specify its path and build context accordingly.

Check running containers:

```bash
docker ps
```

Container images should:

- Use maintained base images.
- Install only required dependencies.
- Avoid embedding credentials.
- Use non-root execution where practical.
- Configure appropriate health checks and resource limits.

## 9. Kubernetes Setup

Kubernetes is required for testing deployment and replica-scaling behavior.

Before deploying, verify cluster access:

```bash
kubectl cluster-info
kubectl get nodes
```

Check the current context:

```bash
kubectl config current-context
```

Ensure that commands target the intended development cluster.

The initial prototype may use a local Kubernetes distribution or a remote cluster. The choice should be documented once finalized.

## 10. KEDA and Prometheus

Install and configure KEDA and Prometheus only when the project reaches the observability and autoscaling stages.

Before running scaling experiments, verify that:

- KEDA is installed and its components are healthy.
- The selected queue scaler can access the queue.
- The scaling target is configured correctly.
- Prometheus can scrape the intended metrics.
- Required metric names and units are documented.
- The worker replica count changes according to the configured policy.

Installation commands should be added after selecting the cluster environment and deployment method. Avoid copying commands for a different cluster configuration without checking compatibility.

## 11. GPU Environment

Real GPU experiments require compatible hardware and software.

Verify:

- The GPU is available to the host or cloud environment.
- Required drivers are installed.
- The container runtime supports GPU access when containers are used.
- Kubernetes can schedule GPU workloads if the experiment uses Kubernetes.
- The necessary device plugin and resource configuration are present.
- The model and inference framework support the selected hardware.

GPU-specific commands depend on the selected hardware and framework. Record the GPU model, driver version, runtime, and framework version for each experiment.

If GPU access is unavailable, continue with mock inference and label the results accordingly.

## 12. Testing Commands

Run the test suite:

```bash
pytest -v
```

Run a particular test directory:

```bash
pytest -v tests/unit/
```

These commands assume that the relevant tests and directories exist.

For reproducibility, record the Python version, dependency versions, and any external services required by each test.

## 13. Troubleshooting

| Issue | Checks |
|---|---|
| Python command not found | Verify the Python installation and PATH configuration |
| Package installation fails | Check the active virtual environment, Python compatibility, and network access |
| API module not found | Confirm the entry-point path and run the command from the repository root |
| Queue connection fails | Check that the queue is running and connection settings are correct |
| Docker command fails | Verify Docker installation and daemon status |
| Kubernetes context fails | Check the active context, cluster connectivity, and permissions |
| Autoscaling does not occur | Inspect KEDA status, scaling configuration, queue metrics, and target replicas |
| Prometheus has no data | Verify scrape configuration, endpoint availability, and metric names |
| GPU is unavailable | Check hardware access, drivers, runtime configuration, and Kubernetes GPU resources |
| Tests fail only in integration | Verify external services and test-specific configuration |

## 14. Environment Checklist

- [ ] Repository cloned successfully.
- [ ] Python version verified.
- [ ] Virtual environment created.
- [ ] Required dependencies installed.
- [ ] Local configuration documented.
- [ ] `.env` excluded from Git.
- [ ] API starts successfully once implemented.
- [ ] Queue connectivity verified once integrated.
- [ ] Unit tests run successfully.
- [ ] Kubernetes access verified before deployment tests.
- [ ] Prometheus and KEDA verified before autoscaling experiments.
- [ ] GPU environment documented before real GPU measurements.
