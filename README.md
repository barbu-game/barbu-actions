# barbu-actions

Reusable CI workflows for the Barbu stack.

## `publish-api.yml`

Implements the publish half of the **getout** contract pipeline. `barbu-server`'s
release workflow generates its code-first OpenAPI spec, tags the release, and
uploads the spec as an artifact, then calls this workflow:

```yaml
publish-api:
  needs: release
  permissions:
    contents: read
    packages: write
  uses: barbu-game/barbu-actions/.github/workflows/publish-api.yml@main
  with:
    version: ${{ needs.release.outputs.version }}
  secrets: inherit
```

It runs Orval against the spec and publishes **`@barbu-game/barbu-api`** (typed
client + DTOs) to GitHub Packages under the version computed by the server's
semantic-version step. `barbu-web` pins that package, so any contract drift breaks
its typecheck.

The workflow is **self-contained**: it scaffolds the package (Orval config,
tsconfig, manifest) inline rather than checking out this repo, so it works even
when every repo in the org is private.

## `build-push-image.yml`

Builds a service image and pushes it to `ghcr.io/barbu-game`. Called by the
`barbu-server` / `barbu-web` release workflows after the version is computed.

| Input | Required | Description |
|---|---|---|
| `image` | yes | Image name under `ghcr.io/barbu-game` (e.g. `barbu-server`). |
| `version` | yes | Semver tag to publish (no leading `v`). |
| `context` | no | Build context path. |
| `dockerfile` | no | Path to the Dockerfile. |
| `build-args` | no | Newline-separated build args (`KEY=value`). |
| `npm-auth` | no | Pass `NPM_AUTH_TOKEN` as a build secret (for private `@barbu-game` packages). |

## `bump-deploy.yml`

Completes the zero-touch pipeline: after the image is published, it pins the new
image tag in `barbu-deploy` and pushes, so ArgoCD rolls it out with no manual
chart edit. Needs a push token to `barbu-deploy`, passed via `secrets: inherit`
(`DEPLOY_BUMP_TOKEN`).

| Input | Required | Description |
|---|---|---|
| `chart` | yes | Chart directory under `barbu-deploy` (e.g. `charts/barbu-web`). |
| `version` | yes | New image version to pin (no leading `v`). |
