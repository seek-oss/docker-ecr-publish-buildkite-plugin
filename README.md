# Docker ECR Publish

A [Buildkite plugin](https://buildkite.com/docs/agent/v3/plugins) to build, tag, cache and push entire docker images to ECR.

# Example

## Basic Usage

Dockerfile
```
FROM bash
RUN echo "my expensive build step"
```

```yml
steps:
  - command: 'echo anything'
    plugins:
      seek-oss/docker-ecr-publish#v1.0.2:
        dockerfile: Dockerfile
        ecr-name: insert-ecr-name
        additional-tags:
            - branch-1.1.0
```

# License

MIT (see [LICENSE](LICENSE))
