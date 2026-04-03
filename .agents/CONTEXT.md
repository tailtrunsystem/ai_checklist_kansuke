# Project Overview
The repository is split into three main folder structures, each serving a distinct role within the Kansuke product suite. 

- **`kansuke-app` (Main App)**: Serves as the primary application and the host repository. It manages top-level authentication, base shell interfaces, and importantly, houses the mono-repo style `packages` directory that provides the foundational building blocks for all modules.
- **`kansuke-photo` (Photo Module)**: A specialized application/module focused primarily on photography and media management. It is tailored to handle heavy image capture functionalities, image editing, EXIF reading (`native_exif`), EXIF data injection, ML-kit barcode scanning, and PDF combining.
- **`kansuke-kensa` (Inspection/Kensa Module)**: Designed specifically for building/property inspection workflows. It combines photography constraints tailored for inspections with additional communication capabilities like Firebase Cloud Messaging (FCM) and deep APNS integration for push notifications, alongside file archiving features (`archive`).

---

# Dependency Mapping
The three codebases are heavily interlinked. Instead of duplicating foundational code, they share generic implementations via Dart package mechanisms.

- **Local Path Dependencies (`kansuke-app`)**: The main app natively hosts the core logic within its `packages/` directory and references them locally (e.g., `path: packages/common`, `path: packages/kansuke_property`).
- **Git Path Dependencies (`kansuke-photo`, `kansuke-kensa`)**: The specialized modules reach out to the Main App's remote repository to pull the shared features. In their `pubspec.yaml`, they define remote packages using specific GitHub repository links, commit hashes targeting `0dab18d554e8efd3187082c3395afc0289354534`, and sub-directory paths (e.g., `git: url: https://github.com/KansukeAppRebuildTeam/kansuke-app.git, path: packages/kansuke_camera`). 
- **Additional Shared Repositories**: All modules also rely on external shared resources maintained by the team, notably `shared_data_plugin` (from `share_data_group`) and a customized fork of FFmpeg (`ffmpeg_kit_flutter`).

---

# Critical Shared Logic (High-Risk Zones)
Modifying the code inside the `kansuke-app/packages/` directory has a widespread impact because it propagates instantly to the main app, and eventually to Kensa and Photo when their Git hashes are updated. The high-risk architectural zones include:

1. **`packages/common`**: The backbone of the entire application suite. It contains core application states, `dio`/`retrofit` network error handling boundaries, generalized extension hooks, custom localization constraints, and fundamental UI widgets.
2. **`packages/kansuke_camera`**: Heavily critical interaction layer. Contains native platform integrations (Method Channels bridging via `KansukeCameraPlugin.swift` for iOS) and device orientation calculation (`device_orientation_service.dart`). A breakage here means no image capture for both Photo and Kensa apps.
3. **`packages/image_memory_manager`**: Memory allocations. Since the tools are photo-heavy, this package coordinates how high-resolution images are buffered and cached. Adjustments here present high risks for OOM (Out Of Memory) iOS crashes across the product line.
4. **`packages/kansuke_property`**: The shared logic coordinating property listings and real-estate data parsing.

---

# Key Configuration Files

### iOS Native Level
- **Method Channels & Plugins**: `kansuke-app/packages/kansuke_camera/ios/Classes/KansukeCameraPlugin.swift` handles the lowest-level camera instructions. Modifications require extensive iOS physical device testing across all 3 modules.
- **Runner Configuration (`AppDelegate.swift`)**: 
  - *Location*: e.g., `kansuke-kensa/ios/Runner/AppDelegate.swift`. 
  - *Importance*: Sets up internal Notification Centers and registers critical Flutter Method Channels (e.g., `"gmo.kansuke.inspection.dev/apns"`). It establishes the exact behavior on how the apps respond to background and foreground activities (routing APNS directly vs. FCM).

### Flutter Level
- **Dependency Definitions (`pubspec.yaml`)**: Controls environment overrides, `dependency_overrides` (like `intl` and `web`), and points modules strictly down to exact commit refs. Modifying these refs breaks or upgrades dependencies organization-wide.
- **Dependency Injection (`lib/di/injection.dart`)**: Utilizing `get_it` and `@InjectableInit`. Each distinct app orchestrates its services separately via its DI config but pulls classes generated from the common packages.
- **Application Navigation (`lib/routes/app_routes.dart` & `auto_route` configs)**: Each of the 3 modules has a uniquely mapped routing tree configuration. Changing base routes impacts deeplinking and push notification behaviors relying on Route propagation.

---

# Deep Logic & Implementation Patterns

### 1. BLoC & State Persistence (`KoteiBloc`)
- **Hierarchy Management**: `KoteiBloc` manages a tree-like structure of `FolderItem` and `KoteiPhotoRow`. It uses `BreadcrumbItem` to track the user's path.
- **State Restoration**: The app persists the last-visited breadcrumbs using `KoteiAppStore.setLastBreadcrumbs`. This allows the app to restore the user's exact folder position upon restart.
- **Add/Insert Logic**: Adding items (`_onAddButtonTapped`) vs. Inserting items between others (`_onAddItemTapped`). Insertion calculates a `sortNo` based on adjacent items.

### 2. BlackBoard (BB) & Metadata Logic
- **Data Structure**: BB data is stored as a `Map<String, dynamic>` (often called `dicBB` or `initialValues`). Keys follow the pattern `bb_val1`, `bb_val2`, etc., mapping to specific labels on the physical blackboard.
- **Offline-First Persistence**: Data is first saved to a local Drift (SQLite) database (`SaveBlackboardInputToDb`) and cached in `SharedPreferences` (`SaveLastUsedBlackboardInputsToPrefs`) before being synced to the server.
- **Shared Package Core**: Much of the BB rendering logic resides in `packages/kansuke_camera` (`ConstructionBoard`, `BlackboardService`), making it a high-impact area for UI changes.

### 3. Lifecycle & Focus Challenges
- **Navigation POP Lifecycle**: When returning from the Camera or BB Edit screen, the `KoteiBloc` must refresh the list. A common failure point is losing the "Focus" (selection) on the newly created or edited item, causing the list to jump to the top (Index 0).
- **Service/Repository Boundary**: Logic is split between `AppStoreService` (Global/Auth state) and `KoteiAppStore` (Module-specific state). Understanding which one holds the truth for a specific ID is critical for debugging session/account switching issues.

### 4. QR Code Flow
- **Payload Processing**: QR scanning triggers a specialized flow in `SharedDataService` that can inject a full folder/BB hierarchy. Testing this flow requires verifying that the UI correctly "expands" and focuses the injected nodes without manual navigation.

### 5. Toritame vs Kotei (Photo App Specific)
- **Distinct Flows**: The `kansuke-photo` app has two completely separate photo capture pipelines:
  - **Kotei (工程)**: Regular progress photos tied strictly to the folder hierarchy. Managed by `KoteiBloc`, uses `koteiTable`, and uploaded via standard `KoteiPhotoService` routines.
  - **Toritame (撮り溜め)**: Batch shooting mode (save for later). This allows taking multiple photos without strict BB/folder mapping immediately. It has its own isolated architecture (`toritame_photo` feature folder, `PhotoListBloc`, `ToritamePreviewRoute`, uses `toritameTable`).
- **AI QA Rule**: Any CR affecting Camera UI, Upload logic, or Image processing in the Photo app **MUST** have test cases split out. You cannot just test "capture a photo". The AI must explicitly write scenarios for:
  - "Take a photo in Kotei mode" AND
  - "Take a photo in Toritame mode".
  If upload UI is changed, verify upload loops for both `Kotei` arrays and `Toritame` arrays separately.
