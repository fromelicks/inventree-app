# AGENTS.md — AI Assistant Guide for inventree-app

This document describes the codebase structure, development conventions, and workflows for AI assistants (Claude, Copilot, etc.) working on this repository.

---

## Project Overview

**inventree-app** is a Flutter mobile application for [InvenTree](https://inventree.org), an open-source stock management system. The app targets Android and iOS, communicates with an InvenTree backend via REST API, supports barcode scanning, and is available in 44 languages.

- **License:** MIT
- **Current version:** 0.24.3+121
- **Flutter version:** 3.41.6 (pinned via FVM — see `.fvmrc`)
- **Dart SDK requirement:** ^3.8.1

---

## Repository Structure

```
inventree-app/
├── lib/                     # All Dart/Flutter source code
│   ├── main.dart            # App entry point
│   ├── api.dart             # REST API client and response handling
│   ├── api_form.dart        # Dynamic form generation from API schema
│   ├── preferences.dart     # Local settings storage (Sembast NoSQL)
│   ├── user_profile.dart    # User authentication and profile management
│   ├── dsn.dart             # Sentry DSN configuration
│   ├── inventree/           # Data models (one file per domain entity)
│   ├── widget/              # UI screens and reusable components
│   ├── barcode/             # Barcode scanning (camera + wedge hardware)
│   ├── settings/            # Settings screen widgets
│   └── l10n/               # ARB translation files (44 locales)
├── test/                    # Unit and widget tests
│   ├── fixtures/            # JSON fixtures for API mock responses
│   └── setup.dart           # Shared test environment initialization
├── android/                 # Android build configuration
├── ios/                     # iOS build configuration
├── assets/                  # Images, audio files, release notes markdown
├── .github/workflows/       # CI/CD: ci.yaml, android.yaml, ios.yaml
├── pubspec.yaml             # Flutter package manifest and dependencies
├── tasks.py                 # Invoke task runner (format/clean/build/translate)
├── analysis_options.yaml    # Dart analyzer + linter configuration
├── l10n.yaml                # Localization config (ARB → app_localizations.dart)
├── .fvmrc                   # Flutter version pin for FVM
├── .pre-commit-config.yaml  # Pre-commit hook: dart format
├── BUILDING.md              # Dev environment setup instructions
└── CONTRIBUTING.md          # Contribution guidelines
```

---

## Technology Stack

| Layer | Technology |
|---|---|
| UI framework | Flutter (Material Design 3) |
| Language | Dart 3.8+ |
| Local storage | Sembast (NoSQL, `preferences.dart`) |
| HTTP client | `http` package |
| Error tracking | Sentry (`sentry_flutter`) |
| Barcode scanning | `mobile_scanner` + wedge hardware support |
| Image handling | `cached_network_image`, `image_picker`, `camera` |
| Localization | Flutter ARB + `intl` (44 languages) |
| Theme | `adaptive_theme` (dark/light/system) |
| Build automation | Invoke (Python) via `tasks.py` |
| Flutter versioning | FVM (Flutter Version Manager) |

---

## Development Setup

All Flutter commands must be prefixed with `fvm` to use the pinned Flutter version.

```bash
# 1. Install FVM (https://fvm.app/documentation/getting-started/installation)
# 2. In the project root:
fvm use                        # Download pinned Flutter SDK, configure IDE
invoke translate               # Generate ARB translation files (required first run)
fvm flutter pub get            # Install Dart dependencies
```

**IDE setup:**
- **VS Code:** Run `mkdir -p .vscode && fvm use` — FVM configures the SDK path automatically.
- **Android Studio:** Run `fvm use`, then set Flutter SDK path to `.fvm/flutter_sdk` in Settings → Languages & Frameworks → Flutter.

**Java requirement:** JDK 17 is required for Android builds. Configure with:
```bash
fvm flutter config --jdk-dir /path/to/jdk17
```

---

## Development Workflows

### Running the app
```bash
fvm flutter run          # Run on connected device/emulator
```

### Running tests
```bash
fvm flutter test                    # All tests
fvm flutter test --coverage         # With coverage report
fvm flutter test test/api_test.dart # Single file
```

### Code formatting (required before committing)
```bash
fvm dart format lib      # Format all Dart source files
invoke format            # Equivalent via invoke task runner
```

### Static analysis
```bash
fvm dart analyze         # Run Dart analyzer
```

### Build automation (via Invoke)
```bash
invoke translate    # Collect/regenerate translation ARB files
invoke clean        # flutter clean
invoke update       # flutter pub get
invoke android      # Full Android release build (clean → update → translate → build)
invoke ios          # Full iOS release build (clean → update → translate → build)
```

---

## Code Conventions

### Formatting
- **Always** run `fvm dart format lib` before committing. The pre-commit hook enforces this, and CI will fail on formatting violations.
- Use **double quotes** for strings (`prefer_double_quotes: true`).
- Place **constructors first** in every class (`sort_constructors_first: true`).
- Strict raw types are enforced (`strict-raw-types: true`).

### Rules that are intentionally disabled
The following linter rules are turned off in `analysis_options.yaml` — do not re-enable them without discussion:
- `prefer_final_locals`, `prefer_final_fields`, `prefer_final_in_for_each`
- `prefer_const_constructors`, `prefer_const_literals_to_create_immutables`
- `always_specify_types`
- `require_trailing_commas`
- `use_build_context_synchronously`
- `avoid_print`

### Excluded from analysis
- `build/**` — generated build artifacts
- `lib/generated/**` — generated localization code

### General Dart style
- Follow Flutter/Dart best practices.
- Write tests for new features where applicable.
- Keep comments focused on the *why*, not the *what*.

---

## Key Source Files

Understanding these files is essential before making changes:

| File | Role |
|---|---|
| `lib/main.dart` | App entry: Sentry init, theme, orientation, localization, navigation |
| `lib/api.dart` | Core HTTP client (`InvenTreeAPI`), response parsing, auth token management |
| `lib/api_form.dart` | Dynamic form generation from InvenTree API field schema |
| `lib/preferences.dart` | All persistent local settings via Sembast; defines keys as constants |
| `lib/user_profile.dart` | Server connection profiles, token-based auth |
| `lib/inventree/model.dart` | `InvenTreeModel` base class — all models extend this |
| `lib/widget/refreshable_state.dart` | `RefreshableStateMixin` — used by most screen widgets |
| `lib/widget/dialogs.dart` | Shared dialog/confirmation utilities |
| `lib/widget/snacks.dart` | Snackbar notification helpers |
| `lib/widget/paginator.dart` | Paginated list view base implementation |
| `lib/barcode/barcode.dart` | Barcode scan dispatch and result handling |

---

## Data Models (`lib/inventree/`)

All models extend `InvenTreeModel` from `lib/inventree/model.dart`. Each file maps to an InvenTree API resource:

| File | Models |
|---|---|
| `part.dart` | `InvenTreePart`, `InvenTreePartCategory`, `InvenTreePartTestTemplate`, `InvenTreePartPricing` |
| `stock.dart` | `InvenTreeStockItem`, `InvenTreeStockLocation` |
| `orders.dart` | Shared order base classes |
| `purchase_order.dart` | `InvenTreePurchaseOrder`, line items |
| `sales_order.dart` | `InvenTreeSalesOrder`, line items |
| `build.dart` | `InvenTreeBuild` (build orders) |
| `company.dart` | `InvenTreeCompany`, `InvenTreeSupplierPart`, `InvenTreeManufacturerPart` |
| `bom.dart` | `InvenTreeBOMItem` |
| `parameter.dart` | `InvenTreeParameter`, `InvenTreeParameterTemplate` |
| `attachment.dart` | `InvenTreeAttachment` |
| `notification.dart` | `InvenTreeNotification` |
| `status_codes.dart` | Status code enum/mapping |
| `project_code.dart` | `InvenTreeProjectCode` |

When adding a new API entity, follow the pattern in an existing model file.

---

## UI Structure (`lib/widget/`)

Screens are organized by domain under subdirectories. Each domain typically has a detail view, list view, and related widgets:

```
widget/
├── build/         # Build order screens
├── company/       # Supplier/manufacturer/customer screens
├── order/         # Purchase & sales order screens
├── part/          # Part detail, category, BOM screens
└── stock/         # Stock item and location screens
```

Most screens use `RefreshableStateMixin` (from `refreshable_state.dart`) which handles pull-to-refresh and async data loading.

---

## Barcode Scanning (`lib/barcode/`)

The barcode system supports two input methods:
- **Camera** (`camera_controller.dart`): Uses `mobile_scanner` for live camera scanning.
- **Wedge** (`wedge_controller.dart`): Hardware barcode scanners that inject keyboard events.

Scan results are routed through `barcode.dart` → `handler.dart` → domain-specific handlers (`stock.dart`, `purchase_order.dart`, `sales_order.dart`).

Audio feedback (success/error tones) is managed by `tones.dart` and configured via preferences.

---

## Localization

- Translation strings live in `lib/l10n/app_en.arb` (English source, ~42KB).
- All 44 locale ARB files are in `lib/l10n/`.
- The generated output class is `I18N` (in `lib/generated/app_localizations.dart`).
- **Never edit generated files** — run `invoke translate` to regenerate.
- New user-visible strings must be added to `app_en.arb` first; translations come from Crowdin contributors.
- Reference strings in Dart as: `I18N.of(context).yourStringKey`

---

## Settings / Preferences

All persistent user settings are stored locally via Sembast in `lib/preferences.dart`. Keys are defined as string constants. Common setting areas:

| Prefix | Area |
|---|---|
| `homeShow*` | Dashboard widget visibility |
| `barcode*` | Barcode scanning behavior |
| `stock*` | Stock display options |
| `part*` | Part display options |
| `label*` | Label printing |
| `appScreen*` | Screen orientation |

When adding new settings, define the key constant in `preferences.dart` and add a default value.

---

## Testing

Tests live in `test/`. The testing setup:

- `test/setup.dart` — shared initialization (call `setupTestEnv()` at the top of each test file)
- `test/fixtures/` — JSON fixtures used to mock API responses

Test files map to source areas:
| Test file | Coverage area |
|---|---|
| `api_test.dart` | REST API client |
| `barcode_test.dart` | Barcode scanning logic |
| `models_test.dart` | Data model serialization/deserialization |
| `preferences_test.dart` | Settings persistence |
| `user_profile_test.dart` | Auth and profile management |
| `wedge_scanner_test.dart` | Hardware scanner input |
| `widget_test.dart` | Flutter widget rendering |

Run tests with: `fvm flutter test --coverage`

---

## CI/CD (`.github/workflows/`)

Three workflows run automatically on every push to `master` and on all pull requests:

### `ci.yaml` — Main CI pipeline
1. Sets up Python 3.11, Java 11, Flutter (via FVM)
2. Runs `invoke translate` to generate translation files
3. **`dart analyze`** — static analysis (must pass)
4. **`dart format --set-exit-if-changed`** — formatting check (must pass)
5. Starts an InvenTree backend server (cloned separately)
6. **`fvm flutter test --coverage`** — runs all tests
7. Uploads coverage to Coveralls

### `android.yaml` — Android build
- Runs on macOS with Java 17
- Builds APK and AAB release artifacts

### `ios.yaml` — iOS build
- Runs on macOS with Xcode
- Builds IPA release artifact

**A PR must pass all three CI checks before merging.**

---

## Pre-commit Hooks

The `.pre-commit-config.yaml` runs `dart format` on all staged Dart files before every commit. Install with:

```bash
pip install pre-commit
pre-commit install
```

If you bypass the hook, CI will catch formatting issues and fail the build.

---

## Common Pitfalls

1. **Forget `invoke translate` on first clone** — the `lib/generated/` directory will be missing and compilation will fail.
2. **Running `flutter` without `fvm`** — uses the system Flutter version instead of the pinned 3.41.6, which may produce unexpected behavior.
3. **Editing `lib/generated/`** — these are generated files; changes will be overwritten. Edit ARB files instead.
4. **JDK version mismatch** — Android builds require JDK 17. Check with `java -version` and configure via `fvm flutter config --jdk-dir`.
5. **Adding user-visible strings without ARB entry** — hardcoded strings bypass the localization system. Always add to `app_en.arb`.

---

## Making Changes — Recommended Approach

1. **Read** the relevant model in `lib/inventree/` and the widget in `lib/widget/` before making UI or API changes.
2. **Check** `lib/api.dart` to understand how HTTP requests are structured — don't bypass the existing client.
3. **Reuse** existing dialog/snackbar utilities from `dialogs.dart` and `snacks.dart`.
4. **Follow** the `RefreshableStateMixin` pattern for new screens that load data from the API.
5. **Add** new settings to `preferences.dart` with a constant key and documented default.
6. **Format** with `fvm dart format lib` and run `fvm dart analyze` before committing.
7. **Write** or update tests in `test/` when adding new logic.
