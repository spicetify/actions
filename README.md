# spicetify/actions

Reusable GitHub Actions for the Spicetify ecosystem. Each action lives in its
own directory and is referenced by path.

## `publish`

Packs a built module, checksums it, and opens the pull request that submits it
to the [Spicetify module store](https://github.com/spicetify/modules). Call it
from your own release workflow.

```yaml
- uses: spicetify/actions/publish@v1
  with:
    dist: dist/my-module@1.0.0
    release-tag: ${{ github.ref_name }}
    token: ${{ secrets.SPICETIFY_SUBMIT_TOKEN }}
```

### Inputs

| Input         | Required | Description                                                                                                                 |
| ------------- | -------- | --------------------------------------------------------------------------------------------------------------------------- |
| `dist`        | no       | The built module directory, containing `metadata.json`. Required unless `zip` is given.                                     |
| `zip`         | no       | An already packed module, such as the attested zip from [`build-module`](#build-module). Submitted byte for byte.           |
| `artifact`    | no       | Public https URL the packed zip is served from. Upload it yourself, or pass `release-tag` to have the action upload it.     |
| `release-tag` | no       | Upload the packed zip to this release in the calling repository and derive the artifact URL from it.                        |
| `token`       | no       | A token with `public_repo` scope on your account, used to open the pull request from your fork. Omit it for print-only mode. |
| `registry`    | no       | The registry repository to submit to. Defaults to `spicetify/modules`.                                                      |

### Outputs

| Output     | Description                        |
| ---------- | ---------------------------------- |
| `entry`    | The vault entry this submission adds. |
| `checksum` | The `sha256` of the packed artifact.  |

### What it does, and does not, guarantee

The action is convenience, not authority. Everything it computes is recomputed
by the registry's validator from the artifact itself, so a submission cannot
talk its way past the checks by lying in the pull request. Running without a
`token` skips the pull request entirely and prints the exact vault entry to
submit by hand, so no one has to hold a credential to publish.

See [publishing a module](https://spicetify.app/docs/development/publishing)
for the full flow.

## `build-module`

A reusable workflow that builds, packs and attests a module from the commit
that called it. The attestation names this workflow as its signer, so the
registry can verify that your zip came out of these steps, run on a
GitHub-hosted runner against one commit of your repository, and a reviewer can
read the source at exactly that commit instead of the bundled output.

```yaml
on:
  release:
    types: [published]

jobs:
  build:
    uses: spicetify/actions/.github/workflows/build-module.yml@v1
    permissions:
      contents: write # upload the zip to your release
      id-token: write # sign the attestation
      attestations: write # store it
    with:
      release-tag: ${{ github.event.release.tag_name }}

  submit:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - uses: actions/download-artifact@v8
        with:
          name: ${{ needs.build.outputs.zip }}
      - uses: spicetify/actions/publish@v1
        with:
          zip: ${{ needs.build.outputs.zip }}
          artifact: ${{ needs.build.outputs.artifact }}
          token: ${{ secrets.SPICETIFY_SUBMIT_TOKEN }}
```

The build runs nothing from your repository except the module source that
the kit bundles:

- Dependencies install from your committed lockfile (npm, pnpm, or bun) with
  their install scripts and pnpm's `.pnpmfile.cjs` hooks skipped.
- Your package scripts never run. The build calls `spicetify-kit build` from
  the npm registry, at the `@spicetify/kit` version your lockfile pins, outside
  your project directory, so it uses the kit's bundled classmap rather than a
  `stitch.config.json` of yours.
- Signing happens in a separate job that never checks out your repository, so
  nothing in the build can obtain a token to sign as this workflow.
- The zip is uploaded to the release without overwriting an existing asset,
  because a published zip is what its vault checksum describes.

### Inputs

Both inputs are optional.

| Input         | Default | Description                                                                         |
| ------------- | ------- | ----------------------------------------------------------------------------------- |
| `module`      | `.`     | Project directory holding `package.json`, its lockfile and `metadata.json`.         |
| `release-tag` |         | Upload the packed zip to this existing release of the calling repository.           |

### Outputs

The `submit` job in the example above reads these to submit the attested zip.

| Output     | Description                                                                |
| ---------- | -------------------------------------------------------------------------- |
| `zip`      | File name of the packed module, also the name of the run artifact holding it. |
| `checksum` | The `sha256` of the packed module.                                         |
| `artifact` | The release download URL, when `release-tag` was given.                    |

To check a build yourself:

```shell
gh attestation verify my-module@1.0.0.zip --repo you/my-module \
  --signer-workflow spicetify/actions/.github/workflows/build-module.yml \
  --deny-self-hosted-runners
```

## Versioning

`v1` is a floating tag that moves to each `v1.x.y` release; pin it for the
latest compatible version, or pin an immutable `v1.2.3` tag for an exact one.
Tags apply to the whole repository, so every action here shares one version
line.

## License

GPLv3. See [LICENSE](LICENSE).
