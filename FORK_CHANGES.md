# Changes this fork carries

This branch is Apple's Unity plug-ins (`https://github.com/apple/unityplugins`, called Apple here) merged up to
Apple's `main` at `2c7d520b`, which Ateo builds and publishes as the `com.apple.unityplugin` packages on its
registry `https://upm.ateo.dev`. Every change the branch carries against that commit is listed below, one entry
each: what it does and why, where it lives and its commit, its upstream issue or pull request, and when it is
dropped. A change not listed here is not carried, apart from this file itself. The packages keep Apple's own
versions, so no version change is carried.

Every merge of a newer Apple `main` walks this list. An entry is kept when the merged code still needs it, and
dropped when Apple fixed the same thing in its own way and that fix is good. The list is then brought in line
with the merged branch, its base and every entry. How an update is taken, and what a further change means for
the packages' versions, are in the runbook `docs/runbooks/apple-plugins-fork.md` of the workshelf that keeps
`ch.ateo.mobile`.

Paths are relative to the repository root. A commit is named by its subject, with the hash it has on `main`,
which is never rewritten.

## The changes

1. **The editor build steps run in every editor, not only on a Mac.** Apple guards each editor build step, and
   the build settings and build profile code beside them, with
   `UNITY_EDITOR_OSX && (UNITY_IOS || UNITY_TVOS || UNITY_STANDALONE_OSX || UNITY_VISIONOS)`.
   `UNITY_EDITOR_OSX` names the host the editor runs on, not the build target, so an iOS build made in a
   Windows or Linux editor got an Xcode project without Apple's processing: no frameworks, capabilities,
   entitlements or Info.plist entries from the plug-ins. Each guard is widened to `UNITY_EDITOR`, and the
   target clause is left as Apple wrote it. This is the reason the fork exists. It lives in these files:
   - `plug-ins/Apple.Accessibility/Apple.Accessibility_Unity/Assets/Apple.Accessibility/Editor/AppleAccessibilityBuildStep.cs`
   - in `plug-ins/Apple.Core/Apple.Core_Unity/Assets/Apple.Core/Editor/`: `AppleBuild.cs`,
     `AppleBuildProfile.cs`, `AppleBuildProfileEditor.cs`, `AppleBuildSettingsProvider.cs`,
     `AppleBuildStep.cs`, `AppleCoreBuildStep.cs`, `AppleNativeLibrary.cs`, `ApplePlugInEnvironment.cs`,
     `AppleSecurityBuildStep.cs`, `AppleUnityPackage.cs` and `AppleUserManagementBuildStep.cs`
   - `plug-ins/Apple.CoreHaptics/Apple.CoreHaptics_Unity/Assets/Apple.CoreHaptics/Editor/CHBuildStep.cs`
   - `plug-ins/Apple.GameController/Apple.GameController_Unity/Assets/Apple.GameController/Editor/AppleGameControllerBuildStep.cs`
   - `plug-ins/Apple.GameKit/Apple.GameKit_Unity/Assets/Apple.GameKit/Editor/AppleGameKitBuildStep.cs`
   - `plug-ins/Apple.PHASE/Apple.PHASE_Unity/Assets/Editor/PHASEBuildStep.cs`
   - `plug-ins/Apple.SpatialController/Apple.SpatialController_Unity/Assets/Apple.SpatialController/Apple/Editor/AppleSpatialControllerBuildStep.cs`

   The StoreKit build step keeps Apple's guard, since Ateo builds no StoreKit package. The commit is "enable
   building outside of OSX", `07903a4c`. The merge "Merge Apple's main at 2c7d520 into the fork, keeping the
   editor build steps open to every editor", `52c75edf`, kept it and took Apple's deletion of seven command
   files in Apple.Core's `Editor` folder whose only change from Apple's was this widening. Upstream,
   [issue 125](https://github.com/apple/unityplugins/issues/125) asks why every editor build step is guarded by
   `UNITY_EDITOR_OSX`, and [issue 75](https://github.com/apple/unityplugins/issues/75) asks for the Apple build
   settings in a Windows editor. No open pull request makes the change. It is dropped when Apple guards these
   files by the build target alone, so that an editor on any host runs them.
2. **Play mode in a Windows or Linux editor starts without four native library errors.** Four calls into
   Apple's native libraries run at every play mode start: `Availability.OnApplicationStart` in Apple.Core reads
   the runtime environment through `AppleCoreNativeMac`, the two `Init` methods in Apple.GameKit's
   `DefaultHandlers.cs` register the NSException and NSError handlers with `GameKitWrapper`, and the static
   constructor of `GKGameActivity` checks its availability through `AppleCoreNativeMac` before it registers its
   callback. Those libraries ship for macOS and Apple devices only, so on any other host each call logs a
   `DllNotFoundException`, `GKGameActivity` is left unusable, and an editor with Error Pause on stops play at
   its first frame. Each call is skipped where `UNITY_EDITOR` is defined and `UNITY_EDITOR_OSX` is not, so a Mac
   editor and every player compile Apple's lines unchanged. Apple.Core's start line then reads "[Apple.Core
   Plug-In Runtime] Availability Runtime Environment is not read in this editor, since the plug-in's native
   library exists only for macOS and Apple devices". It lives in
   `plug-ins/Apple.Core/Apple.Core_Unity/Assets/Apple.Core/Runtime/Availability.cs`,
   `plug-ins/Apple.GameKit/Apple.GameKit_Unity/Assets/Apple.GameKit/Source/DefaultHandlers.cs` and
   `plug-ins/Apple.GameKit/Apple.GameKit_Unity/Assets/Apple.GameKit/Source/GKGameActivity.cs`, each of the four
   spots marked by a comment starting `ATEO:`. The commit is "A Windows or Linux editor starts play mode without
   the plug-ins' native library errors", `f7b00ddc`. Upstream,
   [pull request 95](https://github.com/apple/unityplugins/pull/95) guards the same three files, and
   [pull request 42](https://github.com/apple/unityplugins/pull/42) guards `Availability.cs` alone; both are
   open. A spot is dropped when Apple's own code no longer calls a native library there in an editor on a host
   the library does not ship for, and the change is gone when play mode in a Windows editor logs none of the
   four errors without it.
