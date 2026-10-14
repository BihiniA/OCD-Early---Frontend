# OCD-Early — AI Screening System

An intelligent, dual-perspective (Adolescent + Parent) screening application built with Flutter to assist in the early detection and pre-identification of Obsessive-Compulsive Disorder (OCD) patterns in young individuals aged 5 to 19.

The main Flutter project source code is located in the [`ocd_early/`](./ocd_early/) directory.

---

## Table of Contents
1. [Project Overview](#project-overview)
2. [lib Folder Structure](#lib-folder-structure)
3. [Navigation List & Screen Flows](#navigation-list--screen-flows)
   - [Navigation Architecture](#navigation-architecture)
   - [Complete Navigation Matrix](#complete-navigation-matrix)
   - [Detailed User Flows](#detailed-user-flows)
4. [Setup & Installation Instructions](#setup--installation-instructions)
   - [Prerequisites](#prerequisites)
   - [Installation Steps](#installation-steps)
   - [Backend API Configuration](#backend-api-configuration)
   - [Running the App](#running-the-app)
   - [Building for Production](#building-for-production)
   - [Troubleshooting & Common Issues](#troubleshooting--common-issues)
5. [Architecture & Scoring Model](#architecture--scoring-model)

---

## Project Overview

**OCD-Early** provides a structured, accessible questionnaire for both adolescents and their parents. It scores symptoms across key dimensions (contamination, checking, symmetry/order, counting, hoarding, intrusive thoughts) and computes a dual-score mismatch index. The compiled data is submitted to a predictive machine learning endpoint to estimate risk levels (**Low**, **Moderate**, or **High**) alongside actionable guidance and history tracking.

> **Disclaimer**: *OCD-Early is a pre-identification screening tool and not a clinical diagnostic instrument. A licensed healthcare or mental health professional must be consulted for formal diagnoses and treatments.*

---

## lib Folder Structure

The core Dart source code resides in [`ocd_early/lib/`](./ocd_early/lib/) and follows a clean, feature-driven separation of concerns:

```
ocd_early/lib/
├── config/
│   └── app_config.dart                # Build-time backend environment and URL configuration
├── data/
│   └── questions_data.dart            # Question datasets (18 adolescent & 18 parent questions)
├── models/
│   ├── question_model.dart            # Question data model and QuestionSection enum
│   └── screening_result_model.dart    # Model for historical screening results and scores
├── navigation/
│   └── main_navigation.dart           # Root tab controller using IndexedStack & bottom nav
├── screens/
│   ├── adolescent_questions_screen.dart # Adolescent questionnaire screen wrapper
│   ├── age_input_screen.dart          # Form screen for age validation (5-19 years)
│   ├── dashboard_screen.dart          # Primary home screen with drawer & quick actions
│   ├── help_support_screen.dart       # Help, guidelines, and support resources
│   ├── history_screen.dart            # View list of completed assessments & score cards
│   ├── information_screen.dart        # Educational guide on OCD signs, causes & care
│   ├── parent_questions_screen.dart   # Parent questionnaire screen & completion check
│   ├── processing_screen.dart         # Loading/API submission screen with error retry
│   ├── profile_screen.dart            # User profile, photo picker, and settings
│   ├── questionnaire_screen.dart      # Reusable question pager component (Next/Prev/Score)
│   ├── results_screen.dart            # Displays risk score, breakdowns, and recommendations
│   ├── screening_intro_screen.dart    # Instructions & guidelines before starting test
│   └── welcome_screen.dart            # Initial splash/onboarding landing screen
├── services/
│   ├── api_service.dart               # HTTP client communicating with prediction API
│   └── local_storage.dart             # Persists profile, photo, and screening history locally
├── state/
│   └── screening_state.dart           # Global ChangeNotifier managing answers & scoring
├── theme/
│   └── app_theme.dart                 # App colors, typography, Material 3 ThemeData
├── widgets/
│   ├── custom_button.dart             # Reusable rounded primary/secondary button
│   ├── navigation_bar.dart            # Custom bottom navigation bar widget
│   ├── progress_indicator.dart        # Linear progress bar tracking questionnaire progress
│   └── question_card.dart             # Card UI rendering question text and 0-4 Likert scale
└── main.dart                          # Application entry point with Provider & MaterialApp
```

### Detailed Component Breakdown

| Directory / File | Description & Key Responsibilities |
| :--- | :--- |
| **`main.dart`** | Entry point of the Flutter application. Restores saved profile and screening history before initializing `ScreeningState` with `ChangeNotifierProvider`, then mounts `WelcomeScreen` or `MainNavigation`. |
| **`config/app_config.dart`** | Selects the backend base URL from the build-time `--dart-define=ENV=<name>` value (`emulator`, `usb`, `wifi`, `web`, or `prod`). |
| **`data/questions_data.dart`** | Holds static question lists: `adolescentQuestions` (18 questions) and `parentQuestions` (18 questions), each using a shared 0–4 Likert scale (`Not at all` to `Extremely`). |
| **`models/question_model.dart`** | Defines `QuestionSection` enum (`adolescent`, `parent`) and the `Question` model class holding the question text. |
| **`models/screening_result_model.dart`** | Defines `ScreeningResult` storing `riskLevel`, `riskScore`, `childTotal`, `parentTotal`, `mismatch`, and timestamp `date`. |
| **`navigation/main_navigation.dart`** | Hosts the bottom navigation bar (`AppNavigationBar`) and preserves screen state across 4 main tabs (`Dashboard`, `History`, `Profile`, `Information`) using an `IndexedStack`. |
| **`screens/welcome_screen.dart`** | First screen presented to the user. Features hero branding image, introduction, "Get Started" button (enters app), and "Learn More" button. |
| **`screens/dashboard_screen.dart`** | Home dashboard displaying welcome banner, quick action cards (Start Screening, View Results, Information, Help & Support), notifications dialog, and drawer menu. |
| **`screens/screening_intro_screen.dart`** | Onboarding overview of the dual assessment process (18 questions each, estimated duration, privacy reminder). |
| **`screens/age_input_screen.dart`** | Collects the adolescent's age with boundary validation (5 to 19 years old). Stores value in `ScreeningState`. |
| **`screens/adolescent_questions_screen.dart`** | Configures `QuestionnaireScreen` with `adolescentQuestions` and routes to parent questions upon completion. |
| **`screens/parent_questions_screen.dart`** | Configures `QuestionnaireScreen` with `parentQuestions`. Validates that both child and parent forms are complete before pushing to `ProcessingScreen`. |
| **`screens/questionnaire_screen.dart`** | Generic interactive question pager. Manages current question index, Next/Previous controls, progress indication, and answer selection. |
| **`screens/processing_screen.dart`** | Shows a loading animation while invoking `ApiService.submitScreening(state)`. Handles socket timeouts, network errors, and server exceptions with retry options. |
| **`screens/results_screen.dart`** | Presents the computed risk level, adolescent score (out of 72), parent score (out of 72), calculated risk score, guidance recommendations, and options to save to history or return to dashboard. |
| **`screens/history_screen.dart`** | Lists past screening records restored from local storage. Tapping a card opens a modal dialog with full score breakdowns. Includes empty-state view. |
| **`screens/profile_screen.dart`** | Displays user details, avatar change (camera/gallery using `image_picker`), profile editing modal, privacy dialog, about modal, logout option, and Delete my data action. |
| **`screens/information_screen.dart`** | Educational screen with expandable sections on OCD overview, common symptoms, causes, treatments, and external resources. |
| **`screens/help_support_screen.dart`** | Detailed walkthrough on interpreting scores, support hotlines, parental guidance, and confidentiality principles. |
| **`services/api_service.dart`** | Handles POST requests to the AI prediction backend (`/api/predict`). Serializes answers (`Age`, `Child_Q1..Q18`, `Parent_Q1..Q18`) and parses nested prediction risk levels (`Low`, `Moderate`, `High`). |
| **`services/local_storage.dart`** | Uses `shared_preferences` for profile and history data, and the application documents directory for copied profile photos. Supports clearing all saved user data. |
| **`state/screening_state.dart`** | Central state store (`ChangeNotifier`). Holds answers, age, profile details, history list, calculates total scores, adolescent/parent normalized scores, response mismatch index, and final risk metric. Persists profile changes and saved results. |
| **`theme/app_theme.dart`** | Defines consistent color scheme (`primaryBlue`, `darkBlue`, `softBlue`, `borderColor`, etc.), typography standards, and Material 3 theme data. |
| **`widgets/custom_button.dart`** | Standardized rounded button supporting filled and outlined variants, loading states, and icon placements. |
| **`widgets/navigation_bar.dart`** | Bottom navigation bar displaying 4 tabs: Home, History, Profile, and More with active indicator styling. |
| **`widgets/progress_indicator.dart`** | Reusable linear bar reflecting questionnaire completion percentage. |
| **`widgets/question_card.dart`** | Questionnaire card rendering item index, question text, guidance note, and five selectable Likert scale options. |

---

## Navigation List & Screen Flows

### Navigation Architecture

The app uses Flutter's imperative `Navigator` paired with an `IndexedStack` inside `MainNavigation` for persistent tab switching without resetting screen states.

```
                              [WelcomeScreen]
                                     │
                     ┌───────────────┴───────────────┐
             (Get Started)                     (Learn More)
                     │                               │
                     ▼                               ▼
             [MainNavigation]                [InformationScreen]
             (IndexedStack)
       ┌─────────────┬─────────────┬─────────────┐
       │             │             │             │
     Tab 0         Tab 1         Tab 2         Tab 3
[Dashboard]      [History]     [Profile]    [Information]
       │
   Quick Action: "Start Screening"
       │
       ▼
[ScreeningIntroScreen]
       │ (Continue)
       ▼
[AgeInputScreen]
       │ (Continue)
       ▼
[AdolescentQuestionnaireScreen] ──(18 Questions)──┐
                                                  │
                                                  ▼
                                      [ParentQuestionScreen] ──(18 Questions)──┐
                                                                               │
                                                                               ▼
                                                                     [ProcessingScreen]
                                                                               │ (API Response)
                                                                               ▼
                                                                        [ResultsScreen]
                                                                        ┌──────┴──────┐
                                                                 (Save / History)  (Dashboard)
                                                                        │             │
                                                                        ▼             ▼
                                                                  [HistoryScreen]  [MainNavigation]
```

### Complete Navigation Matrix

| Origin Screen | User Action / Trigger | Destination Screen | Navigation Method | Data Passed / Notes |
| :--- | :--- | :--- | :--- | :--- |
| **`WelcomeScreen`** | Tap "Get Started" | `MainNavigation` | `Navigator.pushReplacement` | Enters main app at Tab 0 (`DashboardScreen`). Discards welcome screen from stack. |
| **`WelcomeScreen`** | Tap "Learn More" | `InformationScreen` | `Navigator.push` | Pushes information screen onto stack with back navigation to welcome. |
| **`MainNavigation`** | Tap Bottom Nav item (0) | `DashboardScreen` | Tab State switch (`_currentIndex = 0`) | Switches view within `IndexedStack`. |
| **`MainNavigation`** | Tap Bottom Nav item (1) | `HistoryScreen` | Tab State switch (`_currentIndex = 1`) | Switches view within `IndexedStack`. |
| **`MainNavigation`** | Tap Bottom Nav item (2) | `ProfileScreen` | Tab State switch (`_currentIndex = 2`) | Switches view within `IndexedStack`. |
| **`MainNavigation`** | Tap Bottom Nav item (3) | `InformationScreen` | Tab State switch (`_currentIndex = 3`) | Switches view within `IndexedStack`. |
| **`DashboardScreen`** | Tap Drawer menu icon | Open Drawer | Scaffold Drawer | Opens side navigation drawer. |
| **`DashboardScreen` (Drawer)** | Tap "Home" | Closes Drawer | `Navigator.pop` | Stays on Dashboard. |
| **`DashboardScreen` (Drawer)** | Tap "Information" | `InformationScreen` | `Navigator.pop` then `Navigator.push` | Opens information screen. |
| **`DashboardScreen` (Drawer)** | Tap "Profile" | `ProfileScreen` | `Navigator.pop` then `Navigator.push` | Opens profile screen. |
| **`DashboardScreen` (Drawer)** | Tap "Logout" | `WelcomeScreen` | `Navigator.of(context).pushAndRemoveUntil` | Shows confirmation dialog; on confirmation, clears all routes and returns to welcome. |
| **`DashboardScreen`** | Tap Notifications bell icon | Notifications Dialog | `showDialog` | Displays modal dialog: "No new notifications at the moment." |
| **`DashboardScreen`** | Tap "Start Screening" card | `ScreeningIntroScreen` | `Navigator.push` | Begins new assessment flow. |
| **`DashboardScreen`** | Tap "View Results" card | `ResultsScreen` | `Navigator.push` | Passes `riskLevel: 'Not available'` (direct preview mode). |
| **`DashboardScreen`** | Tap "Information" card | `InformationScreen` | `Navigator.push` | Opens information overview screen. |
| **`DashboardScreen`** | Tap "Help & Support" card | `HelpSupportScreen` | `Navigator.push` | Opens help, hotlines, and support guidelines. |
| **`ScreeningIntroScreen`** | Tap Back button | Previous screen | `Navigator.pop` | Returns to Dashboard. |
| **`ScreeningIntroScreen`** | Tap "Continue" | `AgeInputScreen` | `Navigator.push` | Proceeds to age collection screen. |
| **`AgeInputScreen`** | Tap Back button | `ScreeningIntroScreen` | `Navigator.pop` | Returns to intro screen. |
| **`AgeInputScreen`** | Enter valid age (5–19) & tap "Continue" | `AdolescentQuestionnaireScreen` | `Navigator.push` | Saves age in `ScreeningState` via `setAge()`, starts adolescent questionnaire. |
| **`AdolescentQuestionnaireScreen`** | Tap Back on question 1 | `AgeInputScreen` | `Navigator.pop` | Returns to age input screen. |
| **`AdolescentQuestionnaireScreen`** | Answer 18 items & tap "Continue" | `ParentQuestionScreen` | `Navigator.push` | Advances to parent questionnaire phase. |
| **`ParentQuestionScreen`** | Tap Back on question 1 | `AdolescentQuestionnaireScreen` | `Navigator.pop` | Returns to adolescent questions. |
| **`ParentQuestionScreen`** | Tap "Submit" with incomplete answers | Incomplete Dialog | `showDialog` | Identifies missing question indices. |
| **`ParentQuestionScreen`** | Tap "Submit" with all answers completed | `ProcessingScreen` | `Navigator.push` | Submits full payload to backend. |
| **`ProcessingScreen`** | Successful API response | `ResultsScreen` | `Navigator.pushReplacement` | Passes received `riskLevel` (`Low`, `Moderate`, or `High`). Replaces processing screen. |
| **`ProcessingScreen`** | Network / Server error | Retry prompt on screen | In-screen state update | Displays error message and "Try Again" button to re-trigger submission. |
| **`ResultsScreen`** | Tap Back button | Previous screen | `Navigator.pop` | Pops result screen. |
| **`ResultsScreen`** | Tap "Save Result" | In-screen state update | `state.addHistoryEntry()` | Saves current assessment into history list. Shows confirmation SnackBar. |
| **`ResultsScreen`** | Tap "View History" | `HistoryScreen` | `Navigator.push` | Opens screening history screen. |
| **`ResultsScreen`** | Tap "Back to Dashboard" | `MainNavigation` | `Navigator.pushAndRemoveUntil` | Resets navigation stack back to root `MainNavigation`. |
| **`HistoryScreen`** | Tap Back button | Dashboard or pops route | Custom `onBack()` or `Navigator.pop` | Handles both tab embedded and pushed navigation. |
| **`HistoryScreen`** | Tap History item card | Result Details Dialog | `showDialog` | Shows full breakdown (scores, response mismatch, timestamp). |
| **`ProfileScreen`** | Tap Back button | Dashboard or pops route | Custom `onBack()` or `Navigator.pop` | Handles both tab embedded and pushed navigation. |
| **`ProfileScreen`** | Tap Profile Avatar | Photo Picker Modal | `showModalBottomSheet` | Options to pick from Camera or Gallery. |
| **`ProfileScreen`** | Tap "Edit Profile" | Edit Profile Dialog | `showDialog` | Modifies name, age, and role. |
| **`ProfileScreen`** | Tap "Personal Information" | Info Dialog | `showDialog` | View-only display of profile values. |
| **`ProfileScreen`** | Tap "Privacy & Security" | Privacy Dialog | `showDialog` | Displays data privacy terms. |
| **`ProfileScreen`** | Tap "About OCD-Early" | About Dialog | `showDialog` | Displays app version and mission statement. |
| **`ProfileScreen`** | Tap "Logout" | `WelcomeScreen` | `Navigator.of(context).pushAndRemoveUntil` | Confirmation dialog leading to reset to welcome screen. |
| **`InformationScreen`** | Tap Back button | Dashboard or pops route | Custom `onBack()` or `Navigator.pop` | Handles both tab embedded and pushed navigation. |
| **`HelpSupportScreen`** | Tap Back button | Dashboard | `Navigator.pop` | Returns to Dashboard. |

---

## Setup & Installation Instructions

### Prerequisites

Ensure you have the following installed on your development machine:

1. **Flutter SDK**: Version `3.13.4` or later (tested on Flutter `3.47.5`).
   - Check version:
     ```bash
     flutter --version
     ```
2. **Dart SDK**: Compatible version (bundled automatically with Flutter).
3. **Target Platform Environment**:
   - **Android**: Android Studio with Android SDK (API 33+), command-line tools, and an Android Virtual Device (AVD) or physical device with USB debugging enabled.
   - **iOS / macOS**: Xcode 15+ and CocoaPods (on macOS only).
   - **Windows Desktop**: Visual Studio 2022 with "Desktop development with C++" workload installed.
   - **Web**: Google Chrome browser.
4. **Git**: Installed and configured on your system path.

---

### Installation Steps

1. **Navigate to the Project Directory**:
   ```bash
   cd ocd_early
   ```

2. **Verify Flutter Environment**:
   Run Flutter doctor to confirm that all required toolchains are satisfied:
   ```bash
   flutter doctor
   ```

3. **Install Package Dependencies**:
   Fetch all required packages declared in `pubspec.yaml` (including `provider`, `http`, `image_picker`, `shared_preferences`, `path_provider`, and `cupertino_icons`):
   ```bash
   flutter pub get
   ```

4. **Verify Assets**:
   Confirm that the application assets exist at:
   ```
   ocd_early/assets/images/ocd_logo.png
   ocd_early/assets/images/app_logo.jpg
   ```
   `ocd_logo.png` is used in the welcome and dashboard screens. `app_logo.jpg` is used to generate the Android launcher icon.

---

### Backend API Configuration

The application communicates with a prediction server to process questionnaires.

1. Open `ocd_early/lib/config/app_config.dart`.
2. Select an environment at run/build time with `--dart-define=ENV=<name>`. Available environments are:
   - **`emulator`**: `http://10.0.2.2:5000` for an Android emulator.
   - **`usb`**: `http://127.0.0.1:5000` after running `adb reverse tcp:5000 tcp:5000`.
   - **`wifi`**: The configured PC network address (currently `http://192.168.8.131:5000`); update it if the PC IP changes.
   - **`web`**: `http://127.0.0.1:5000` for Chrome or Windows desktop.
   - **`prod`**: The deployed HTTPS endpoint used by release builds when no environment is supplied.

The selected base URL is combined with the `/api/predict` path in `api_service.dart`; the URL is fixed when the app is built.

#### Expected Request Format (JSON POST)
```json
{
  "Age": 14,
  "Child_Q1": 2,
  "Child_Q2": 1,
  "...": 0,
  "Child_Q18": 3,
  "Parent_Q1": 2,
  "Parent_Q2": 2,
  "...": 1,
  "Parent_Q18": 4
}
```

#### Expected Response Format (JSON)
```json
{
  "success": true,
  "prediction": {
    "risk_level": "Low"
  }
}
```
*(Accepted `risk_level` values: `"Low"`, `"Moderate"`, or `"High"`)*

### Local Data and Privacy

Profile name, age, role, copied profile photo path, and saved screening history are restored from local device storage when the app starts. The **Delete my data** action removes the saved profile, screening history, and copied profile photo from the device. Screening results remain a pre-identification aid and are not a medical diagnosis.

---

### Running the App

Check available devices:
```bash
flutter devices
```

Run on your selected target (from inside `ocd_early/`):
- **Default Device**:
  ```bash
  flutter run
  ```
- **Google Chrome (Web)**:
  ```bash
  flutter run -d chrome
  ```
- **Windows Desktop**:
  ```bash
  flutter run -d windows
  ```
- **Android Device / Emulator**:
  ```bash
  flutter run -d <device-id>
  ```

---

### Building for Production

#### Android APK
```bash
flutter build apk --release
```
Output path: `build/app/outputs/flutter-apk/app-release.apk`

#### Android App Bundle (for Google Play Store)
```bash
flutter build appbundle --release
```
Output path: `build/app/outputs/bundle/release/app-release.aab`

#### Windows Executable
```bash
flutter build windows --release
```
Output path: `build/windows/runner/Release/`

#### Web Build
```bash
flutter build web --release
```
Output path: `build/web/`

---

### Troubleshooting & Common Issues

| Issue | Cause | Resolution |
| :--- | :--- | :--- |
| **`SocketException` / No Internet** | App cannot reach the backend endpoint in `ProcessingScreen`. | Verify your network connection and ensure the selected URL in `config/app_config.dart` is reachable. On Android emulators, use the `emulator` environment (`10.0.2.2`). |
| **`TimeoutException` (15s limit)** | Backend server is slow or sleeping (e.g., cold starts on cloud free tiers). | Check server logs or increase timeout duration in `api_service.dart`. |
| **Asset not found (`ocd_logo.png`)** | Asset path mismatch in `pubspec.yaml`. | Ensure `assets/images/ocd_logo.png` is present and rerun `flutter pub get`. |
| **Camera/Gallery permissions** | `image_picker` requires camera/storage permissions on physical devices. | For Android, ensure `READ_EXTERNAL_STORAGE` / `CAMERA` permissions are declared in `AndroidManifest.xml` if targeting older APIs. |

---

## Architecture & Scoring Model

Inside `ScreeningState` (`lib/state/screening_state.dart`), scores are calculated with clinical normalization:

1. **Child Total**: $\sum_{i=1}^{18} \text{Child\_Q}_i \in [0, 72]$
2. **Parent Total**: $\sum_{i=1}^{18} \text{Parent\_Q}_i \in [0, 72]$
3. **Normalized Scores**:
   $$\text{TeenNorm} = \frac{\text{Child Total}}{72}, \quad \text{ParentNorm} = \frac{\text{Parent Total}}{72}$$
4. **Mismatch Index**:
   $$\text{Mismatch} = \frac{1}{18 \times 4} \sum_{i=1}^{18} |\text{Child\_Q}_i - \text{Parent\_Q}_i| \in [0, 1]$$
5. **Final Composite Risk Score**:
   $$\text{FinalRiskScore} = 0.6 \times \text{TeenNorm} + 0.4 \times \text{ParentNorm} + 0.25 \times \text{Mismatch}$$

This formula factors in both youth self-reporting, parental observation, and divergence between the two perspectives.

## How to run

### One-time: pair the phone over Wi-Fi (Android 11+)

On the phone: Settings → Developer options → Wireless debugging → **Pair device with pairing code**.

```
adb pair <PHONE_IP>:<PAIRING_PORT>   # e.g. adb pair 192.168.8.165:42155, then enter the 6-digit code
```

Pairing is remembered until you tap Forget on the phone.

### Every session: connect the phone

Use the **IP address & Port** on the main Wireless debugging screen. It is a different port from the pairing port, and it changes whenever Wireless debugging is toggled or the phone rejoins Wi-Fi.

```
adb connect <PHONE_IP>:<CONNECT_PORT>   # e.g. adb connect 192.168.8.165:37251
adb devices                             # phone should show as "device"
adb disconnect                          # when finished, or before reconnecting with a new port
```

### Run the app (choose one method)

The backend URL is chosen with `--dart-define=ENV=<name>`.

```
flutter run                                  # emulator
flutter run --dart-define=ENV=usb            # USB phone (run adb reverse tcp:5000 tcp:5000 first)
flutter run --dart-define=ENV=wifi           # Wi-Fi phone (PC and phone on same network)
flutter run -d chrome --dart-define=ENV=web  # web
```

If several devices are connected, add `-d <PHONE_IP>:<CONNECT_PORT>` to `flutter run`.

### Build and install an APK

The URL is baked into the APK, so build with the flag that matches where the backend runs.

```
flutter build apk --release --dart-define=ENV=wifi   # APK that talks to the local PC server
flutter build apk --release                          # production APK (uses the 'prod' URL)
adb install -r build\app\outputs\flutter-apk\app-release.apk
```

### Server checklist (Wi-Fi mode)

- Server runs with `host="0.0.0.0"`, port 5000
- PC IP matches the `wifi` URL in `app_config.dart` (check with `ipconfig`). If the IP changes, update the URL and rebuild
- Phone and PC are on the same Wi-Fi
- Windows Firewall allows port 5000