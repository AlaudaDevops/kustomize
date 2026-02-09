# Kustomize alauda Branch Development Guide

## Background

Previously, kustomize was used as a general-purpose CLI across multiple plugins, each needing to fix vulnerabilities in kustomize independently.

To avoid duplicated efforts, we forked the [kustomize](https://github.com/kubernetes-sigs/kustomize) repository and maintain it through branches named `alauda-vx.xx.xx`.

We use [renovate](https://gitlab-ce.alauda.cn/devops/tech-research/renovate/-/blob/main/docs/quick-start/0002-quick-start.md) to automatically fix vulnerabilities in corresponding versions.

## Repository Structure

Based on the original code, the following content has been added:

- [alauda-auto-tag.yaml](./.github/workflows/alauda-auto-tag.yaml): Automatically tags and triggers goreleaser when a PR is merged into the `alauda-vx.xx.xx` branch
- [release-alauda.yaml](./.github/workflows/release-alauda.yaml): Supports triggering goreleaser manually or upon tag updates (this pipeline isn't triggered when tags are created by actions due to GitHub Actions design limitations)
- [reusable-release-alauda.yaml](./.github/workflows/reusable-release-alauda.yaml): Executes goreleaser to create a release
- [scan-alauda.yaml](.github/workflows/scan-alauda.yaml): Runs trivy vulnerability scans (`rootfs` scans for Go binaries)
- [.goreleaser-alauda.yml](.goreleaser-alauda.yml): Configuration file for releasing alauda versions

## Special Modifications

None at present

## Pipelines

### Triggered on PR Submission

- [go.yml](.github/workflows/go.yml): Official testing pipeline including unit tests, integration tests, etc.

### Triggered on Merge to alauda-vx.xx.xx Branch

- [alauda-auto-tag.yaml](.github/workflows/alauda-auto-tag.yaml): Automatically tags and triggers goreleaser
- [reusable-release-alauda.yaml](.github/workflows/reusable-release-alauda.yaml): Executes goreleaser to create a release (triggered by `alauda-auto-tag.yaml`)

### Scheduled or Manual Triggering

- [scan-alauda.yaml](.github/workflows/scan-alauda.yaml): Runs trivy vulnerability scans (`rootfs` scans for Go binaries)

### Others

Other officially maintained pipelines remain unchanged; some irrelevant pipelines have been disabled on the Actions page.

## Renovate Vulnerability Fix Mechanism

The renovate configuration file is [renovate.json](https://github.com/AlaudaDevops/kustomize/blob/main/renovate.json)

1. renovate detects vulnerabilities in the branch and submits a PR for fixes
2. Tests run automatically on the PR
3. After all tests pass, renovate automatically merges the PR
4. After the branch updates, an action automatically tags the commit (e.g., v5.8.1-alauda-0, with patch version and last digit incremented)
5. goreleaser automatically publishes a release based on the tag

## Maintenance Plan

When upgrading to a new version, follow these steps:

1. Create an alauda branch from the corresponding tag, e.g., tag `v5.8.0` corresponds to branch `alauda-v5.8.0`
2. Cherry-pick previous alauda branch changes onto the new branch and push

Renovate automatic fix mechanism:
1. After renovate submits a PR, pipelines run automatically; if all tests pass, the PR will be merged automatically
2. After merging into the `alauda-v5.8.0` branch, goreleaser will automatically create a `v5.8.1-alauda-0` release (note: not `v5.8.0-alauda-0`, because upgrading the version allows renovate to recognize it)
3. renovate configured in other plugins will automatically fetch artifacts from the release according to its configuration
