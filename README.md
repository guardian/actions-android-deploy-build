# actions-android-deploy-build

Deploys an android build to a track on the google play store

## Requirements

1. Needs the client to checkout repository (because it uses it to determine the SHA for a gitref, and it needs to push a new tag)
2. Needs the filename of the aab to follow the following format: `v${version_name}-*.aab`
3. The clients needs to call the action with  `actions: read` and `contents: write` permissions. The first enables this action to download the aab, the other to push a tag to the client repo.
4. A workflow named `build.yml` needs to have ran against the commit being deployed and built the aab