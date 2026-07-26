# macOS native build experiment handoff

## Scope

This branch is an **experimental handoff**, not a completed macOS implementation.

- Repository: `https://github.com/orca-svg/app-ios-buddy`
- Branch: `experiment/macos-channel-talk-isolation`
- Base commit: `a6b305f fix widget presenting negative D-Day (#139)`
- Goal: isolate ChannelTalk from the first Mac Destination build experiment.

The Mac Destination was selected locally in Xcode. This branch does not claim that a new native macOS target has already been created.

## Changes in this branch

### `soap/soapApp.swift`

- `ChannelIOFront` import is guarded by `#if os(iOS)`.
- ChannelTalk initialization and boot code run only on iOS.

### `BuddyFeature/Sources/BuddyFeatureSettings/Settings/SettingsView.swift`

- `ChannelIOFront` import is guarded by `#if os(iOS)`.
- The `Chat with Us` / `ChannelIO.showMessenger()` section is shown only on iOS.

### `BuddyFeature/Package.swift`

- The `ChannelIOSDK` target dependency is marked with:

```swift
condition: .when(platforms: [.iOS])
```

## Important known limitation

`soap.xcodeproj/project.pbxproj` still contains a direct `ChannelIOSDK` product/framework reference. Therefore this branch has **not** proven that ChannelTalk is fully excluded from the Mac build.

The next Mac/Xcode experiment must verify whether the direct Xcode target link still causes a build error. Do not report this branch as a successful Mac build.

## Next steps on macOS/Xcode

1. Clone or fetch this branch:

```bash
git fetch origin
git switch experiment/macos-channel-talk-isolation
git pull --ff-only origin experiment/macos-channel-talk-isolation
```

2. Open `soap.xcodeproj` in Xcode.
3. Select the same `soap` scheme and Mac Destination used in the original experiment.
4. Build without further changes and save the first actionable error.
5. Check whether the error is caused by the direct `ChannelIOSDK` reference in the Xcode target.
6. If needed, inspect the `soap` target's Frameworks/Package Products phase and determine the smallest platform-specific way to exclude ChannelIOSDK for Mac while preserving iOS.
7. Build again and record the result.
8. Only after ChannelTalk is no longer the first blocker, investigate:
   - `UIApplicationDelegate` / `NSApplicationDelegate`
   - `UIApplication.shared`
   - APNs/FCM and UserNotifications
   - Firebase iOS-only services
   - UIKit-backed TaxiChat (`UIViewRepresentable` + `UICollectionView`)
9. Restore or test one boundary at a time.
10. Capture login and TaxiChat UI behavior only after the app can launch on Mac.

## What to record

- macOS version
- Xcode version
- branch and commit
- scheme and destination
- first error message and file/module
- whether removing/isolation of `ChannelIOSDK` changes the error
- build result after each change
- login screen result
- TaxiChat result
- screenshots or screen recording for UI problems

## PR rule

Do not open a feature PR from this branch until a real Mac/Xcode build has been run. If the experiment identifies a safe, minimal fix, update this branch with that fix and include the build evidence in the PR description.

Possible later PRs:

1. ChannelTalk/Xcode target platform isolation, if confirmed necessary.
2. App lifecycle platform separation, if confirmed necessary.
3. Login UI correction, only if reproduced.
4. TaxiChat Mac UI adaptation, only if reproduced and scoped.

## Verification performed before handoff

On the current Linux environment:

- Source-level conditional guards were checked.
- Package manifest conditional dependency was checked.
- `git diff --check` passed.
- Xcode/macOS build was **not** run because `xcodebuild` and the Swift compiler are unavailable here.
