# Docker ECR Publish Buildkite Plugin

[![GitHub Release](https://img.shields.io/github/release/seek-oss/docker-ecr-publish-buildkite-plugin.svg)](https://github.com/seek-oss/docker-ecr-publish-buildkite-plugin/releases)

A [Buildkite plugin](https://buildkite.com/docs/agent/v3/plugins) to build, tag, and push Docker images to Amazon ECR.

## Example

The following pipeline builds the default `./Dockerfile` and pushes it to a pre-existing ECR repository `my-repo`:

```yaml
steps:
  - plugins:
      - seek-oss/docker-ecr-publish#v2.1.0:
          ecr-name: my-repo
```

An alternate Dockerfile may be specified:

```yaml
steps:
  - plugins:
      - seek-oss/docker-ecr-publish#v2.1.0:
          dockerfile: path/to/final.Dockerfile
          ecr-name: my-repo
```

[Build-time variables](https://docs.docker.com/engine/reference/commandline/build/#set-build-time-variables---build-arg) are supported,
either with an explicit value,
or without one to propagate an environment variable from the pipeline step:

```yaml
steps:
  - plugins:
      - seek-oss/docker-ecr-publish#v2.1.0:
          args:
            - BUILDKITE_BUILD_NUMBER # propagate environment variable
          branch-args:
            - BRANCH_TYPE=branch # explicit value
          default-args:
            - BRANCH_TYPE=default # explicit value
          ecr-name: my-repo
```

All images are tagged with their corresponding `$BUILDKITE_BUILD_NUMBER`.

Images built from the default branch are automatically tagged with `latest`.
Additional tags may be listed:

```yaml
steps:
  - plugins:
      - seek-oss/docker-ecr-publish#v2.1.0:
          branch-tags:
            - branch-$BUILDKITE_BUILD_NUMBER
          default-tags:
            # - latest
            - default-$BUILDKITE_BUILD_NUMBER
          ecr-name: my-repo
          tags:
            # - $BUILDKITE_BUILD_NUMBER
            - any-$BUILDKITE_BUILD_NUMBER
```

If you're working with [immutable image tags](https://docs.aws.amazon.com/AmazonECR/latest/userguide/image-tag-mutability.html),
you can disable the `latest` tag with the `add-latest-tag` property:

```yaml
steps:
  - plugins:
      - seek-oss/docker-ecr-publish#v2.1.0:
          add-latest-tag: false
          ecr-name: my-repo
```

More complex branch workflows can be achieved by using multiple pipeline steps with differing `branches`:

```yaml
steps:
  - branches: '!dev !prod'
    plugins:
      - seek-oss/docker-ecr-publish#v2.1.0:
          args: BRANCH_TYPE=branch
          ecr-name: my-repo
          tags: branch-$BUILDKITE_BUILD_NUMBER
  - branches: dev
    plugins:
      - seek-oss/docker-ecr-publish#v2.1.0:
          args: BRANCH_TYPE=dev
          ecr-name: my-repo
          tags: dev-$BUILDKITE_BUILD_NUMBER
  - branches: prod
    plugins:
      - seek-oss/docker-ecr-publish#v2.1.0:
          args: BRANCH_TYPE=prod
          ecr-name: my-repo
          tags: prod-$BUILDKITE_BUILD_NUMBER
```

Additional `docker build` arguments can be passed via the `additional-build-args` setting:

```yaml
steps:
  - command: 'echo amaze'
    env:
      DOCKER_BUILDKIT: '1'
    plugins:
      - seek-oss/docker-ecr-publish#v2.1.0:
          additional-build-args: '--progress=plain --ssh= default=\$SSH_AUTH_SOCK'
      - docker#v3.5.0
```

This plugin can be used in combination with the [Create ECR](https://github.com/seek-oss/create-ecr-buildkite-plugin) plugin to fully manage an ECR application repository within one pipeline step:

```yaml
steps:
  - plugins:
      - seek-oss/create-ecr#v1.1.2:
          name: my-repo
      - seek-oss/docker-ecr-publish#v2.1.0:
          ecr-name: my-repo
```

This plugin can be used in combination with the [Docker ECR Cache](https://github.com/seek-oss/docker-ecr-cache-buildkite-plugin) plugin to reuse a base image across pipeline steps:

```yaml
steps:
  - command: npm test
    plugins:
      - seek-oss/docker-ecr-cache#v1.7.0:
          ecr-name: my-cache
          target: deps
      - docker#v3.5.0:
          volumes:
            - /workdir/node_modules
  - plugins:
      - seek-oss/docker-ecr-cache#v1.7.0:
          ecr-name: my-cache
          target: deps
      - seek-oss/docker-ecr-publish#v2.1.0:
          cache-from: ecr://my-cache # defaults to latest tag
          ecr-name: my-repo
```

We can target registries in other accounts and regions, provided the current IAM user/role has the ability to auth against said account/registry:

```yaml
steps:
  - plugins:
      - seek-oss/docker-ecr-publish#v2.1.0:
          account_id: '12345678910'
          region: eu-west-1
          ecr-name: my-repo
```

## Configuration

- `args` (optional, array|string)

  Build args to provide to all builds. These are listed _before_ the
  branch-specific `branch-args` and `default-args` properties in the resulting
  `docker build` command.

  Sensitive arguments should be propagated as an environment variable (`MY_ARG`
  instead of `MY_ARG=blah`), so that they are not checked into your source
  control and then logged to Buildkite output by this plugin.

- `add-latest-tag` (optional, boolean)

  Whether to add a `latest` tag to default branch builds.

  Default: `true`

- `additional-build-args` (optional, string)

  Allows specifying additional arguments directly to the `docker build` command.

- `branch-args` (optional, array|string)

  Build args to provide to non-default branch builds.

- `branch-tags` (optional, array|string)

  Tags to push on non-default branch builds.

- `build-context` (optional, string)

  The Docker build context. Valid values are as per the [API](https://docs.docker.com/engine/reference/commandline/build/#extended-description)

  Default: `.`

- `cache-from` (optional, array|string)

  Images for Docker to use as cache sources, e.g. a base or dependency image.

  Use standard Docker image notation (e.g. `debian:jessie`,
  `myregistry.local:5000/testing/test-image`), or the `ecr://cache-repo:tag` shorthand
  to point to an ECR repository in the current AWS account.

- `default-args` (optional, array|string)

  Build args to provide to default branch builds.

- `default-tags` (optional, array|string)

  Tags to push on default branch builds.

  Default: `latest` (non-removable)

- `dockerfile` (optional, string)

  Local path to a custom Dockerfile.

  Default: `Dockerfile`

- `ecr-name` (required, string)

  Name of the ECR repository.

- `account_id` (optional, string)

  Account ID for ECR registry, defaults to output of `aws sts get-caller-identity` e.g. current account ID.

- `region` (optional, string)

  Region the ECR registry is in, defaults to `$AWS_DEFAULT_REGION` and then `eu-west-1` if not set.

- `tags` (optional, array|string)

  Tags to push on all builds.

  Default: `$BUILDKITE_BUILD_NUMBER` (non-removable)

- `target` (optional, string)

  When building a Dockerfile with multiple build stages, target can be used to specify an intermediate build stage by name as the final stage for the resulting image. This corresponds to the Docker CLI `--target` parameter.

## License

MIT (see [LICENSE](LICENSE))
