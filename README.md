# github-action-release-apk

Composite action to publish signed android APK file and make github releases automatically

## ⌨️ Usage

Follow the code below to use this workflow

this workflow does the following.

+ Always debug build, upload artifact when push or pull-request on main branch.
+ Perform a release build, upload artifact and upload signed release apk on tags created in ``'v*.*.*'`` format
+ This workflow was used for projects
[compose-cocktail-recipes](https://github.com/gogoadl/compose-cocktail-recipes)


## 🎹 Inputs

``asset-name`` : asset name

``github-token`` : for using create github Releases and upload assets

``base64-keystore`` : to signing apk on github action, you should make your base64 encoded keystore file

``key-file`` : it is name that the keystore file

``keystore-password`` : keystore password

``keystore-alias`` : keystore alias

``key-password`` : key password

## 🎸 How to make encoded keyStore

first, 
generate keystore on your android app

``Android Studio > Build > Generate Signed Bundle or APK > APK > Create New or Choose Existing``

second, 
base64 make encoded text file that your keystore file

``openssl base64 -in [keystore file] -out [text file name]``

last,
make your encoded text to github secrets!

move to `` repository Settings > Security > Secrets and variables > Actions > New Repository Secrets ``

copy your encoded text and paste to secrets body!  

## 🎮 Example

```
name: Android CI

on:
  push:
    branches:
      - main
    tags:
      - 'v*.*.*'
  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: set up JDK 11
        uses: actions/setup-java@v3
        with:
          java-version: '11'
          distribution: 'temurin'
          cache: gradle

      - name: Grant execute permission for gradlew
        run: chmod +x gradlew

      - name: Set output
        id: vars
        run: echo "tag=${GITHUB_REF#refs/*/}" >> $GITHUB_OUTPUT
      - name: Check output
        env:
          RELEASE_VERSION: ${{ steps.vars.outputs.tag }}
        run: |
          echo $RELEASE_VERSION
          echo ${{ steps.vars.outputs.tag }}

      - name: Bump version
        uses: chkfung/android-version-actions@v1.2.1
        with:
          gradlePath: app/build.gradle # or app/build.gradle.kts
          versionCode: ${{github.run_number}}
          versionName: ${{env.RELEASE_VERSION}}

      - name: Build Debug APK
        run: ./gradlew assembleDebug

      - name: Upload artifact Debug APK
        uses: actions/upload-artifact@v3
        with:
          name: apk
          path: app/build/outputs/apk/debug/*.apk

  release:
    permissions:
      contents: write
    needs: build
    if: startsWith(github.ref, 'refs/tags/v')
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3 # Necessary to access local action
      - name: github-action-release-apk
        uses: gogoadl/github-action-release-apk@v1.0.0-alpha09
        with:
          asset-name: 'Cocktails.apk'
          github-token: ${{ secrets.GITHUB_TOKEN }}
          base64-keystore: ${{ secrets.APP_KEYSTORE_BASE64 }}
          key-file: ${{ secrets.APP_KEY_FILE }}
          keystore-password: ${{ secrets.APP_KEYSTORE_PASSWORD }}
          keystore-alias: ${{ secrets.APP_KEYSTORE_ALIAS }}
          key-password: ${{ secrets.APP_KEY_PASSWORD }}
```
