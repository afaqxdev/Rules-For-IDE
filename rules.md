# Project Rules — Flutter Development Guidelines

## Purpose

This file is written for the AI assistant (Claude/Copilot/Cursor/etc.) generating code in this project. Its main goal: **every piece of AI-generated code must be clean, reusable, and consistent with the rest of the codebase** — as if a senior developer wrote it by hand, not as a one-off patch.

Before writing or editing any file, the AI must:
- Read this file first. Treat these rules as project-level constraints unless they conflict with an explicit task requirement or an established working pattern in the codebase.
- Never generate a screen, widget, or function that violates a rule below "just to get it working" — if a shortcut is tempting, stop and structure it correctly instead.
- Prefer extending or reusing an existing widget/constant/util over creating a near-duplicate. Search the relevant folder (`shared/components`, `core/constants`, feature `widgets/`) before creating something new.
- Do not hardcode project-level design values when an equivalent centralized value exists. Reuse theme/design-system values whenever possible. Repeated or meaningful values should be centralized.
- If a rule can't be followed for a specific case, say so explicitly and explain why, rather than silently breaking it.

These rules apply to every screen, widget, and feature in this project. They exist to keep the codebase consistent, scalable, reusable, and easy to hand off between developers — human or AI.

---

## 0. Workflow — Every Time You're Asked to Create a Screen

Do this sequence before writing a single line of the new screen:

1. **Re-read this file.** Confirm folder structure, size limit, theming, and localization rules in your head before planning anything.
2. **Inspect the existing project first**, not just this doc:
   - Look inside `core/constants/` (`app_colors.dart`, `app_text_styles.dart`, `app_images.dart`, `app_dimens.dart`) and `core/theme/` to see what already exists — use it, don't redefine it.
   - Look inside `shared/components/` and the target feature's `widgets/` folder for buttons, inputs, cards, or layouts you can reuse instead of building new ones.
   - Check `l10n/app_en.arb` for existing string keys before adding new ones — reuse a key if one already fits (e.g. a shared "Save" or "Cancel" label).
   - Check the feature's existing viewmodel/controller/cubit and repository — don't duplicate a data-fetch method that already exists.
3. **Plan the file breakdown before coding**: if the screen looks like it'll exceed ~170–200 lines, decide up front which sections become separate widget files (header, list item, form, bottom sheet) rather than writing one big file and splitting it after.
4. **Identify what's new vs. reused**: state explicitly (to yourself or in a short note to the user) which colors/strings/widgets are being reused from existing files and which are genuinely new and need to be added to constants/`.arb`/`shared/components`.
5. **Implement following the rules in sections 1–16 below.**
6. **Self-check against the "Before Marking a Screen Done" checklist** (section 11) before presenting the result.

If the project structure described in section 1 doesn't exist yet (e.g. this is the very first screen in a new project), create it first — scaffold `core/`, `l10n/`, `shared/`, and the feature folder — rather than dropping a screen file into an ad hoc location.

---

## 1. Project Structure

```
lib/
├── main.dart
├── app/
│   ├── app.dart                 # MaterialApp / router setup
│   └── routes/
│       ├── app_routes.dart      # Route name constants
│       └── app_router.dart      # Route generation
├── core/
│   ├── constants/
│   │   ├── app_colors.dart
│   │   ├── app_text_styles.dart
│   │   ├── app_images.dart      # All image/asset paths as consts
│   │   ├── app_icons.dart
│   │   ├── app_dimens.dart      # Spacing, radius, sizing constants
│   │   └── app_strings.dart     # Non-localized static keys (if any)
│   ├── theme/
│   │   ├── app_theme.dart       # ThemeData (light/dark)
│   │   ├── app_fonts.dart       # Font family definitions
│   │   └── app_theme_extension.dart
│   ├── utils/
│   │   ├── validators.dart
│   │   ├── formatters.dart
│   │   └── extensions/
│   ├── network/
│   │   ├── api_client.dart
│   │   └── api_endpoints.dart
│   └── widgets/                 # Shared/reusable widgets used app-wide
├── l10n/
│   ├── app_en.arb
│   ├── app_<lang>.arb
│   └── app_localizations.dart   # Generated
├── data/
│   ├── models/
│   ├── repositories/
│   └── services/
├── features/
│   └── <feature_name>/
│       ├── models/
│       ├── viewmodels/ (or controllers/, cubits/, blocs/ — per chosen state pattern)
│       ├── widgets/             # Widgets used only within this feature
│       └── screens/
│           └── <feature_name>_screen.dart
└── shared/
    └── components/              # Buttons, inputs, cards, dialogs used across features
```

**Rules:**
- One feature = one folder under `features/`. Never mix unrelated feature logic in `core/` or `shared/`.
- A widget used in more than one feature moves to `shared/components/`; a widget used only within one screen's feature stays local to that feature's `widgets/` folder.
- No business logic inside `screens/` files — screens only compose widgets and bind to a viewmodel/controller/cubit.

---

## 2. Screen File Size & Complexity

- **Target: 170–200 lines. If a screen exceeds ~200 lines, review it and extract meaningful sections where appropriate** (excluding imports and generated code).
- If a screen grows beyond this, break it into smaller private widgets or extract sections into separate files under that feature's `widgets/` folder (e.g. `_profile_header.dart`, `_stats_section.dart`).
- A `build()` method should not itself sprawl — extract any section repeated or logically distinct (headers, list items, forms, bottom sheets) into its own widget class.
- Business/data logic never lives in the screen file — it belongs in the viewmodel/controller layer.

---

## 3. Separation of Concerns

- **UI (screens/widgets)** — layout and presentation only. No API calls, no direct Firestore/DB access, no business rules.
- **ViewModel / Controller / Cubit** — holds state, calls repositories/services, exposes data and actions to the UI.
- **Repository** — abstracts data sources (Firestore, REST API, local DB). Screens and viewmodels never call Firebase/SDKs directly.
- **Service** — wraps third-party SDKs (RevenueCat, notifications, analytics, etc.) behind a clean interface.
- **Model** — plain data classes with `fromJson`/`toJson`/`copyWith`, no logic beyond parsing/serialization.

A screen should never import `cloud_firestore`, `http`, or any SDK package directly — only its viewmodel/repository does.

---

## 4. Theming & Design System

- Do not hardcode project-level design values when an equivalent centralized value exists. Reuse theme/design-system values whenever possible. Repeated or meaningful values should be centralized:
  - Colors → `AppColors.xxx` or `Theme.of(context).colorScheme.xxx`
  - Text styles → `AppTextStyles.xxx` or `Theme.of(context).textTheme.xxx`
  - Meaningful spacing/radius constants → `AppDimens.xxx`
- Font family is defined once in `app_fonts.dart` and wired into `ThemeData`. Widgets use `Theme.of(context).textTheme` or the shared `AppTextStyles`, never a raw `TextStyle(fontFamily: 'X')`.
- Support both light and dark theme via `ThemeData` (or a `ThemeExtension`) — do not branch UI code with manual `if (isDark)` color checks scattered through screens.
- **Responsive Layout**: Avoid device-specific hardcoded dimensions. Use the project's responsive system and Flutter's adaptive layout widgets where appropriate. Design-system spacing, icon sizes, radii, and similar constants may use fixed logical pixels when intentionally defined by the design system.

---

## 5. Assets & Images

- Every image/icon path is declared as a constant in `app_images.dart` / `app_icons.dart`. Never reference an asset path as a raw string inline (`Image.asset('assets/img/logo.png')` is not allowed).
- Group constants logically (e.g. `AppImages.onboarding`, `AppImages.icons`) if the asset list grows large.
- Keep `pubspec.yaml` asset declarations organized by folder, matching the constants file structure.

---

## 6. Localization

- **If localization is enabled in the project, every screen and user-facing feature must support localization** — no hardcoded user-facing strings anywhere in the UI.
- All text goes through `AppLocalizations.of(context)!.xxx` (or the project's chosen l10n access pattern).
- Add new keys to the base localization file and update all supported locale files according to the project's localization workflow. Never leave user-facing text hardcoded.
- Never concatenate translated strings with string interpolation in a way that breaks grammar across languages — use ICU placeholders in the `.arb` entry instead (e.g. `"Hello, {name}"`).

---

## 7. State Management

- **Use the project's established state-management and presentation architecture.** For projects using Provider + MVVM, follow Provider + MVVM consistently within that project. Do not introduce BLoC, GetX, Riverpod, or another competing pattern unless explicitly required.
- Business logic and side effects (API calls, navigation triggers, validation) live in the viewmodel/controller, not in widget callbacks.
- Expose state to the UI via reactive streams/observables/notifiers — widgets should rebuild only the smallest necessary subtree (use scoped selectors/builders, avoiding full-screen rebuilds).

---

## 8. Data & Backend Readiness

- Build components to receive real data via passed parameters/models — never wire dummy or placeholder data directly into a widget's `build()`.
- Use repository abstractions when they provide meaningful separation, testability, or the ability to swap data sources. Do not create interfaces solely for the sake of abstraction. UI and presentation layers should depend on repository contracts rather than concrete external data sources where the project architecture requires it.

---

## 9. Performance

- Animations must run smoothly with no dropped frames — avoid heavy work in `build()`, use `const` constructors wherever possible, and avoid unnecessary `setState`/rebuild triggers.
- Use `ListView.builder`/`GridView.builder` (not `.map()` into a `Column`) for any list that could grow.
- Debounce/throttle expensive listeners (search fields, scroll listeners, Firestore streams).

---

## 10. Naming & Style

- File names: `snake_case.dart`. Class names: `PascalCase`. Variables/functions: `camelCase`.
- Screen files end in `_screen.dart`, widgets in a descriptive name (no generic `widget1.dart`).
- Private widgets extracted from a screen are prefixed with `_` and kept in the same file only if small; otherwise moved to their own file.
- Run `flutter analyze` and fix all lints before committing. No `// ignore:` without a one-line comment explaining why.

---

## 11. Before Marking a Screen "Done"

- [ ] Screen complexity has been reviewed; large or complex sections are appropriately extracted
- [ ] Centralized theme tokens and design constants reused where available
- [ ] All strings localized (if project has localization enabled)
- [ ] Layout is adaptive/responsive; no brittle device-specific hardcoded dimensions
- [ ] No direct SDK/API calls in the screen — goes through viewmodel/repository
- [ ] Works with real/backend-shaped data, not fake placeholders
- [ ] No dropped frames on scroll/animation
- [ ] Passes `flutter analyze` clean

---

## 12. Scope Control

- Only modify files necessary to complete the requested task.
- Do not refactor unrelated code unless explicitly requested.
- Do not rename existing classes, files, variables, or folders without a clear reason.
- Do not introduce a new architecture while implementing a small feature.
- If an existing implementation is imperfect but unrelated to the requested task, leave it unchanged unless it directly blocks the task.

---

## 13. Rule Priority

When making implementation decisions, follow this priority:

1. **Existing working project architecture and conventions**
2. **Explicit requirements in the current task / design**
3. **This rules.md**
4. **General best practices**

```
Existing working architecture
        ↓
Explicit task/design requirements
        ↓
Project rules (rules.md)
        ↓
General best practices
```

Never break an existing working feature simply to satisfy a generic rule in this document.

### Core Principles
- **Reuse > duplicate**
- **Existing architecture > invented architecture**
- **Design system > random styling**
- **Real implementation > fake data**
- **Separation of concerns > quick patches**
- **Simple solution > over-engineering**
- **Task scope > unrelated refactoring**

---

## 14. Design Fidelity

- When implementing a screen from an existing design, screenshot, Figma file, or specification, reproduce the intended layout and hierarchy rather than inventing a different UI.
- Reuse the project's existing design tokens and components wherever they match the design.
- Do not replace specified components with visually different alternatives simply because they are easier to implement.
- If the design conflicts with the existing design system, follow the explicit task/design requirement and update the design system when appropriate rather than introducing isolated styling.

---

## 15. No Fake or Placeholder Functionality

- Do not create fake API responses, hardcoded user data, fake loading states, or placeholder business logic unless explicitly requested.
- If a required backend/service is unavailable, clearly identify the missing dependency instead of pretending the feature is functional.
- UI prototypes may use mock data only when the task explicitly requires a prototype or mockup.
- Never present mock data as real application data.

---

## 16. Dependency Changes

- Do not add, remove, or replace packages without a clear reason.
- Before adding a dependency, check whether the required functionality already exists in Flutter, Dart, or the current project dependencies.
- If a dependency is necessary, use the project's existing conventions and compatible versions.
- Do not replace an existing package simply because another package is preferred personally.
