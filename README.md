# azure-devops-pipeline-templates

A reference library of reusable Azure DevOps YAML templates: build once, promote the same artifact through environments, deploy feature branches to dedicated App Service slots, and verify every deployment.

> Status: reference implementation with synthetic names (`contoso-api`). It is written from scratch and is not based on any employer's or client's pipelines. It has not been run against a live Azure DevOps organization, so expect to adjust service connection names and environment settings.

## What this demonstrates

- **Template library, thin pipelines.** Applications call shared job, step and variable templates with a few parameters.
- **Build once, deploy many.** CI publishes one versioned artifact; release pipelines consume it through a `resources: pipelines` trigger.
- **Feature branch to slot mapping.** `feature-s1` to `feature-s7` deploy to slots `s1` to `s7` for isolated pre-merge testing.
- **Layered variables.** Shared plus per-environment values, with approvals attached to Azure DevOps environments.
- **Self-hosted agent selection by demand** for production.
- **Fail-fast verification** after every deployment.

## Layout

```text
templates/
  jobs/        build-dotnet, build-python, deploy-appservice-zip
  steps/       verify-health
  variables/   common, dev, prod
pipelines/
  contoso-api/ ci.yml, release.yml, feature-slot.yml   (example consumers)
docs/design.md  principles, flow diagram, template reference
docs/adr/0006-...  decision record: build once, deploy many, feature slots
```

## Use it

1. Create three pipelines from `pipelines/contoso-api/`: `contoso-api-ci`, `contoso-api-release`, `contoso-api-feature-slot`. The `source:` value in the release files must match the CI pipeline's name.
2. Create service connections `sc-contoso-dev` and `sc-contoso-prod` (or change the names).
3. Create environments `contoso-dev`, `contoso-feature`, `contoso-prod` and add an approval check to prod.
4. Push to `main` for dev then prod; push `feature-s1` for slot `s1`.

## Design in one paragraph

Pipelines stay small because they only pass parameters. CI builds one artifact. Release stages download exactly that artifact, so promotion cannot introduce a different build. Environment differences live in variable layers and Azure DevOps environments. Slots give each feature branch its own live URL.

See [docs/design.md](docs/design.md).

## License

MIT
