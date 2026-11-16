# ghr

Throwaway repo used to test release-please with immutable releases.

## Question

Does release-please work on a repo with immutable releases enabled when it uses `draft: true` together with `force-tag-creation: true`? Specifically, can a draft release created from a pre-existing tag have an asset attached and then be published?

## Setup

- Immutable releases enabled on the repo.
- Actions default workflow permissions set to `write`, with Actions allowed to create pull requests.
- No rulesets on the repo.
- `release-please-config.json`:
  ```json
  {
    "packages": {
      ".": {
        "release-type": "simple",
        "draft": true,
        "force-tag-creation": true
      }
    }
  }
  ```
- `.github/workflows/release.yml` runs on push to `main` and, in one job:
  1. runs `googleapis/release-please-action@v4`
  2. uploads an asset to the draft release (`gh release upload`)
  3. publishes it (`gh release edit <tag> --draft=false`)
  4. prints the tag's commit and the release state

## Result

Passed. See the [release run](https://github.com/kalosisz/ghr/actions/runs/43886325838).

| Check | Result |
|---|---|
| Release-please outputs | `release_created=true`, `tag_name=v1.0.0`, `sha=3bb138eef382048b8dc3ea6a8ec5ffbaca7be701` |
| Asset upload to the draft | Succeeded (`asset.txt`) |
| `gh release edit v1.0.0 --draft=false` | Succeeded, no 422 or 403 |
| Tag `v1.0.0` points at | `3bb138eef382048b8dc3ea6a8ec5ffbaca7be701` |
| Release-PR merge commit and tip of `main` | `3bb138eef382048b8dc3ea6a8ec5ffbaca7be701` (same commit) |
| Release state after publish | `draft: false`, `immutable: true`, asset attached |

## Not covered

- A ruleset restricting tag creation (`GH013` for `GITHUB_TOKEN`).
- `main` moving between the release-PR merge and the publish step.
- Trying to move or delete the tag after publish to confirm the lock. The `immutable: true` flag on the release is the evidence that the lock applies.
- `gh release create --target` with a commit that is not the tip of the branch.

## Notes

- After publish, the tag name `v1.0.0` is reserved permanently on this repo, even if the release is deleted.
