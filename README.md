# Docker Image Builder for GitHub Actions

This repository contains a GitHub Action to build, tag and push a Docker image based on a `Dockerfile`.

## Configuration

The inputs to the action are documented below. At least one of `tag-latest`, `tag-snapshot` or `additional-tags` must be
specified, or the action will fail. Generally, pull requests and pushes to `master` should be built with `tag-snapshot`
enabled, and release tags should be built with `tag-latest` enabled.

```yaml
- uses: pulumi/action-docker-build@v1
  with:
      # Required. The name of the Docker repository to push to.
      repository: pulumi/test-action-docker-build
      # Required. The username with which to authenticate with the
      # specified Docker registry.
      username: pulumibot
      # Required. The password with which to authenticate with the
      # specified Docker registry. It is strongly recommended that
      # this value be considered a secret.
      password: ${{ secrets.DOCKER_HUB_PASSWORD }}
      # Optional. The path to the Dockerfile relative to the root
      # of the repository. Defaults to "Dockerfile".
      dockerfile: Dockerfile
      # Optional. The URL of the Docker registry to which to push.
      # Uses the Docker CLI default if unset.
      registry: https://customregistry.example.com/v2
      # Optional. Whether or not to apply the `latest` tag to the
      # built image, and push it to the registry. Defaults to false.
      tag-latest: true
      # Optional. Whether or not to apply a `snapshot` tag to the
      # built image, and push it to the registry. Snapshot tags consist
      # of the build date (in `YYYYMMDD-HHMMSS` format) and the short
      # commit hash of the source material. Defaults to false.
      tag-snapshot: true
      # Optional. A list of additional tags to apply to the image, and
      # push to the registry. Tags must be listed in a comma-separated
      # fashion.
      additional-tags: firstadditionaltag, secondadditionaltag
      # Optional. The platform architecture for which to build the image.  If
      # not supplied, the image will build on whatever the platform is of the
      # GH Action container as indicated by the `use` directive.
      platform: linux/amd64
```

## Notes on Docker Credentials

The action uses `docker login` in order to authenticate with a registry. Sadly none of Docker's credential helpers seem
to work with GitHub actions (citing a lack of X11 support for `secretservice`, and "inappropriate ioctl for device" when
initializing a GPG key for use with `pass`). Consequently, the login falls back to storing the credentials as base-64
encoded strings in a configuration directory with a warning emitted showing the path.

Pull requests to use a credentials helper are welcomed!

## Testing notes

The tests are themeselves a run in a GitHub Action workflow - see `.github/workflows/test.yml`.  The tests are hard-coded to use Pulumi's Docker Hub credentials and thus will not work on forks of this PR.  In addition, the repos to which we push test images (`pulumi/test-action-docker-build*`) are intentionally marked private on Docker Hub to avoid any confusion from Pulumi users downloading these test images because the test images themselves do not do anything useful.

## Development Notes

In order to correctly package the release for consumption by external repositories, we must do both of the following:

1. Run `npm install` to ensure that `node_modules/` is included when the Action is run by another repo.  (The Action will fail if these are not included.)
2. Run `npx tsc` to transpile the TypeScript into JS.  (The action will not include the latest changes unless this command is run.)
