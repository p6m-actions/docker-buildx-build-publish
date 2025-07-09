# Docker Buildx Build Publish

![Latest Release](https://img.shields.io/github/v/release/p6m-actions/docker-buildx-build-publish?style=flat-square&label=Latest%20Release&color=blue)

## Description

A generic GitHub Action for building and publishing multi-platform Docker images using Docker Buildx. This action provides comprehensive support for modern Docker build features including secrets, caching, multi-platform builds, and flexible output options.

## Usage

```yaml
- name: Build and publish Docker image
  uses: p6m-actions/docker-buildx-build-publish@v1
  with:
    image-name: my-app
    image-tag: ${{ github.sha }}
    registry: ghcr.io
    platforms: linux/amd64,linux/arm64
    secrets: |
      username=${{ secrets.REGISTRY_USERNAME }}
      password=${{ secrets.REGISTRY_PASSWORD }}
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `dockerfile-path` | Path to the Dockerfile | No | `Dockerfile` |
| `context-path` | Build context path | No | `.` |
| `image-name` | Docker image name | Yes | - |
| `image-tag` | Docker image tag | No | `latest` |
| `registry` | Container registry URL | No | - |
| `push` | Whether to push the image to registry | No | `true` |
| `load` | Load the built image into local Docker daemon | No | `false` |
| `build-args` | Build arguments (multiline string) | No | - |
| `platforms` | Target platforms for multi-platform build | No | `linux/amd64` |
| `secrets` | Build secrets (multiline string, format: `id=value`) | No | - |
| `outputs` | Output configuration (e.g., `type=docker,dest=./image.tar`) | No | - |
| `labels` | Metadata labels for the image (multiline string) | No | - |
| `annotations` | Manifest annotations (multiline string) | No | - |
| `cache-from` | Cache import configuration | No | `type=gha` |
| `cache-to` | Cache export configuration | No | `type=gha,mode=max` |
| `skip-setup` | Skip Docker Buildx setup (use when already configured) | No | `false` |
| `builder-name` | Name of the buildx builder to use | No | - |

## Outputs

| Output | Description |
|--------|-------------|
| `image-digest` | The digest of the built image |
| `image-metadata` | Build result metadata |
| `image-uri` | Full image URI with registry and tag |

## Examples

### Basic Usage
```yaml
- name: Build Docker image
  uses: p6m-actions/docker-buildx-build-publish@v1
  with:
    image-name: my-app
    image-tag: latest
```

### Advanced Setup with Companion Action
For complex workflows requiring custom Buildx configuration, use with [docker-buildx-setup](https://github.com/p6m-actions/docker-buildx-setup):

```yaml
- name: Set up Docker Buildx
  id: buildx
  uses: p6m-actions/docker-buildx-setup@v1

- name: Build and push Docker image
  uses: p6m-actions/docker-buildx-build-publish@v1
  with:
    image-name: my-app
    image-tag: ${{ github.sha }}
    registry: ghcr.io
    platforms: linux/amd64,linux/arm64
    skip-setup: true
    builder-name: ${{ steps.buildx.outputs.name }}
```

> **Note**: When using the setup action, set `skip-setup: true` to avoid duplicate setup steps.

### Multi-platform Build with Registry
```yaml
- name: Build and push multi-platform image
  uses: p6m-actions/docker-buildx-build-publish@v1
  with:
    image-name: my-app
    image-tag: ${{ github.sha }}
    registry: ghcr.io
    platforms: linux/amd64,linux/arm64
    push: true
```

### With Build Secrets
```yaml
- name: Build with secrets
  uses: p6m-actions/docker-buildx-build-publish@v1
  with:
    image-name: my-app
    registry: ghcr.io
    secrets: |
      username=${{ secrets.ARTIFACTORY_USERNAME }}
      password=${{ secrets.ARTIFACTORY_TOKEN }}
```

### With Build Arguments and Labels
```yaml
- name: Build with args and labels
  uses: p6m-actions/docker-buildx-build-publish@v1
  with:
    image-name: my-app
    build-args: |
      VERSION=${{ github.sha }}
      BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ')
    labels: |
      org.opencontainers.image.version=${{ github.sha }}
      org.opencontainers.image.source=${{ github.server_url }}/${{ github.repository }}
```

### Load Image Locally
```yaml
- name: Build and load image
  uses: p6m-actions/docker-buildx-build-publish@v1
  with:
    image-name: my-app
    push: false
    load: true
```