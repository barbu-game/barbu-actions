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
