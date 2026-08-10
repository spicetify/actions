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
| `dist`        | yes      | The built module directory, containing `metadata.json`.                                                                     |
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

## Versioning

`v1` is a floating tag that moves to each `v1.x.y` release; pin it for the
latest compatible version, or pin an immutable `v1.2.3` tag for an exact one.
Tags apply to the whole repository, so every action here shares one version
line.

## License

GPLv3. See [LICENSE](LICENSE).
