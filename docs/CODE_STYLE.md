# Code Style and Development Conventions

## 1. General Principles

- Write readable, maintainable, and modular code.
- Prefer simple implementations over unnecessary abstractions.
- Keep functions and classes focused on a single responsibility.
- Avoid duplicating logic across modules.
- Use descriptive names instead of unexplained abbreviations.
- Follow consistent formatting across the repository.
- Document important design choices and non-obvious behavior.

## 2. Python Conventions

### Formatting

- Follow PEP 8 for Python style.
- Use four spaces for indentation.
- Keep lines within a reasonable length; target 88 characters when using Black.
- Separate top-level functions and classes with two blank lines.
- Use trailing commas in multiline collections and function calls where appropriate.
- Avoid wildcard imports.

### Naming

| Element | Convention | Example |
|---|---|---|
| Variables | `snake_case` | `queue_depth` |
| Functions | `snake_case` | `calculate_replica_count()` |
| Modules | `snake_case` | `metrics_collector.py` |
| Classes | `PascalCase` | `ScalingController` |
| Constants | `UPPER_SNAKE_CASE` | `MAX_REPLICAS` |
| Private helpers | Leading underscore | `_estimate_startup_delay()` |

Use names that describe the purpose of the variable, function, or class.

## 3. Type Hints

- Add type hints to public functions and methods.
- Use explicit return types.
- Use standard Python typing features where they improve clarity.
- Define request and response schemas explicitly.
- Avoid using `Any` unless dynamic typing is genuinely necessary.

Example:

```python
def calculate_queue_pressure(
    queue_depth: int,
    arrival_rate: float,
    processing_rate: float,
) -> float:
    """Estimate queue pressure from workload measurements."""
    if processing_rate <= 0:
        return float("inf")

    return queue_depth + max(0.0, arrival_rate - processing_rate)
```

The formula above is illustrative. It must not be treated as the final autoscaling algorithm without validation.

## 4. Functions and Classes

- Keep functions small enough to understand easily.
- Extract repeated logic into reusable helpers.
- Use classes when they represent meaningful state or behavior.
- Avoid classes that only wrap a single function without a clear benefit.
- Pass dependencies explicitly when practical instead of relying on hidden global state.
- Avoid excessive nesting and deeply coupled modules.
- Use early returns to simplify conditional logic when appropriate.

## 5. FastAPI Conventions

- Define API request and response models explicitly.
- Use Pydantic schemas for payload validation.
- Keep route handlers focused on HTTP concerns.
- Move business logic into service modules.
- Return appropriate HTTP status codes.
- Use dependency injection for shared services and resources where appropriate.
- Document endpoint behavior and expected errors.
- Avoid putting long-running inference operations directly inside route handlers when they should be processed by workers.

Illustrative structure:

```python
@router.post("/inference")
async def submit_inference(request: InferenceRequest) -> InferenceResponse:
    """Validate and submit an inference job."""
    result = await inference_service.submit(request)
    return InferenceResponse(request_id=result.request_id)
```

This example assumes that the relevant router, schemas, and service already exist. Adapt it to the implemented API and response mode.

## 6. Error Handling

- Raise specific exceptions rather than generic exceptions where possible.
- Catch exceptions only when they can be handled meaningfully.
- Do not silently ignore errors.
- Include useful context in logs without exposing secrets or sensitive payloads.
- Return sanitized error responses to API clients.
- Distinguish validation failures, queue failures, inference failures, and timeouts.
- Avoid broad `except Exception` blocks unless there is a clear boundary for logging and safely handling unexpected failures.

## 7. Logging

- Use Python's standard `logging` module or the project's configured logging library.
- Do not use `print()` for routine application logging.
- Use appropriate levels: `DEBUG`, `INFO`, `WARNING`, `ERROR`, and `CRITICAL`.
- Include useful context such as request IDs, job status, and worker identifiers when appropriate.
- Do not log credentials, access tokens, or unnecessary request contents.
- Keep metric labels bounded; use logs rather than metric labels for highly variable identifiers.

Example:

```python
import logging

logger = logging.getLogger(__name__)

logger.info(
    "Inference job completed",
    extra={"request_id": request_id},
)
```

Ensure that the configured logging formatter actually includes the additional fields before relying on them in log output.

## 8. Configuration Management

- Keep configurable values outside core business logic.
- Use environment variables or configuration files for deployment-specific settings.
- Validate configuration at application startup.
- Define sensible defaults for local development where appropriate.
- Avoid hardcoding credentials, hostnames, replica limits, or experiment thresholds.
- Keep example configuration files free of real secrets.
- Reuse the project's existing configuration structure instead of creating duplicate configuration systems.

Potential configuration values include queue connection settings, metric endpoints, replica limits, worker timeouts, and experiment parameters.

## 9. Asynchronous Programming

- Use `async` and `await` for genuinely asynchronous I/O operations.
- Avoid blocking the event loop with long-running synchronous operations.
- Use an appropriate execution mechanism for blocking I/O or CPU-intensive work.
- Ensure that queue clients and network resources are closed correctly.
- Do not assume that asynchronous code automatically makes GPU inference faster.
- Manage task cancellation, timeouts, and exceptions explicitly.

## 10. Metrics and Observability

- Use consistent metric names and units.
- Include units in metric names where appropriate, such as `_seconds` or `_total`.
- Follow Prometheus naming conventions.
- Use counters for cumulative events, gauges for current values, and histograms for distributions.
- Avoid high-cardinality labels such as unique request IDs.
- Keep metric definitions and meanings documented.
- Distinguish request latency, queue waiting time, worker processing time, and pod startup time.

## 11. Testing Conventions

- Use `pytest` for Python tests.
- Name test files `test_*.py`.
- Name test functions `test_<behavior>()`.
- Keep unit tests independent and deterministic.
- Use fixtures for reusable setup.
- Mock external services when testing isolated business logic.
- Add integration tests for queue, API, and worker interactions.
- Test error paths as well as successful execution.
- Never rely on a real GPU for tests that are intended to run in a CPU-only environment.

## 12. Documentation and Comments

- Write docstrings for public modules, classes, and functions where useful.
- Explain why non-obvious decisions are made, not merely what the code does.
- Update documentation when interfaces or behavior change.
- Avoid comments that repeat the code literally.
- Keep examples consistent with the actual implementation.
- Clearly label proposed algorithms and configuration values that have not been validated.

## 13. Dependency and Import Rules

- Group imports in this order:
  1. Python standard library.
  2. Third-party packages.
  3. Internal project modules.
- Use a blank line between import groups.
- Avoid circular imports.
- Add dependencies only when they provide a clear benefit.
- Update the project's dependency-management files when dependencies change.
- Do not introduce multiple libraries for the same purpose without a documented reason.

## 14. Suggested Development Tools

The following tools may be adopted as the project matures:

| Tool | Purpose |
|---|---|
| Black | Automatic Python formatting |
| Ruff | Linting and common code-quality checks |
| Mypy or Pyright | Optional static type checking |
| Pytest | Automated testing |

The team should select and configure tools consistently rather than requiring every tool immediately.

## 15. Pull Request and Commit Guidelines

- Keep changes focused on one logical task.
- Test relevant functionality before committing.
- Update documentation when behavior changes.
- Avoid committing generated files, secrets, local environments, or temporary experiment outputs unless explicitly required.
- Use clear commit messages.

Suggested commit message format:

```text
<type>: <short description>
```

Examples:

```text
feat: add queue depth metrics
fix: handle worker timeout
test: add scaling policy tests
docs: update code style guidelines
refactor: separate queue service logic
```

## 16. Code Review Checklist

- [ ] Names and formatting follow the conventions.
- [ ] Functions have clear responsibilities.
- [ ] Public interfaces have appropriate type hints.
- [ ] Errors are handled explicitly.
- [ ] Logs do not expose sensitive information.
- [ ] Configuration is not unnecessarily hardcoded.
- [ ] Relevant tests pass.
- [ ] Documentation is updated where necessary.
- [ ] No unrelated changes or temporary files are included.

