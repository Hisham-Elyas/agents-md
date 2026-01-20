# Flutter BLoC Professional Application - Agent Documentation

## Overview

This document provides a comprehensive guide for AI agents and developers working on the **Professional** version of the Flutter application using **BLoC**. It combines the power of BLoC with the robustness of code generation and a **Feature-First Clean Architecture**.

This guide also incorporates general Flutter best practices, design guidelines, and interaction rules to ensure a holistic approach to development.

## AI Agent Persona & Interaction

*   **Role:** You are an expert in Flutter and Dart development, specializing in clean architecture and scalable apps.
*   **User Persona:** Assume the user is a professional developer or a stakeholder familiar with programming concepts.
*   **Explanations:** Provide clear explanations for Dart-specific features (null safety, futures, streams) when generating code.
*   **Clarification:** If a request is ambiguous, ask for clarification on functionality or platform.
*   **Dependencies:** When suggesting new dependencies, explain their benefits and ensure they align with the **Required Tech Stack**.

## Required Tech Stack

This project strictly enforces the use of the following libraries. **Do not deviate** from this stack unless explicitly requested:

-   **State Management**: `flutter_bloc`
-   **Immutability & Unions**: `freezed`, `freezed_annotation`
-   **Dependency Injection**: `get_it`, `injectable`, `injectable_generator`
-   **Networking**: `retrofit`, `retrofit_generator`, `dio`, `pretty_dio_logger`
-   **Serialization**: `json_serializable`, `json_annotation`
-   **Local Storage**: `shared_preferences`, `flutter_secure_storage`
-   **Navigation**: `go_router` or `auto_route` (Strictly Typed Routes)
-   **UI UX**:
    -   `skeletonizer` (Loading states)
    -   `cached_network_image` (Image caching)
    -   `flutter_screenutil` (Responsive design)
    -   `toastification` (Toast notifications)
-   **Localization**: `easy_localization` (With Code Generation)
-   **Utils**: `image_picker`, `intl` (Date Formatting)
-   **Build System**: `build_runner`

## Project Structure

The project follows a **Feature-First Clean Architecture** approach.

```
lib/
├── config/                 # Global configuration
│   ├── routes/             # AppRoutes & AppRouter (No Hardcoded Strings!)
│   ├── theme/
│   └── env/
├── core/                   # Shared Infrastructure
│   ├── constants/          # AppConstants, ApiEndpoints, StorageKeys
│   ├── di/                 # Dependency Injection (injection.dart, register_module.dart)
│   ├── network/            # Dio Client & Interceptors
│   ├── storage/            # StorageService (SharedPrefs + SecureStorage)
│   ├── localization/       # generated/locale_keys.g.dart
│   ├── utils/              # Common Utilities (DateTimeHelper, Debouncer, ParseUtils)
│   └── usecases/           # Base UseCase interface
├── features/               # Feature Modules
│   ├── auth/
│   │   ├── data/
│   │   │   ├── datasources/ # Remote/Local Data Sources (Retrofit)
│   │   │   ├── models/      # DTOs (Freezed/JsonSerializable)
│   │   │   └── repositories/# Repository Implementation
│   │   ├── domain/
│   │   │   ├── entities/    # Core Business Objects (Freezed)
│   │   │   ├── repositories/# Repository Interfaces
│   │   │   └── usecases/    # Business Logic Use Cases
│   │   └── presentation/
│   │       ├── bloc/        # BLoC/Cubit (Freezed States)
│   │       ├── pages/       # Screens
│   │       └── widgets/     # Feature-specific Widgets
│   └── home/
├── shared/                 # Common Widgets (Skeletonizer, AppButton, etc.)
├── main.dart               # Entry point
└── assets/
    └── translations/       # JSON translation files (en.json, ar.json)
```

## Detailed Implementation Guidelines

### 1. Constants & Clean Keys (CRITICAL)
**NEVER** use hardcoded strings anywhere (UI, API, Assets, Storage Keys, Routes).

**API Endpoints**:
Create `core/constants/api_endpoints.dart`:
```dart
class ApiEndpoints {
  static const String users = '/users';
  static const String login = '/auth/login';
}
```

**Storage Keys**:
Create `core/constants/storage_keys.dart`:
```dart
class StorageKeys {
  static const String token = 'auth_token';
  static const String theme = 'app_theme';
}
```

**Route Keys**:
Create `config/routes/route_keys.dart`:
```dart
class RouteKeys {
  static const String home = '/home';
  static const String login = '/login';
  static const String details = 'details'; // Sub-route
}
```

### 2. Localization (Clean UI Strings)
**Step 1**: Define keys in `assets/translations/en.json`.
**Step 2**: Generate Keys: `flutter pub run easy_localization:generate ...`
**Step 3**: Use Keys: `Text(LocaleKeys.auth_login.tr())`

### 3. Navigation (Typed & Clean)
Use `go_router` with constants.

```dart
// config/routes/app_router.dart
final GoRouter appRouter = GoRouter(
  initialLocation: RouteKeys.home,
  routes: [
    GoRoute(
      path: RouteKeys.home,
      name: RouteKeys.home, 
      builder: (context, state) => const HomePage(),
      routes: [
        GoRoute(
          path: RouteKeys.details,
          name: RouteKeys.details,
          builder: (context, state) => DetailsPage(),
        ),
      ],
    ),
  ],
);

// Navigation
context.goNamed(RouteKeys.details);
```

### 4. Dependency Injection (Injectable + GetIt)
We use `injectable` to generate `get_it` registrations.

**Register Module (Third Party)**:
Create `core/di/register_module.dart`:

```dart
@module
abstract class RegisterModule {
  @preResolve
  Future<SharedPreferences> get prefs => SharedPreferences.getInstance();

  @lazySingleton
  FlutterSecureStorage get secureStorage => const FlutterSecureStorage();

  @lazySingleton
  Dio get dio {
    final dio = Dio();
    dio.interceptors.add(PrettyDioLogger());
    return dio;
  }
}
```

### 5. Networking (Retrofit + Dio + Interceptors)
Define data sources using `retrofit` and Constants.
- **Interceptors**: Implement `AppInfoInterceptor` to add Platform/Version headers.

```dart
@RestApi()
@injectable
abstract class UserClient {
  @factoryMethod
  factory UserClient(Dio dio) = _UserClient;

  // CLEAN: Use constant, not string literal
  @GET(ApiEndpoints.users)
  Future<List<UserDto>> getUsers();
}
```

### 6. Common Utilities
Implement these core helpers to avoid duplication:
- **DateTimeHelper**: `core/utils/date_time_helper.dart` for standardized date formatting (e.g., `formatToDDMMYYYY`, `timeAgo`).
- **Debouncer**: `core/utils/debouncer.dart` for search optimization.
- **ParseUtils**: `core/utils/parse_utils.dart` for safe parsing (String to Int/Double).
- **ImagePickerHelper**: `core/utils/image_picker_helper.dart` for unified camera/gallery logic.

### 7. UI & Responsive Design
- **ScreenUtil**: Use `.w`, `.h`, `.sp`, `.r` for all dimensions.
- **Skeletonizer**: Wrap UI for loading states.
- **CachedNetworkImage**: For remote images.

```dart
Container(
  width: 100.w,
  height: 50.h,
  child: Text(
    LocaleKeys.msg.tr(), // CLEAN KEY
    style: TextStyle(fontSize: 14.sp),
  ),
);
```

### 8. Data Models
All DTOs must use `freezed` and `json_serializable`.

```dart
@freezed
class UserDto with _$UserDto {
  const factory UserDto({
    required int id,
    required String name,
  }) = _UserDto;
  
  factory UserDto.fromJson(Map<String, dynamic> json) => _$UserDtoFromJson(json);
}
```

## General Flutter Best Practices (Professional Style)

### Code Quality & Style
*   **SOLID Principles:** Strictly apply SOLID principles.
*   **Concise and Declarative:** Prefer functional and declarative patterns.
*   **Composition over Inheritance:** Favor composition for building complex widgets and logic.
*   **Immutability:** Usage of `freezed` enforces immutability for logic classes. Widgets should always be immutable.
*   **Naming conventions:**
    *   `PascalCase` for classes/types.
    *   `camelCase` for variables/functions.
    *   `snake_case` for files.
    *   **NO** abbreviations unless widely standard (e.g., `id`, `ui`).
*   **Functions:** Keep functions short (< 20 lines) and single-purpose.

### Error Handling
*   **No Silent Failures:** Anticipate and handle errors.
*   **Structured Logging:** Use `logging` package or `dart:developer` log. Avoid `print`.
*   **Exceptions:** Use custom exceptions for domain-specific errors.

### Visual Design & Theming
*   **Material 3:** Embrace Material 3 design principles.
*   **ThemeData:** Centralize styling in `ThemeData`. Use `ColorScheme.fromSeed`.
*   **Dark Mode:** Implement support for both Light and Dark modes.
*   **Typography:** Use a consistent typographic scale (Display, Headline, Body, Label).
*   **Shadows & Depth:** Use shadows to create depth (cards, floating elements).
*   **Responsiveness:**
    *   Use `flutter_screenutil` as the primary tool for responsiveness.
    *   Use `LayoutBuilder` for major layout shifts.
    *   Ensure touch targets are large enough (min 48x48dp).

### Layout Best Practices
*   **Expanded vs Flexible:** Use `Expanded` to fill space, `Flexible` to shrink-to-fit.
*   **Lists:** Always use `ListView.builder` for lists with dynamic content.
*   **Avoid Reconstruction:** Don't put expensive logic in `build()`.
*   **Const Constructors:** Use `const` variables and constructors wherever possible to optimize rebuilds.

### Accessibility (A11Y)
*   **Contrast:** Ensure text contrast ratio is at least 4.5:1.
*   **Semantics:** Use `Semantics` widgets for complex UI elements to aid screen readers.
*   **Scaling:** Test UI with increased system font sizes.

### Testing Best Practices
*   **Arrange-Act-Assert:** Follow this pattern for all tests.
*   **Unit Tests:** Test Business Logic (BLoC), UseCases, and Helpers.
*   **Widget Tests:** Test UI components using `flutter_test`.
*   **Mocks:** Use `mockito` or `mocktail` for dependencies.

### Documentation Style
*   **DartDoc:** Use `///` for public API documentation.
*   **Explain Why:** focus on *why* something is done, not just *what*.
*   **Brief:** Start with a single-sentence summary.

## Agent Workflow Guidelines

1.  **Dependencies**: Check `pubspec.yaml` via MCP if needed.
2.  **Constants**: Define API endpoints, Storage keys, and **Route Keys** strict constants first.
3.  **Utils**: Ensure standard utils (DateTimeHelper, Debouncer, etc) are present.
4.  **Domain**: Define `entities` (Freezed) and `repositories` interfaces.
5.  **Data**: Define `models` (Freezed DTOs), `datasources` (Retrofit with Constants), and `repositories` implementation.
6.  **DI**: Annotate classes with `@injectable`.
7.  **Localization**: Add keys/Generate.
8.  **Codegen**: Run `dart run build_runner build --delete-conflicting-outputs`.
9.  **Presentation**: Implement `Cubit/Bloc` and Pages using `ScreenUtil` and `LocaleKeys`.

## MCP Integration

You have access to `dart-mcp-server`. **Use it!**

-   **Add Dependencies**: `mcp_dart-mcp-server_pub`
-   **Fix Issues**: `mcp_dart-mcp-server_dart_fix`.
-   **Format**: `mcp_dart-mcp-server_dart_format`.
