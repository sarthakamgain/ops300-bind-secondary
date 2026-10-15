# OPS300 Bind Secondary CI/CD Pipeline

This repository contains the Dockerfile, BIND configuration, and Jenkins declarative pipeline used to build the bind-secondary container image.

## CVE Scanning

The Jenkins pipeline includes a Trivy vulnerability scanning stage before the image is pushed to the private registry.

The scan checks for HIGH and CRITICAL vulnerabilities. Scan results are saved as `trivy-report.txt` and archived as a Jenkins build artifact.

For this assignment, the pipeline is configured to continue after the scan instead of failing automatically on detected vulnerabilities. This was chosen so the vulnerability results can be preserved and reviewed while still demonstrating the complete build-and-push workflow.

Build #4 detected:
- 5 HIGH vulnerabilities
- 0 CRITICAL vulnerabilities

The image was scanned before being pushed to the private registry.
