# Docker ECR Publish Buildkite Plugin

[![GitHub Release](https://img.shields.io/github/release/seek-oss/docker-ecr-publish-buildkite-plugin.svg)](https://github.com/seek-oss/docker-ecr-publish-buildkite-plugin/releases)

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

Images built from the default branch are tagged with `latest`. Additional tags
may be listed, and are split between non-default branches and the default
branch:

```yaml
steps:
  - plugins:
      - seek-oss/docker-ecr-publish#v1.1.4:
          branch-tags:
            - $BUILDKITE_BUILD_NUMBER
          default-tags:
            # - latest
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

This plugin can be used in combination with the [Docker ECR
Cache](https://github.com/seek-oss/docker-ecr-cache-buildkite-plugin) plugin to
reuse a base image across pipeline steps:

```yaml
steps:
  - command: npm test
    plugins:
      - seek-oss/docker-ecr-cache#v1.1.1:
          target: deps
      - docker#v3.0.1:
          volumes:
            - /workdir/node_modules
  - plugins:
      - seek-oss/docker-ecr-cache#v1.1.1:
          target: deps
      - seek-oss/docker-ecr-publish#v2.0.0:
          cache-from: ecr://build-cache/my-org/my-repo
          name: my-repo
```

## Configuration

- `args` (optional, array|string):

  build-args to pass into docker build.

  Sensitive arguments should be propagated as an environment variable (`MY_ARG`
  instead of `MY_ARG=blah`), so that they are not checked into your source
  control and then logged to Buildkite output by this plugin.

- `branch-tags` (optional, array)

  Tags to push on a non-default branch build.

  Default: none (image is not pushed)

- `cache-from` (optional, array|string):

  Images for Docker to use as cache sources, e.g. a base or dependency image.

  Use standard Docker image notation (e.g. `debian:jessie`,
  `myregistry.local:5000/testing/test-image`), or the `ecr://my-repo` shorthand
  to point to an ECR repository in the current AWS account.

- `default-tags` (optional, array)

  Tags to push on a default branch build.

  Default: `latest` (this cannot be disabled)

- `dockerfile` (optional, string)

  Local path to a custom Dockerfile.

  Default: `Dockerfile`

- `ecr-name` (required, string)

  Name of the ECR repository.

## License

MIT (see [LICENSE](LICENSE))
