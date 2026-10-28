# Security Policy

## 1. Purpose

Define the security requirements for development, deployment, testing, and maintenance of this repository.

## 2. Secrets and Configuration

- Never commit API keys, passwords, access tokens, private keys, or cloud credentials.
- Store sensitive configuration in environment variables or an appropriate secret-management system.
- Maintain a `.env.example` file containing placeholder values only.
- Add `.env` and other local secret files to `.gitignore`.
- Do not print secrets in application logs, error messages, or test output.
- Rotate any credential that is accidentally exposed.
- Avoid storing credentials in Docker images, container layers, or Kubernetes manifests committed to the repository.

## 3. API Security

- Validate all incoming request payloads using explicit schemas.
- Reject malformed, oversized, or unsupported requests.
- Apply request-size limits and appropriate timeouts.
- Return meaningful HTTP status codes without exposing stack traces or internal configuration.
- Restrict CORS to approved origins when browser-based access is required.
- Apply authentication and authorization before exposing the API beyond a trusted development environment.
- Add rate limiting or other admission controls where appropriate.
- Ensure that clients cannot access another user's jobs or results without authorization, if user-level access is supported.

## 4. Queue and Job Security

- Do not expose the message queue directly to public networks.
- Use authentication and access controls for queue connections.
- Keep queue credentials outside source code.
- Validate jobs before placing them on the queue and before processing them.
- Define maximum job sizes, processing deadlines, and retry limits.
- Prevent unbounded retries and uncontrolled queue growth.
- Avoid placing sensitive information in job payloads unless necessary.
- Ensure that job identifiers cannot be used to bypass authorization checks.

## 5. Container Security

- Use trusted base images and pin versions where practical.
- Install only required system packages and Python dependencies.
- Avoid running containers as root unless strictly necessary.
- Use a non-root user in Docker images where compatible.
- Configure CPU and memory requests and limits.
- Do not embed secrets in image build arguments or image layers.
- Scan dependencies and container images for known vulnerabilities when suitable tooling is available.
- Rebuild images when important security fixes are required.

## 6. Kubernetes Security

- Follow the principle of least privilege for Kubernetes service accounts and RBAC permissions.
- Avoid granting cluster-admin privileges to application components.
- Keep Kubernetes API access restricted.
- Use Kubernetes Secrets or an external secret manager for sensitive values; do not commit real secret values.
- Restrict network access between services where the environment supports network policies.
- Apply resource limits to reduce the risk of resource exhaustion.
- Avoid mounting the host filesystem or privileged devices unless required.
- Grant GPU device access only to workloads that need it.
- Separate development experiments from production or shared environments.

## 7. Monitoring and Observability Security

- Restrict access to Prometheus, Grafana, controller endpoints, and administrative dashboards.
- Do not expose metrics endpoints publicly without appropriate access controls.
- Avoid including request payloads, credentials, or sensitive user data in metric labels.
- Do not use request IDs or other high-cardinality values as metric labels without a justified need.
- Protect monitoring data when it reveals infrastructure details or workload patterns.
- Use sanitized logs and meaningful error messages.

## 8. Dependencies and Supply Chain

- Track Python dependencies in the project's dependency-management files.
- Pin or constrain dependency versions where appropriate for reproducibility and security.
- Review dependency changes before merging.
- Prefer maintained packages from trusted sources.
- Remove unused dependencies.
- Investigate reported vulnerabilities and update affected packages when feasible.
- Keep container base images and deployment tools reasonably current.

## 9. GPU and Workload Safety

- Apply resource limits to prevent accidental resource exhaustion.
- Validate model and input compatibility before executing inference.
- Restrict access to GPU resources to authorized workloads.
- Avoid loading untrusted model files or executing untrusted serialized objects.
- Monitor memory usage, worker crashes, timeouts, and abnormal resource consumption.
- Use workload limits to prevent a single experiment from exhausting shared infrastructure.

## 10. Logging and Data Handling

- Collect only the data needed for debugging and evaluation.
- Avoid logging authentication headers, secrets, and unnecessary request content.
- Sanitize externally supplied strings before logging.
- Define retention and cleanup rules for stored results and experiment logs.
- Use synthetic or non-sensitive data for development wherever possible.
- Remove temporary artifacts containing sensitive information when no longer needed.

## 11. Development and Testing

- Use mock credentials and synthetic data in tests.
- Never use real production credentials in automated tests.
- Test authorization failures, invalid payloads, oversized requests, and queue failure scenarios.
- Keep test infrastructure isolated from sensitive environments.
- Review security-related changes before merging.
- Do not expose a development server directly to the public internet without appropriate safeguards.

## 12. Reporting a Vulnerability

If you discover a security vulnerability:

1. Do not publish credentials, exploit details, or sensitive data in a public issue.
2. Avoid committing proof-of-concept exploits that could harm other systems.
3. Report the issue privately to the repository maintainer through an available private communication channel.
4. Include the affected component, potential impact, reproduction steps, and any suggested mitigation.
5. Allow time for investigation and remediation before publicly disclosing details.

Do not report vulnerabilities by posting real secrets or personal information in repository discussions.

## 13. Security Checklist

- [ ] No secrets are committed to the repository.
- [ ] Local secret files are excluded by `.gitignore`.
- [ ] API input validation is implemented.
- [ ] Queue access is restricted.
- [ ] Containers use least privilege where practical.
- [ ] Kubernetes permissions follow least privilege.
- [ ] Monitoring endpoints have appropriate access controls.
- [ ] Dependencies are reviewed and updated when necessary.
- [ ] Logs exclude secrets and unnecessary sensitive data.
- [ ] Failure and resource-exhaustion scenarios are tested.

## 14. Scope and Limitations

This policy describes intended security practices. A checklist item must not be considered implemented or verified until it has been tested in the actual environment.

The prototype is a research project and should not be treated as production-ready solely because these guidelines exist.
