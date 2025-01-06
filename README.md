


Setups the macos environment to be able to build a flutter app targetting ios. 

This requires some steps in order for this action to be successful:

# See also

  - [flutter build android](https://github.com/cedvdb/action-flutter-build-android)
  - [flutter build web](https://github.com/cedvdb/action-flutter-build-web)





## Steps to Enable Automatic Signing for App Store in GitHub Actions

This action now does automatic signing since version 2. If you do not want automatic signing use version 1.


### 1. Configure Your Project in Xcode
1. Open the project in Xcode.
2. Navigate to the **Signing & Capabilities** tab.
3. Enable **"Automatically manage signing."**
4. Ensure the **Team** is selected.

---

### 2. Authenticate with Apple Developer Account
- Use an **app-specific password** for your Apple ID.
- Store the following credentials in GitHub Secrets:
  - `APPLE_ID`: Your Apple ID (email).
  - `APPLE_ID_PASSWORD`: The app-specific password.

---

# 3. Build locally

Follow this step closely:

1. In Xcode, automatic signing should be checked for a new flutter project. Build your project once with this enabled.
2. Once your app has been built locally, you should have a `ExportOptions.plist` file created in the build directory `build/ios/ipa/ExportOptions.plist`. 
3. Copy the content of `ExportOptions.plist` and add it to a new `GithubActionsExportOptions.plist` file in the ios directory. This will be used by the action to sign the app.
4. You should now be able to build the project locally the same way it will be built in the action, verify that it is the case:

```
flutter build ipa --release --export-options-plist ios/GithubActionsExportOptions.plist
```

# 4. Usage


```yaml
name: Build and distribute

on:
  push:
    branches:
      - main

jobs:
  build:
    name: build
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v2

      - uses: cedvdb/action-flutter-build-ios@v1
        with:
          # always use --export-options-plist=ios/GithubActionsExportOptions.plist
          # this is an example command with flavor, use the one that suits your need.
          build-cmd: flutter build ipa --release --flavor prod --export-options-plist=ios/GithubActionsExportOptions.plist
          apple-id: ${{ secrets.APPLE_ID }}
          apple-id-password: ${{ secrets.APPLE_ID_PASSWORD }}

      - name: Archive IPA
        uses: actions/upload-artifact@v2
        with:
          name: release-ipa
          # Try running the build locally with the build command to be sure of this path
          path: build/ios/ipa/App-dev.ipa
```

Note: when using the build-cmd, use the `--export-options-plist=ios/GithubActionsExportOptions.plist` argument, so it uses the export option created in step #3.
