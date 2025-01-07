


Setups the macos environment to be able to build a flutter app targetting ios. 

This requires some steps in order for this action to be successful:

## See also

  - [flutter build android](https://github.com/cedvdb/action-flutter-build-android)
  - [flutter build web](https://github.com/cedvdb/action-flutter-build-web)



### Steps to Enable Automatic Signing for App Store in GitHub Actions

This action now does automatic signing since version 2. If you do not want automatic signing use version 1.


#### 1. Configure Your Project in Xcode
1. Open the project in Xcode.
2. Navigate to the **Signing & Capabilities** tab.
3. Enable **"Automatically manage signing."**
4. Ensure the **Team** is selected.

---

#### 2. Authenticate with Apple Developer Account 

There is a section at the end of this readme explaining how to get the `APPLE_ID_PASSWORD` and how to store it in github secrets

- Store the following credentials in GitHub Secrets:
  - `APPLE_ID`: Your Apple ID (email).
  - `APPLE_ID_PASSWORD`: The app-specific password 


---

#### 3. Build locally

Follow this step closely:

1. In Xcode, automatic signing should be checked for a new flutter project. Build your project once with this enabled.
2. Once your app has been built locally, you should have a `ExportOptions.plist` file created in the build directory `build/ios/ipa/ExportOptions.plist`. 
3. Copy the content of `ExportOptions.plist` and add it to a new `GithubActionsExportOptions.plist` file in the ios directory. This will be used by the action to sign the app.
4. Replace the `method` with the desired one in `GithubActionsExportOptions.plist` (eg: ad-hoc)

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>destination</key>
	<string>export</string>
	<key>generateAppStoreInformation</key>
	<false/>
	<key>manageAppVersionAndBuildNumber</key>
	<true/>
	<key>method</key>
	<string>app-store-connect</string>
	<key>signingStyle</key>
	<string>automatic</string>
	<key>stripSwiftSymbols</key>
	<true/>
	<key>teamID</key>
	<string>XXX</string>
	<key>testFlightInternalTestingOnly</key>
	<false/>
	<key>uploadSymbols</key>
	<true/>
</dict>
</plist>

```
5. You should now be able to build the project locally the same way it will be built in the action, verify that it is the case:

```
flutter build ipa --release --export-options-plist ios/GithubActionsExportOptions.plist
```

---

### 4. Usage


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

---

## How to Generate `APPLE_ID_PASSWORD` for GitHub Actions

The `APPLE_ID_PASSWORD` is an **App-Specific Password** generated from your Apple ID account. This guide explains how to create it and use it securely in GitHub Actions.

---

### Steps to Generate `APPLE_ID_PASSWORD`

#### 1. Sign In to Your Apple ID Account
- Go to the [Apple ID Account Management page](https://appleid.apple.com/).
- Log in with your Apple ID and password.

#### 2. Navigate to Security Settings
- Scroll down to the **Security** section.
- Ensure **Two-Factor Authentication** is enabled (this is required to generate app-specific passwords).

#### 3. Generate an App-Specific Password
1. In the **Security** section, find the **App-Specific Passwords** option.
2. Click **Generate an App-Specific Password**.
3. Enter a label for the password (e.g., "GitHub Actions").
4. Click **Create**.

#### 4. Copy the Password
- A password in the format `abcd-efgh-ijkl-mnop` will be displayed.
- Copy the password and store it securely.

---

### Adding the `APPLE_ID_PASSWORD` to GitHub Actions

#### 1. Open Your GitHub Repository
- Go to the repository on GitHub.

#### 2. Navigate to Secrets and Variables
- Go to **Settings > Secrets and variables > Actions**.

#### 3. Add a New Secret
- Click **New repository secret**.
- Fill in the following details:
  - **Name:** `APPLE_ID_PASSWORD`
  - **Value:** Paste the app-specific password you copied.

---

### Important Notes

- **What is `APPLE_ID_PASSWORD`?**
  - It's a password tied to your Apple ID, specifically for automating workflows like App Store submissions.

- **Revoking and Regenerating:**
  - If you revoke the app-specific password or regenerate it, update it in your GitHub repository secrets.

- **Security Best Practices:**
  - Never hard-code the `APPLE_ID_PASSWORD` in your workflow files.
  - Keep the password secure and do not share it publicly.


