# Image Building Security Practices
Focus on creating secure, minimal container images from the start. 
Use Minimal, Trusted Base Images: Start with official, verified images (e.g., alpine, distroless) to reduce the attack surface by avoiding unnecessary software and libraries.
Good practice:
````
FROM node:18.0.1-alpine
...
````
Not good practice:
````
FROM ubuntu
RUN apt-get update && apt-get install -y \
    node \
    rm -rf /var/lib/apt/lists/*
````


Run as a Non-Root User: By default, if not set a USER in your Dockerfile, the user will default to root, a major security risk. Create a dedicated non-root user within your Dockerfile using RUN command and switch to it using the USER instruction.

````
RUN useradd -ms /bin/bash myuser
USER myuser
````

Avoid Leaking Secrets: Never store secrets (passwords, keys) in the Dockerfile itself or commit them to image layers. Use Docker Secrets or environment variables injected at runtime, or leverage multi-stage builds to ensure sensitive data is not included in the final image.


Use COPY instead of ADD: The COPY instruction is more predictable and less error-prone. ADD can automatically extract compressed files or fetch remote URLs, which introduces potential "zip bomb" or man-inthe-middle vulnerabilities.

````
COPY ./app /usr/src/app
COPY requirements.txt /usr/src/app/
````

Scan Images for Vulnerabilities: Integrate vulnerability scanners (like Docker Scout, Trivy, or Snyk) into your CI/CD pipeline to identify and remediate known flaws early in the development process.

````
jobs:
  security-snyk:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@master
      - name: Run Snyk to check for vulnerabilities
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
        with:
          args: --severity-threshold=high
````

# Daemon and Host Security

Secure the underlying infrastructure running your containers. 
Keep Everything Updated: Regularly update the Docker engine, the host operating system, and the kernel to protect against known vulnerabilities and container escape exploits.

Do Not Expose the Docker Daemon Socket: Exposing the /var/run/docker.sock to containers is equivalent to granting root access to the host. Use a restricted proxy or run Docker in rootless mode instead.


Enforce Access Controls: Use strict access controls (IAM, MFA, RBAC) for your registries and host systems to prevent unauthorized access and image tampering. 

Runtime Security
Implement the principle of least privilege and monitor container activity. 
Limit Container Privileges and Capabilities: Run containers with the minimum required permissions. Drop unnecessary Linux capabilities using the --cap-drop flag and avoid using the --privileged flag.
Use a Read-Only Filesystem: Run containers with the --read-only flag to prevent attackers from modifying the filesystem, forcing logs and temporary files to be written to controlled volumes.
Set Resource Limits: Limit CPU and memory usage using flags like --memory and --cpus to prevent a compromised container from monopolizing host resources (Denial-of-Service attacks).
Restrict Networking: Only expose the ports that are absolutely necessary for the application to function. Use network segmentation and firewall rules (like iptables) to control traffic flow.
Monitor and Log Activity: Use Docker's logging drivers and tools like Prometheus, Grafana, or Falco to monitor container activity in real-time and set up alerts for suspicious behavior.
Use Security Frameworks: Leverage built-in Linux Security Modules (LSM) like AppArmor or Seccomp to enforce mandatory access controls and restrict container system calls. 