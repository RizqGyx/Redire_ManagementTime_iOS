<div align="center">

  <img src="C4-MIRACLE/ICONREAL.icon/Assets/Graphic.png" alt="Redire app icon" width="108" height="108" />

  <h1>Redire</h1>

  <p>
    <strong>An intentional-break companion that helps people return to focus.</strong><br />
    Screen Time intervention · purpose-led breaks · Dynamic Island accountability
  </p>

  <p>
    <img src="https://img.shields.io/badge/Platform-iOS%2026.5%2B-blue?logo=apple" alt="iOS 26.5+" />
    <img src="https://img.shields.io/badge/Language-Swift-orange?logo=swift" alt="Swift" />
    <img src="https://img.shields.io/badge/UI-SwiftUI-0A84FF" alt="SwiftUI" />
    <img src="https://img.shields.io/badge/Screen%20Time-FamilyControls-5E5CE6" alt="FamilyControls" />
    <img src="https://img.shields.io/badge/Live%20Activity-ActivityKit-FF375F" alt="ActivityKit" />
  </p>

</div>

---

## Overview

**Redire** is an iOS focus companion that reframes app blocking as a deliberate return-to-work ritual. When a user tries to open a selected distracting app during Work Mode, iOS presents a custom Screen Time shield. Choosing **Take a Break** opens Redire, where the user sets a break duration and names the task they will return to.

The break is shown as a Live Activity on the Lock Screen and Dynamic Island. At the end of the countdown, the user chooses whether to continue working or start another intentional break. A nautical fishing theme turns each successful return into a small, positive piece of accountability rather than a punitive interruption.

> **Project note:** the repository and Xcode target retain the internal name `C4-MIRACLE`; the product experience is branded as **Redire**.

## The Focus Loop

```text
Start Work Mode
      ↓
Selected distracting apps are shielded by iOS
      ↓
User opens a blocked app → taps “Take a Break”
      ↓
Redire collects break length + next task
      ↓
All selected apps unlock temporarily + Live Activity starts
      ↓
Countdown reaches zero
      ↓
Continue to Work (re-shield)  ←→  Take a New Break
```

## Key Features

### Screen Time–powered Work Mode

- Uses `FamilyControls` to request individual authorization and let users select distracting apps or categories.
- Uses `ManagedSettings` to apply an iOS-native shield during Work Mode, rather than relying on a foreground timer or a custom overlay.
- Customizes the system-owned shield with Redire’s visual language and a direct **Take a Break** path back into the app.
- Persists the selected apps and shared state so the main app and Screen Time extensions remain synchronized.

### Intentional Break Planning

- Opens a focused break-planning sheet from the shield action.
- Lets users set a flexible minute-and-second duration (up to 3 hours) and write their next task before resuming the distracting app.
- Temporarily unlocks the complete selected set of distracting apps, treating a break as a real break instead of permitting only one app.
- Reopens the requested app after the shield state has propagated, with a retry path for iOS timing delays.

### Dynamic Island & Lock Screen Accountability

- Starts a Live Activity with a countdown and the user’s intended next task.
- Surfaces a one-minute warning and an end-of-break state with clear actions: **Dismiss** or **Continue to work**.
- Supports app-deep-link routes and App Intents for starting Work Mode, ending it, and handling Live Activity actions.
- Uses `DeviceActivityMonitor` for wall-clock and aggregate app-usage break boundaries, because third-party apps cannot depend on a background countdown.

### Privacy-conscious, Resilient Architecture

- Uses an App Group-backed store to coordinate the app, Shield Action extension, Shield Configuration extension, Device Activity monitor, and Live Activity extension.
- Stores opaque Screen Time tokens—not browsing content—and resolves app identity through a token catalogue with fallbacks for iOS privacy restrictions.
- Reconciles break state when the app returns to the foreground, providing a fallback when Device Activity callbacks are delayed.
- Includes a diagnostics screen with a cross-process event timeline to make extension behavior observable during development.

### Gentle Motivation

- Gives positive fishing-themed feedback when a user returns to work and an encouraging consequence when they dismiss the loop.
- Shows a daily recap of committed returns and missed returns.
- Includes a visual achievement experience for the product’s gamification direction.

## Technical Highlights

| Challenge | Implementation |
|---|---|
| Block app access from an iOS app | `FamilyControls` authorization + `ManagedSettings` shields |
| Receive an action from Apple’s shield UI | `ShieldActionExtension` writes a `BreakRequest` and returns `openParentalControlsApp` |
| Coordinate five isolated processes | Shared `UserDefaults` suite in an App Group, Codable models, and foreground reconciliation |
| Keep a break visible outside the app | `ActivityKit` Live Activity with Dynamic Island and Lock Screen layouts |
| Handle break expiry without a normal background timer | `DeviceActivity` schedule plus aggregate usage threshold |
| Preserve the selected app’s identity | Opaque-token mapping, `FamilyActivityData` catalogue, and shield-observation fallback |
| Avoid re-opening before unshielding propagates | Two-phase save/launch flow with an asynchronous retry |

## Tech Stack

| Area | Technology |
|---|---|
| Language | Swift 5+ |
| UI | SwiftUI |
| State | Combine, `@StateObject`, `@EnvironmentObject`, `@Published` |
| App restrictions | FamilyControls, ManagedSettings, ManagedSettingsUI |
| Activity scheduling | DeviceActivity |
| Live surface | ActivityKit, WidgetKit, Dynamic Island |
| System actions | App Intents, App Shortcuts, custom URL scheme |
| Notifications | UserNotifications |
| Cross-target persistence | App Groups + `UserDefaults` + `Codable` |
| Design | Custom SwiftUI design system and nautical illustration assets |

## Architecture

```text
C4-MIRACLE/
├── App/
│   ├── AppState.swift                     # App lifecycle, routing, shared-state reconciliation
│   └── RootView.swift                     # Onboarding / home / break-sheet routing
├── Core/
│   ├── Services/
│   │   ├── ScreenTimeService.swift        # Authorization, shields, break grants, monitoring
│   │   └── AppLaunchService.swift         # Reopen requested app after a break is granted
│   ├── Components/                        # Reusable UI components
│   └── DesignSystem/                      # Brand colors and typography
├── Features/
│   ├── ScreenTime/                        # Onboarding, permissions, break planner, Work Mode
│   ├── Home/                              # Focus-session home and daily recap
│   ├── Achievement/                       # Gamification UI
│   └── Settings/                          # Configuration and diagnostics
├── Shared/
│   ├── Models/                            # Break request, grant, outcome, Live Activity data
│   ├── Storage/                           # App Group store and Screen Time token encoding
│   ├── Services/                          # Live Activity and local-notification helpers
│   └── Intents/                           # Live Activity actions
└── Targets/
    ├── ShieldExtension/                   # Styles the iOS Screen Time shield
    ├── ShieldActionExtension/             # Handles shield button taps
    ├── DeviceActivityMonitorExtension/    # Observes break time / usage boundaries
    └── LiveActivityExtension/             # Lock Screen and Dynamic Island UI
```

### Cross-process data flow

```text
Shield Action Extension
        │ writes BreakRequest
        ▼
App Group SharedStore ───────► Main app presents BreakConfigView
        ▲                                      │
        │                                      ▼
Device Activity Monitor ◄──── ScreenTimeService grants break
        │                                      │
        └──────────────► Live Activity Extension renders countdown
```

## Requirements

- Xcode with the iOS 26.5 SDK
- iOS 26.5+ on a physical iPhone or iPad
- An Apple Developer team provisioned for the **Family Controls** capability and an App Group
- Live Activities enabled on the test device

> The Simulator cannot exercise the complete Screen Time / shield / Device Activity flow. Test this project on real hardware with a development provisioning profile that includes the required entitlements.

## Getting Started

1. Clone the repository and open the project:

   ```sh
   git clone https://github.com/USERNAME/Redire_ManagementTime_iOS.git
   cd Redire_ManagementTime_iOS
   open C4-MIRACLE.xcodeproj
   ```

2. In Xcode, select your development team for the main app and all extension targets.

3. Ensure the bundle identifiers and App Group (`group.com.daffa.miracle`) are registered under that team. If you change them, update the corresponding entitlements and `Shared/Constants/AppGroup.swift` together.

4. Choose a physical device, build, and run with **Cmd + R**.

5. Complete onboarding:

   - Grant Screen Time access.
   - Choose the apps or categories to shield.
   - Enable notifications and Live Activities if prompted.
   - Start Work Mode, attempt to open a selected app, and take a planned break.
