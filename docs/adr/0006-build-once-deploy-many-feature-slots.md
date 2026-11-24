# ADR-0006: Build once, deploy many with shared YAML templates and feature-branch slots

- **Status:** Proposed
- **Date:** 2026-11-23
- **Decision owner:** Avinash Padaki
- **Jira:** KAN-17
- **Repo / PR:** azure-devops-pipeline-templates (PR pending)

## Context

Application teams need consistent build and deployment pipelines without copying YAML into every repository. Rebuilding per environment means the artifact that was tested is not the artifact that ships, and developers need isolated environments to test a branch before merging.

## Decision

1. Keep application pipelines thin. Build, deploy and verification logic lives in shared templates (`templates/jobs`, `templates/steps`, `templates/variables`).
2. Build once: CI publishes one versioned artifact. Release pipelines consume it through a `resources: pipelines` reference and never rebuild it.
3. Layer configuration: `common.yml` plus one variables file per environment. Approvals and checks live on Azure DevOps environments, not in pipeline logic.
4. Map feature branches `feature-s1` to `feature-s7` to App Service deployment slots `s1` to `s7`. The resolve job fails the run if a branch does not match the pattern.
5. End every deployment with a health check that fails the job unless the target returns HTTP 200.
6. Use literal service connection names and, for production, a self-hosted agent pool selected by demand.

## Alternatives considered

- Classic release pipelines: not versioned as code and harder to review.
- Rebuild per environment: simpler pipelines, but the tested artifact is not the shipped artifact.
- One pipeline for build and every stage: couples CI to CD and makes feature slot deployment awkward.
- Dynamic slot or environment per pull request: stronger isolation, but higher cost and a cleanup problem that a fixed slot count avoids.

## Consequences

**Benefits:** Promotion cannot introduce a different build; new applications need a few lines of YAML; every branch gets a live URL for testing; failed deployments stop themselves.

**Costs and risks:** A fixed slot count limits parallel feature branches to seven. Slots share the App Service plan's resources. The pipelines are coupled by name through the `source:` value. Template changes affect every consumer, so consumers should pin a version.

## Non-goals

Infrastructure provisioning, secrets, container or Kubernetes deployment, slot cleanup automation.

## Validation criteria

In a scratch Azure DevOps project: the artifact hash is identical in dev and prod; a push to `feature-s3` deploys to slot `s3`; a branch that does not match the pattern fails; a failing health check fails the job; a production run waits for the environment approval.
