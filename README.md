# Steam deploy
Github Action to deploy a game to Steam

## Setup

#### Prerequisites

This action assumes you are registered as a [partner](https://partner.steamgames.com/) with Steam.

#### 1. Create a Steam Build Account

Create a specialised builder account that only has access to `Edit App Metadata` and `Publish App Changes To Steam`.

https://partner.steamgames.com/doc/sdk/uploading#Build_Account

#### 2. Export your build

In order to upload a build, this action is assuming that you have created that build in a previous `step` or `job`.

For an example of how to do this in Unity, see [Unity Actions](https://github.com/game-ci/unity-actions).

The exported artifact will be used in the next step.

#### 3. Configure for deployment

In order to configure this action, configure a step that looks like the following:

_(The parameters are explained below)_

```yaml
jobs:
  deployment:
    name: Deployment to Steam ☁
    runs-on: ubuntu-latest
    steps:
      - uses: game-ci/steam-deploy@v1
        with:
          username: ${{ secrets.STEAM_USERNAME }}
          password: ${{ secrets.STEAM_PASSWORD }}
          mfaCode: ${{ secrets.STEAM_MFA_CODE }}
          personalAccessToken: ${{ secrets.PERSONAL_ACCESS_TOKEN }}
          configVdf: ${{ secrets.STEAM_CONFIG_VDF}}
          ssfnFileName: ${{ secrets.STEAM_SSFN_FILE_NAME }}
          ssfnFileContents: ${{ secrets.STEAM_SSFN_FILE_CONTENTS }}
          appId: 1234560
          buildDescription: v0.0.1
          rootPath: build
          depot1Path: StandaloneWindows64
          depot2Path: StandaloneLinux64
          releaseBranch: prerelease
```

## Configuration

#### username

The username of the Steam Builder Account that you created in setup step 1.

#### password

The password of the Steam Builder Account that you created in setup step 1.

#### mfaCode, personalAccessToken, configVdf, ssfnFileName, and ssfnFileContents

Deploying to Steam requires using Multi-Factor Authentication (MFA) through Steam Guard. 
This means that simply using username and password isn't enough to authenticate with Steam. 
Fortunately, GitHub runners share the same machine ID, so it is possible to go through the MFA process only once by using secrets for configVdf, ssfnFileName, and ssfnFileContents.

To go through the MFA process and get the values for configVdf, ssfnFileName, and ssfnFileContents, we recommend following these steps:
1. Copy [this request_steam_code.yml workflow](.github/workflows/request_steam_code.yml) to your repo's workflows.
1. Create [GitHub Secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets) for `STEAM_USERNAME` and `STEAM_PASSWORD`.
1. [Manually run](https://docs.github.com/en/actions/managing-workflow-runs/manually-running-a-workflow) the "Request Steam Code" workflow. 
Note that the workflow will fail at this time, but it will also cause an MFA email to be sent to your Steam Builder Account's email address.
1. Get the MFA code from the email, and create a Secret called `STEAM_MFA_CODE` using that value. 
1. Create a [Personal Access Token (PAT)](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token) with full `repo` access and save it as a Secret called `PERSONAL_ACCESS_TOKEN`. 
You could instead use an existing PAT if you already have one.
1. Run your deployment workflow, which should successfully run and create Secrets for `STEAM_CONFIG_VDF`, `STEAM_SSFN_FILE_NAME` and `STEAM_SSFN_FILE_CONTENTS`.
1. If `STEAM_CONFIG_VDF`, `STEAM_SSFN_FILE_NAME` and `STEAM_SSFN_FILE_CONTENTS` have been succesfully created, you should then DELETE the Secrets for both `STEAM_PASSWORD` and `STEAM_MFA_CODE`. 
Subsequent runs of your deployment workflow will fail if you re-use the same MFA code, and you need to remove the password to prevent requesting a new MFA code.

#### appId

The identifier of your app on steam. You can find it on your [dashboard](https://partner.steamgames.com/dashboard).

#### buildDescription

The identifier for this specific build, which helps you identify it in steam. 

It is recommended to use the semantic version of the build for this.

#### rootPath

The root path to your builds. This is the base of which depots will search your files.

#### depot[X]Path

Where X is any number between 1 and 9 (inclusive both).

The relative path following your root path for the files to be included in this depot.

If your appId is 125000 then the depots 125001 ... 125009 will be assumed.

_(feel free to contribute if you have a more complex use case!)_

#### releaseBranch

The branch within steam that this build will be automatically put live on.

It is recommended to **not use** branch `default` for this as it is potentially dangerous.
