# Android: Deploy build

Deploys an android build to a track on the Google Play store

Part of a set, see also [Android: Promote build](https://github.com/guardian/actions-android-promote-build)

## Usage

`@guardian/actions-android-deploy-build` is an opiniated but flexible building block on which to build android deployment automation. It takes a **target build** identified by a git reference and an artifact name, and deploys it to a **target track** identified by package name, track name and rollout %.

It can be used to implement continuous deployment workflows, or on-demand workflows with an interface such as

<img width="312" alt="Screenshot 2024-10-25 at 17 06 31" src="https://github.com/user-attachments/assets/913f6366-9d43-47a6-8c60-e874a65168ca">


### Requirements

To integrate with your github repository, this action expects your target build to
- Be built by a github workflow named `build.yml`
- Be named according to the format `v${unique_build_identifier}-*.aab`, e.g. `v2.1.0.7239-myapp-release.aab`, where `unique_build_identifier` does not contain a `-` character and typically contains both the version name and version code
- Be stored in a uniquely identifiable github artifact. Storing multiple `.aab` files per artifact will result in undetermined behaviour.

To be successfully used in a workflow, this actions requires the client to
1. Check out the repository beforehand (this is required for the action to process the git reference and to tag it when a deployment is successful)
1. Provide the following permissions:
   1. `actions: read` enables this action to download the target build `.aab`
   2. `contents: write` enables this action tagging the git reference with the version of the .aab just deployed

### Inputs

| Name | Description | Example |
| - | - | - |
| `dry-run` | If enabled, read operations will be performed but any write operations such as deploying will be printed to stdout instead of executed. | `boolean`, **optional**, default: **false** |
| `package-name` | The package name that identifies the android application in the google play console | `string`, **required** |
| `track` | The track in the play console to deploy to. Default track names are `production`, `beta` (open testing), `alpha` (default closed testing track) or `internal`. Custom closed testing tracks can also be identified by their name | `string`, **required** |
| `rollout` | The percentange rollout for the deployment, expressed as a string between 0.00 and 1.00. For example, 50% rollout is "0.50" | `string`, **optional**, default: **1.00** |
| `artifact-name` | The artifact name containing the target build. This name is given during the `upload-artifact` step in `build.yml` | `string`, **required** |
| `gitref` | A git branch or tag (e.g. `main`) identifying your target build. This input is optional, but one of either `gitref` or `sha` must be passed to this action.  | `string`, **optional** |
| `sha` | A git SHA (e.g. `main`) identifying your target build. This input is optional, but one of either `gitref` or `sha` must be passed to this action. If both are given, then `sha` takes precedence over `gitref` | `string`, **optional** |
| `github-token` | Token used to authenticate git operations, such as `fetch` and `tag`. Usually passsing `${{ secrets.GITHUB_TOKEN }}` is the desired setup | `string`, **required** |
| `console-credentials` | The JSON credentials for the service account to use in deployment | `string`, **required** |

### Outputs

| Name | Description | Example |
| - | - | - |
| `aab` | Filename of the aab just deployed | `v2.1.0.7239-myapp-release.aab` |
| `sha` | Commit hash of the target build  | `496fc2b4f395ed4f097ed959cc6493ccf9704519` |

## Examples

Ad-hoc deployment
```yml
jobs:
  deploy:
    name: Deploy to Google Play
    runs-on: ubuntu-latest
    permissions:
      contents: write
      actions: read
    outputs:
      aab: ${{ steps.deploy.outputs.aab  }}
      sha: ${{ steps.deploy.outputs.sha }}
    steps:
      - uses: actions/checkout@v4

      - name: Release ${{ inputs.type }} build to ${{ inputs.rollout }} of ${{ inputs.type  }} track from ${{ inputs.GITREF }}@${{ inputs.SHA }}
        uses: guardian/actions-android-deploy-build@496fc2b4f395ed4f097ed959cc6493ccf9704519 #v1
        id: deploy
        with:
          dry-run: false
          package-name: com.guardian
          track: ${{ inputs.type }}
          rollout: ${{ inputs.rollout }}
          artifact-name: ${{ inputs.type }}-apks
          gitref: ${{ inputs.gitref }}
          github-token: ${{ secrets.GITHUB_TOKEN }}
          console-credentials: ${{ secrets.CONSOLE_CREDENTIALS }}

```
