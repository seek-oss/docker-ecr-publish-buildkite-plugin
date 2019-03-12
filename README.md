# Docker ECR Publish Buildkite Plugin

[![Build Status](https://img.shields.io/github/release/seek-oss/docker-ecr-publish-buildkite-plugin.svg)](https://github.com/seek-oss/docker-ecr-publish-buildkite-plugin/releases)

A [Buildkite plugin](https://buildkite.com/docs/agent/v3/plugins) to build, tag,
and push Docker images to Amazon ECR.

## Example

The following pipeline builds the default `./Dockerfile` and pushes it to a
pre-existing ECR repository `my-repo`:

```yaml
steps:
  - plugins:
      - seek-oss/docker-ecr-publish#v1.1.4:
          ecr-name: my-repo
```

An alternate Dockerfile may be specified:

```yaml
steps:
  - plugins:
      - seek-oss/docker-ecr-publish#v1.1.4:
          dockerfile: path/to/final.Dockerfile
          ecr-name: my-repo
```

[Build-time
variables](https://docs.docker.com/engine/reference/commandline/build/#set-build-time-variables---build-arg)
are supported, either with an explicit value, or without one to propagate an
environment variable from the pipeline step:

```yaml
steps:
  - plugins:
      - seek-oss/docker-ecr-publish#v1.1.4:
          args:
            - BUILDKITE_BUILD_NUMBER # propagate environment variable
            - ENVIRONMENT=prod # explicit value
          ecr-name: my-repo
```

Images triggered from the default branch are tagged with `latest` and `master`,
while images triggered from any other branch are tagged with `branch`.
Additional tags may be listed:

```yaml
steps:
  - plugins:
      - seek-oss/docker-ecr-publish#v1.1.4:
          branch-tags:
            # - branch
            - $BUILDKITE_BUILD_NUMBER
          default-tags:
            # - latest
            # - master
            - production
          ecr-name: my-repo
```

This plugin can be used in combination with the [Create
ECR](https://github.com/seek-oss/create-ecr-buildkite-plugin) plugin to fully
manage an ECR application repository within one pipeline step:

```yaml
steps:
  - plugins:
      - seek-oss/create-ecr#v1.1.2:
          name: my-repo
      - seek-oss/docker-ecr-publish#v1.1.4:
          ecr-name: my-repo
```

## Configuration

- `args` (required, array|string):

  build-args to pass into docker build.

- `branch-tags` (optional, array)

  Tags to push on a non-default branch build.

- `default-tags` (optional, array)

  Tags to push on a default branch build.

- `dockerfile` (optional, string)

  Local path to a custom Dockerfile.

  Default: `Dockerfile`

- `ecr-name` (required, string)

  Name of the ECR repository.

## License

MIT (see [LICENSE](LICENSE))
