# iOS Foundation

Native iOS starter app (Swift + SwiftUI) maintained through GitHub only:
all building and testing runs on GitHub Actions macOS runners. No local
Mac is required.

## Current configuration

| Item | Value |
| --- | --- |
| Language / UI | Swift 5, SwiftUI |
| Xcode used in CI | Latest stable on the `macos-26` runner image |
| SDK built against | iOS 27 SDK (via the runner's Xcode) |
| Deployment target | iOS 17.0 |
| Bundle identifier | `com.wf.robot.muse.iosfoundation` |
| Signing | Automatic, no team set (CI builds unsigned) |
| App target | iPhone and iPad (`TARGETED_DEVICE_FAMILY = 1,2`) |

## How CI works

`.github/workflows/ios.yml` runs on every push to `main`, on pull
requests, and on manual dispatch:

1. Selects the latest stable Xcode available on the runner.
2. Picks the first available iPhone simulator on the image (whatever the
   newest device the runner ships) and builds the app for it.
3. Uploads the simulator `.app` as a workflow artifact.
4. Archives the app for device (`generic/platform=iOS`) with signing
   disabled and uploads the `.xcarchive` as an artifact.

Public repositories get unlimited standard GitHub Actions minutes,
including macOS runners, so this pipeline costs nothing to run.

## What you can and cannot do yet

- The simulator `.app` artifact proves the code compiles and links; it
  runs only inside an iOS Simulator on a Mac.
- The `.xcarchive` is **unsigned**. It cannot be installed on an iPhone
  or uploaded to TestFlight/App Store Connect until signing is set up.

## Signing and TestFlight

Activates when Wade's Apple Developer Program membership ($99/year) is
in place. Wade handles the account, fee, and App Store submission.

1. Set `DEVELOPMENT_TEAM` in the Xcode project to Wade's team ID.
2. Create a private repository for `fastlane match` signing assets and
   store the certificate and provisioning profile there.
3. Add the GitHub repository secrets listed in `fastlane/Fastfile`
   (`MATCH_PASSWORD`, App Store Connect API key material).
4. Implement the `beta` lane in `fastlane/Fastfile` (steps are written
   out in comments) and add a workflow step or a second workflow that
   runs it on version tags.
5. Signed builds can then be uploaded to TestFlight and installed on
   Wade's iPhone for device testing.

Note: App Store Connect currently requires uploads built with Xcode 26
or later and the iOS 26 SDK or later. From April 2027 the floor rises to
the iOS 27 SDK and a minimum target of iOS 15. This foundation already
satisfies both.

## Upgrading when Apple releases new versions

- **New Xcode / iOS SDK:** nothing to change for routine releases — CI
  selects `latest-stable` Xcode on the runner image. If a build starts
  failing after a runner image update, check the workflow run's
  "Show toolchain versions" step first.
- **New iPhone models:** the workflow auto-selects whatever iPhone
  simulator the runner image provides, so new hardware support arrives
  with runner image updates. No device names are hard-coded.
- **Raising the deployment target:** change `IPHONEOS_DEPLOYMENT_TARGET`
  in `iOSFoundation.xcodeproj/project.pbxproj` (project and target
  configurations) when older iOS support is dropped.
- **April 2027 requirement change:** no action needed; see above.

## Repository visibility

This repository is public by design: it contains no secrets, signing
material lives in a separate private `match` repository and in GitHub
secrets, and public visibility is what makes the Actions minutes free
and unlimited.
