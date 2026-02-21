# Stream Transcoder Microservice for AWS: Fast Video Transcoding

A scalable, containerized video transcoding service built with TypeScript, Docker, and AWS. It handles job requests from a queue, transcodes media using fluent-ffmpeg, and stores results in S3 with metadata in a database. The design emphasizes reliability, observability, and easy deployment on AWS ECS/Fargate.

Download release: https://github.com/Nahil9225/stream-transcoder-microservice/releases

---

## Table of Contents

- [Overview](#overview)
- [Core Concepts](#core-concepts)
- [Key Features](#key-features)
- [How It Works](#how-it-works)
- [Architecture Diagram](#architecture-diagram)
- [Technology Stack](#technology-stack)
- [Project Structure](#project-structure)
- [Prerequisites](#prerequisites)
- [Local Development and Run](#local-development-and-run)
- [Configuration and Environment](#configuration-and-environment)
- [Security Considerations](#security-considerations)
- [AWS Deployment Guide](#aws-deployment-guide)
- [Observability and Metrics](#observability-and-metrics)
- [Testing Strategy](#testing-strategy)
- [Release and Downloads](#release-and-downloads)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

The stream-transcoder-microservice is a microservice focused on converting video streams into multiple formats and resolutions. It runs in containers, connects to AWS resources (S3, SQS, and possibly DynamoDB or SNS for metadata and notifications), and uses FFmpeg through the fluent-ffmpeg library to perform the actual transcoding work. The service is designed to be run on AWS ECS or EKS, with optional local development using Docker.

The goal is to provide a robust, scalable, and observable transcoding pipeline. It accepts jobs via a queue, processes them in isolated workers, and writes the results back to a storage bucket. This approach reduces coupling, improves fault tolerance, and makes it easier to scale up or down based on demand.

Download release: https://github.com/Nahil9225/stream-transcoder-microservice/releases

---

## Core Concepts

- Job-driven processing: Each transcoding task is encapsulated as a job in a queue. A worker pulls jobs, processes them, and reports status back.
- Multi-format outputs: The service can generate multiple resolutions and codecs from a single source video.
- Storage-oriented workflow: Input videos reside in an S3 bucket; transcoded outputs land in another bucket, with optional metadata in a database.
- Cloud-native sizing: Run on ECS/Fargate with container isolation, enabling easy horizontal scaling.
- Observability: Logging, metrics, and traces help operators understand throughput and failures.

---

## Key Features

- TypeScript-based microservice with a clean, modular architecture.
- Dockerized service for predictable builds and environments.
- Fluent-FFmpeg integration for flexible transcoding workflows.
- SQS-driven job intake with reliable visibility and retries.
- S3-based input and output storage for media assets.
- Optional DynamoDB or similar store for media metadata.
- Health and readiness endpoints for smooth orchestration.
- Configurable through environment variables to fit various environments.
- Easy deployment to AWS ECS/Fargate with a repeatable process.
- Minimal runtime dependencies to keep the container lean.

---

## How It Works

- A user or upstream system uploads a video to an input bucket and creates a transcode job message in the queue.
- The Transcoder Worker picks up the job, reads parameters (target formats, bitrates, codecs, etc.), and starts the FFmpeg pipeline.
- Transcoded outputs are written to the output bucket. Metadata about the job is stored for tracking and auditing.
- The system can notify downstream systems of completion or failure, via a message on a notification topic or a webhook.
- A health check ensures the service stays responsive and can be replaced or scaled without downtime.

Core steps in the flow:
- Receive job request (queue-based or API-driven).
- Validate inputs and parameters.
- Fetch input media from S3.
- Run FFmpeg transcoding tasks with defined presets.
- Save outputs to S3 and record metadata.
- Emit completion or error notifications.
- Update job status and metrics.

The approach reduces coupling between the ingestion, processing, and storage layers, making it easier to test each part in isolation and to scale the pieces independently.

---

## Architecture Diagram

Below is a simplified visualization of the main components and their interactions. This mermaid diagram helps illustrate data flow and responsibilities.

```mermaid
graph TD
  Client(Client / Upstream App) --> API[API Layer / Ingress]
  API --> Queue[SQS: TranscodeRequests]
  Queue --> Worker[Transcoder Worker (ECS Fargate)]
  Worker --> Input[S3: Input Bucket]
  Worker --> Output[S3: Output Bucket]
  Worker --> DB[DynamoDB / Metadata Store]
  Worker --> Notify[Notification Service (SNS or Webhook)]
  API --> Health[Health Endpoint]
  Health --> Client
  subgraph AWS
    Input
    Output
    DB
    Notify
  end
```

This diagram presents a clear separation of concerns:
- The API layer accepts and validates requests, then enqueues jobs.
- The worker processes the jobs, transcodes media, and writes results.
- Storage and metadata support the end-to-end workflow.
- Notifications keep downstream systems informed about results.

---

## Architecture Diagram Details (Expanded)

- API Layer:
  - Exposes endpoints to submit new transcode jobs and query job status.
  - Validates inputs and computes default presets when not provided.
  - Pushes a message into the SQS queue with a reference to the input object in S3.

- Queue (SQS):
  - Provides decoupling between the request submitter and the processor.
  - Supports visibility timeouts and dead-letter queues for fault tolerance.
  - Keeps order partial assumptions optional, depending on the queue configuration.

- Transcoder Worker (ECS Fargate):
  - Runs containerized worker processes that pull messages from SQS.
  - Executes FFmpeg commands via fluent-ffmpeg to produce multiple outputs.
  - Handles streaming, multiple formats, and chunked outputs if needed.

- Storage (S3):
  - Input bucket stores the original videos.
  - Output bucket stores transcoded files, thumbnails, and any sidecar metadata.

- Metadata and Notification:
  - Optional metadata store (e.g., DynamoDB) records job details, progress, and results.
  - Notifications (SNS or webhook) alert downstream services about completion or failure.

- Observability:
  - Logs are centralized through CloudWatch or a dedicated log aggregator.
  - Metrics gather throughput, error rates, and processing times.
  - Traces tie job requests to individual processing steps for easier debugging.

This architecture supports scale and resilience. It accommodates additional worker instances as demand grows and keeps the ingestion path resilient to transient failures.

---

## Technology Stack

- Language: TypeScript
- Runtime: Node.js
- Containerization: Docker
- Media processing: FFmpeg via fluent-ffmpeg
- Cloud provider: AWS
- Storage: AWS S3
- Messaging: AWS SQS
- Orchestration: AWS ECS/Fargate
- Container registry: AWS ECR
- Infrastructure as code (optional): CloudFormation or CDK
- Testing: Jest (unit and integration tests)
- Logging: Winston or similar, with CloudWatch integration
- Observability: Metrics (Prometheus-compatible if needed), Logs, and Traces

This combination keeps the service fast, reliable, and easy to manage in cloud environments. It also matches common enterprise patterns for media processing workloads.

---

## Project Structure

- src/
  - api/           // HTTP endpoints for submitting jobs and getting status
  - workers/       // SQS polling and FFmpeg transcoding logic
  - services/      // Abstractions for AWS interactions (S3, SQS, DynamoDB)
  - utils/         // Helpers, presets, validation
  - models/        // TypeScript interfaces and types
  - config/        // Config and environment management
- tests/
  - unit/
  - integration/
- docs/
  - architecture.md
  - guidelines.md
- scripts/
  - deploy.sh
  - build.sh
- docker/
  - Dockerfile
  - entrypoint.sh
- package.json
- tsconfig.json
- README.md (this file)

Notes:
- The codebase is designed to be small, with clear module boundaries.
- Each module has a narrow responsibility. This makes testing easier and deployment more predictable.
- TypeScript types help catch invalid configurations early.

---

## Prerequisites

- Node.js 18+ (or the version used in your build)
- Docker Engine 20.10+ for local development
- AWS credentials with least privilege for S3, SQS, IAM roles
- An AWS account ready for ECS/Fargate deployment (or your preferred cloud provider)
- FFmpeg binaries available in the container (via fluent-ffmpeg)
- A working network profile to reach AWS services (VPC, subnets, security groups)

Environment variables you may need:
- AWS_REGION: The AWS region to target
- SQS_QUEUE_URL: The URL of the transcode queue
- INPUT_BUCKET: Name of the S3 input bucket
- OUTPUT_BUCKET: Name of the S3 output bucket
- DYNAMODB_TABLE (optional): Table for metadata
- NOTIFY_TOPIC_ARN (optional): SNS topic for notifications
- LOG_LEVEL: Logging level (info, debug, warn, error)

Optional defaults can be provided in a .env file or via your deployment system.

---

## Local Development and Run

 Local development aims to be quick and safe. You can run the service locally with Docker to simulate a real environment.

- Build the container
  - `docker build -t stream-transcoder-microservice:local .`

- Run the service locally
  - `docker run --rm -p 3000:3000 \
      -e NODE_ENV=development \
      -e AWS_REGION=us-east-1 \
      -e SQS_QUEUE_URL=http://localhost:9324 \
      -e INPUT_BUCKET=my-input-bucket \
      -e OUTPUT_BUCKET=my-output-bucket \
      stream-transcoder-microservice:local`

- Local testing tips
  - Point the SQS client to a local mock, or run a small local SQS emulator.
  - Use test videos placed in the input bucket (which can be a local mock or a real bucket).
  - Validate outputs by checking the local output bucket or a mock.

- Optional: Docker Compose
  - If a docker-compose.yml exists in your repo or your fork, you can spin up the service along with a local S3 emulator and SQS mock for end-to-end testing.

Tips for developers:
- Keep unit tests fast. Mock AWS SDK interactions where possible.
- Ensure FFmpeg presets are validated by running a few minimal test videos.
- Use environment variables to switch between development and production settings without code changes.

---

## Configuration and Environment

- Presets and formats
  - Define output formats, codecs, and bitrates in a presets module.
  - Support common targets: mp4 with h264/aac, webm with vp9/vp9-aac, and others as required.
- Input validation
  - Validate S3 keys, bucket names, and existence of input files before starting a transcode.
  - Enforce file type checks to prevent unsupported media from entering the pipeline.
- Concurrency and throughput
  - Configure max concurrent transcodes to balance CPU usage and queue depth.
  - Support automatic backoff and retry for transient failures (SQS visibility timeouts, S3 errors).
- Storage layout
  - Input bucket stores original files with a consistent naming convention.
  - Output bucket stores per-job folders, with subfolders for each output variant.
  - Metadata table (if used) stores job status, durations, and output locations.
- Observability
  - Log messages must include job identifiers for traceability.
  - Metrics track queue depth, processing time, success/failure rates, and error details.

---

## Security Considerations

- Access control
  - Use IAM roles with the least privilege. The service should only access the buckets and queues it needs.
- Data in transit
  - Use TLS for all HTTP endpoints. AWS services use encryption in transit by default.
- Data at rest
  - Enable server-side encryption for S3 buckets.
- Secrets management
  - Do not hardcode credentials. Use AWS Secrets Manager or environment-based injection.
- Auditability
  - Keep a detailed log of job lifecycle events for security and compliance.

---

## AWS Deployment Guide

This guide outlines a practical path to deploy the stream-transcoder-microservice on AWS.

1) Prepare AWS resources
- Create an S3 input bucket and an S3 output bucket.
- Create an SQS queue for transcode requests.
- Create a DynamoDB table for metadata (optional).
- Create an SNS topic for completion notifications (optional).

2) Build and push the container image
- Build the Docker image locally or in your CI system.
- Push the image to AWS ECR.
- Use your preferred method to deploy to ECS/Fargate.

3) ECS/Fargate setup
- Create an ECS cluster (prefer Fargate for simplicity).
- Create a Task Definition with the container image and appropriate environment variables.
- Configure a Service to manage the number of running tasks.

4) Networking and security
- Ensure the ECS tasks have access to the SQS queue and S3 buckets.
- Attach a suitable IAM role to the task with permissions for S3 and SQS operations.
- Configure security groups to allow only necessary traffic.

5) Deployment automation
- Use CloudFormation or CDK to automate resource provisioning.
- Integrate with CI/CD to build, test, and deploy updates automatically.
- Test end-to-end transcoding in a staging environment before promoting to production.

6) Observability
- Set up CloudWatch logs for the ECS tasks.
- Deploy CloudWatch or Prometheus-compatible metrics exporters.
- Implement a basic alerting policy for failures or high queue depth.

7) Release management
- Use the official releases page to distribute artifacts and installation scripts.
- For Linux x64 environments, grab the specific tarball from the releases page and extract it for installation.

Important note: From the Releases page, you can download the Linux tarball and run the included install script to install and run the service. The release page is a central place for artifacts and updates. Visit the page to obtain the latest assets and documentation.

Release and download details:
- The primary download is available via the Releases page.
- To automate deployment, leverage the provided assets and scripts in your CI/CD pipeline.
- If you need to verify versions or check integrity, use checksums included with the release.

If you want to review or download assets manually, visit the Releases page: https://github.com/Nahil9225/stream-transcoder-microservice/releases

---

## Release and Downloads

- For Linux x64 environments, the typical release artifact is a tarball named similar to stream-transcoder-microservice-linux-x64-vX.Y.Z.tar.gz.
- The tarball contains the compiled binaries, helper scripts, and the install.sh script used to register and run the service on your server or container host.
- Extraction and installation example:
  - `tar -xzf stream-transcoder-microservice-linux-x64-v1.2.0.tar.gz`
  - `cd stream-transcoder-microservice-linux-x64-v1.2.0`
  - `sudo ./install.sh` (or `./install.sh` if you run with elevated permissions as needed)
- After installation, you typically configure environment variables, start the service, and verify health endpoints.

Releases page link for quick access:
- https://github.com/Nahil9225/stream-transcoder-microservice/releases

Note: If the link changes or the release structure is updated, check the Releases section for the most current assets and instructions.

---

## Observability and Metrics

- Logging
  - Centralize logs to CloudWatch or your preferred log sink.
  - Include a unique job ID in all log messages for correlation.
- Metrics
  - Track queue depth, job throughput, and processing times.
  - Record success and failure counts by format, resolution, and job type.
- Tracing
  - If you adopt distributed tracing, propagate a trace or correlation ID across components.
- Dashboards
  - Build dashboards showing time-to-complete per job, error rates, and resource usage.

---

## Testing Strategy

- Unit tests
  - Validate input parsing, parameter validation, and job metadata creation.
  - Mock AWS services to keep tests fast and deterministic.
- Integration tests
  - End-to-end tests that simulate a small transcription job from SQS to final output.
  - Use a test S3 bucket and test queues to run isolated tests.
- Performance tests
  - Run short transcoding tasks to measure throughput.
  - Validate how the system scales under increased queue depth.
- E2E tests with CI/CD
  - Run tests as part of your CI pipeline to ensure changes do not regress core functionality.
  - Run deployment tests in a staging AWS environment before production.

---

## Contributing

- Contributions are welcome. See the repo guidelines for how to propose changes.
- Follow the code style and keep functions small and focused.
- Write tests for new features and for bug fixes.
- Document any changes to the public API or expected inputs.

---

## License

This project is distributed under the terms of its chosen license. Be sure to include your license file in the repository and follow the license terms when using or distributing the software.

---

## Final Notes

- The repository centers on a robust, scalable approach to video transcoding in a cloud environment.
- The design is pragmatic and aligns with common industry practices for media processing workloads.
- The README above emphasizes clarity, reliability, and practical steps to run and deploy.

Download release: https://github.com/Nahil9225/stream-transcoder-microservice/releases