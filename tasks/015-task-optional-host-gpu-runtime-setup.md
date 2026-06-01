# Task: Optional Host GPU Runtime Setup

Type: Task
Status: Not Started
Optional: Yes

## Goal

Enable reliable GPU execution for the classifier container on developer or deployment hosts that should run the GPU variant.

## Context

- `sentir-mais-backend/docker-compose.yml` now requests GPU access for the classifier service.
- Running the compose stack produced this host-level Docker error:

```text
Error response from daemon: failed to discover GPU vendor from CDI: no known GPU vendor found
```

- This is not an application code problem. It indicates that the host Docker/NVIDIA runtime stack is not fully configured for GPU containers.

## Why This Is Separate

- This work is environment and infrastructure specific.
- It should not block application development on CPU-only machines.
- It may differ across Linux distributions, Docker installations, and GPU driver versions.

## Scope

- Verify host NVIDIA driver installation
- Verify `nvidia-smi` works on the host
- Install and configure NVIDIA Container Toolkit
- Verify Docker GPU access using an isolated test container
- If CDI is used, generate and validate the CDI spec
- Document the final host setup steps for future environments

## Suggested Validation Steps

Host checks:

```bash
nvidia-smi
docker run --rm --gpus all ubuntu nvidia-smi
```

If CDI is required:

```bash
sudo nvidia-ctk cdi generate --output=/etc/cdi/nvidia.yaml
nvidia-ctk cdi list
```

Possible explicit NVIDIA runtime validation:

```bash
docker run --rm -ti --runtime=nvidia -e NVIDIA_VISIBLE_DEVICES=all ubuntu nvidia-smi -L
```

## Deliverables

- Working GPU container access on the target host
- Short environment setup notes in project docs or infra docs
- Clear fallback guidance for CPU-only hosts

## Acceptance Criteria

- The classifier container can start with GPU access on a configured host.
- The setup is documented well enough to reproduce on another similar host.
