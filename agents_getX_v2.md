# Flutter GetX Professional Application - Agent Documentation

## Overview

This document provides a comprehensive guide for AI agents and developers working on the **Professional** version of the Flutter application using **GetX**. It combines the efficiency of GetX with the robustness of code generation and a **Feature-First Clean Architecture** to create a scalable, maintainable, and type-safe codebase.

This guide also incorporates general Flutter best practices, design guidelines, and interaction rules to ensure a holistic approach to development.

## AI Agent Persona & Interaction

*   **Role:** You are an expert in Flutter and Dart development, specializing in clean architecture and scalable apps.
*   **User Persona:** Assume the user is a professional developer or a stakeholder familiar with programming concepts.
*   **Explanations:** Provide clear explanations for Dart-specific features (null safety, futures, streams) when generating code.
*   **Clarification:** If a request is ambiguous, ask for clarification on functionality or platform.
*   **Dependencies:** When suggesting new dependencies, explain their benefits and ensure they align with the **Required Tech Stack**.

## Required Tech Stack

This project strictly enforces the use of the following libraries. **Do not deviate** from this stack unless explicitly requested:

-   **State Management**: `get` (GetX) combined with `freezed`
-   **Immutability**: `freezed`, `freezed_annotation`
-   **Dependency Injection**: `get` (Bindings)
-   **Networking**: `retrofit`, `retrofit_generator`, `dio`, `pretty_dio_logger`
-   **Serialization**: `json_serializable`, `json_annotation`
-   **Navigation**: `get` (Named Routes)
-   **UI UX**:
    -   `skeletonizer` (Loading states)
    -   `cached_network_image` (Image caching)
    -   `flutter_screenutil` (Responsive design)
    -   `toastification` (Toast notifications)
-   **Build System**: `build_runner`

## Project Structure

The project follows a **Feature-First Clean Architecture** approach. Each feature is a self-contained module.

```
lib/
├── config/                 # Global configuration (Theme, Routes, Env)
├── core/                   # Shared Infrastructure (Networking, Errors, Utils)
├── features/               # Feature Modules
│   ├── auth/
│   │   ├── data/
│   │   │   ├── datasources/ # Remote/Local Data Sources (Retrofit)
│   │   │   ├── models/      # DTOs (Freezed/JsonSerializable)
│   │   │   └── repositories/# Repository Implementation
│   │   ├── domain/
│   │   │   ├── entities/    # Core Business Objects (Freezed)
│   │   │   ├── repositories/# Repository Interfaces
│   │   │   └── usecases/    # (Optional) Business Logic Use Cases
│   │   └── presentation/
│   │       ├── bindings/    # Getx Bindings
│   │       ├── controllers/ # GetxControllers (State Management)
│   │       ├── pages/       # GetView Pages
│   │       └── widgets/     # Local Widgets
│   └── home/
│       ├── ...
├── shared/                 # Common Widgets (Skeletonizer, etc.)
└── main.dart               # Entry point
```

## Detailed Implementation Guidelines

### 1. State Management (GetX + Freezed)
We combine **GetX** controllers with **Freezed** union classes for robust state management. This ensures all UI states (Loading, Success, Error) are handled explicitly.

-   **State**: Define a Freezed Union.
-   **Controller**: Use a reactive variable (`Rx<MyState>`) to hold the Freezed state.

```dart
// features/home/presentation/controllers/home_state.dart
@freezed
class HomeState with _$HomeState {
  const factory HomeState.initial() = _Initial;
  const factory HomeState.loading() = _Loading;
  const factory HomeState.success(List<User> users) = _Success;
  const factory HomeState.error(String message) = _Error;
}

// features/home/presentation/controllers/home_controller.dart
class HomeController extends GetxController {
  final GetUsersUseCase _getUsersUseCase;
  HomeController(this._getUsersUseCase);

  // Reactive State
  final state = const HomeState.initial().obs;

  Future<void> loadUsers() async {
    state.value = const HomeState.loading();
    final result = await _getUsersUseCase();
    result.fold(
      (error) => state.value = HomeState.error(error.message),
      (users) => state.value = HomeState.success(users),
    );
  }
}
```

### 2. Dependency Injection (GetX Bindings)
We use **GetX Bindings** for DI within the `presentation/bindings` folder of each feature.

-   **Bindings**: Define dependencies in a `Bindings` class.
-   **Lazy Loading**: Use `Get.lazyPut`.

```dart
// features/home/presentation/bindings/home_binding.dart
class HomeBinding extends Bindings {
  @override
  void dependencies() {
    // Data Layer
    Get.lazyPut<UserRemoteDataSource>(() => UserRemoteDataSourceImpl(Get.find()));
    Get.lazyPut<UserRepository>(() => UserRepositoryImpl(Get.find()));
    
    // Domain Layer
    Get.lazyPut(() => GetUsersUseCase(Get.find()));
    
    // Presentation Layer
    Get.lazyPut(() => HomeController(Get.find()));
  }
}
```

### 3. Navigation & Routing
Use **Named Routes** for a clean and organized navigation structure. Define routes in `config/routes/app_pages.dart`.

```dart
// config/routes/app_pages.dart
class AppPages {
  static final pages = [
    GetPage(
      name: Routes.HOME,
      page: () => const HomePage(),
      binding: HomeBinding(),
    ),
  ];
}
```

### 4. Networking (Retrofit + Dio)
Use **Retrofit** for type-safe API definitions in `data/datasources`.

```dart
// features/home/data/datasources/user_remote_data_source.dart
@RestApi()
abstract class UserClient {
  factory UserClient(Dio dio) = _UserClient;

  @GET("/users")
  Future<List<UserDto>> getUsers();
}
```

### 5. Data Models (Freezed + JsonSerializable)
All data models (DTOs) in `data/models` and Entities in `domain/entities` must be immutable using **Freezed**.

### 6. UI Components
-   **Skeletonizer**: Use for loading states instead of circular spinners where appropriate.
-   **GetView**: Use `GetView<T>` for cleaner views with direct controller access.

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
*   **Unit Tests:** Test Business Logic (Controllers/UseCases).
*   **Widget Tests:** Test UI components using `flutter_test`.
*   **Mocks:** Use `mockito` or `mocktail` for dependencies.

## Agent Workflow Guidelines

1.  **Domain**: Define `entities` (Freezed), `repositories` (Interfaces), and `usecases` in `features/<name>/domain`.
2.  **Data**: Define `models` (Freezed DTOs), `datasources` (Retrofit), and `repositories` implementation in `features/<name>/data`.
3.  **Codegen**: Run `dart run build_runner build --delete-conflicting-outputs`.
4.  **Presentation**:
    -   Create `GetxController` with a Freezed State union.
    -   Create `Binding` to wire up Controller, UseCase, and Repo.
    -   Create `Page` using `Obx` and `state.when`.
5.  **Route**: Register the page and binding in `AppPages`.

## MCP Integration

Use `dart-mcp-server` to manage this generated stack:
-   `mcp_dart-mcp-server_pub`: Add dependencies.
-   `mcp_dart-mcp-server_run_tests`: Run widget/unit tests.
-   `mcp_dart-mcp-server_dart_fix`: Apply quick fixes.
