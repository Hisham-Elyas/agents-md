# Flutter GetX Professional Application - Agent Documentation

## Overview

This document provides a comprehensive guide for AI agents and developers working on the **Professional** version of the Flutter application using **GetX**. It combines the efficiency of GetX with the robustness of code generation (Freezed, Retrofit, JSON Serializable) and a **Feature-First Clean Architecture** to create a scalable, maintainable, and type-safe codebase.

## Project Structure

The project follows a **Feature-First Clean Architecture** approach. Each feature is a self-contained module containing its own Domain, Data, and Presentation layers.

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

## Tech Stack & Guidelines

### 1. State Management (GetX + Freezed)
We combine **GetX** controllers with **Freezed** union classes for robust state management. This ensures all UI states (Loading, Success, Error) are handled explicitly.

- **State**: Define a Freezed Union.
- **Controller**: Use a reactive variable (`Rx<MyState>`) to hold the Freezed state.

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

- **Bindings**: Define dependencies in a `Bindings` class.
- **Lazy Loading**: Use `Get.lazyPut`.

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
- **Skeletonizer**: Use for loading states instead of circular spinners where appropriate.
- **GetView**: Use `GetView<T>` for cleaner views with direct controller access.

## Agent Workflow Guidelines

1.  **Domain**: Define `entities` (Freezed), `repositories` (Interfaces), and `usecases` in `features/<name>/domain`.
2.  **Data**: Define `models` (Freezed DTOs), `datasources` (Retrofit), and `repositories` implementation in `features/<name>/data`.
3.  **Codegen**: Run `dart run build_runner build --delete-conflicting-outputs`.
4.  **Presentation**:
    - Create `GetxController` with a Freezed State union.
    - Create `Binding` to wire up Controller, UseCase, and Repo.
    - Create `Page` using `Obx` and `state.when`.
5.  **Route**: Register the page and binding in `AppPages`.

## MCP Integration

Use `dart-mcp-server` to manage this generated stack:
- `mcp_dart-mcp-server_pub`: Add dependencies.
- `mcp_dart-mcp-server_run_tests`: Run widget/unit tests.
- `mcp_dart-mcp-server_dart_fix`: Apply quick fixes.
