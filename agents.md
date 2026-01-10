# Flutter Application - Agent Documentation

## Overview

This document provides a comprehensive guide for AI agents and developers working on the Incart Flutter application. It outlines the architecture, state management patterns, networking, and key implementation details to ensure consistency and efficiency in development.

## Project Structure

follows a feature-based architecture with a clear separation of concerns (Clean Architecture principles):

```
lib/
├── config/                 # Global constants, theme data, and app styles
├── core/                   # Shared infrastructure
│   ├── caching/            # SharedPreferences services
│   ├── di/                 # Dependency Injection (GetIt) - locator.dart
│   ├── localization/       # easy_localization services
│   ├── services/           # Routing (go_router) and Permissions
│   └── utls/               # Shared utilities (extensions, formatters, etc.)
├── features/               # Vertical feature modules (Auth, Cart, Products, etc.)
│   ├── feature_name/
│   │   ├── data/           # API clients and Repositories
│   │   └── presentation/   # BLoCs/Cubits and Widgets
├── shared/                 # Shared widgets, models, and common modules
├── main.dart              # Entry point
└── providers.dart         # Global MultiBlocProvider setup
```

## State Management

We use **BLoC (Business Logic Component)** for complex logic and **Cubit** for simpler states.

### Guidelines
- **BLoC**: Use for multi-step processes or features with many interrelated events (e.g., `OrderBloc`, `RegisterBloc`).
- **Cubit**: Use for simple data fetching, list displays, or toggling UI states (e.g., `GetHomeCubit`, `SessionCubit`).
- **Equatable**: Always extend `Equatable` for states and events to ensure efficient rebuilding.
- **Lazy Registration**: Registered in `locator.dart` and provided via `MultiBlocProvider` in `providers.dart` or locally in the widget tree.

## Dependency Injection (GetIt)

Registered in `lib/core/di/locator.dart`. Use `sl` (Service Locator) to access instances.

```dart
// Factory for Blocs/Cubits (new instance every time)
sl.registerFactory(() => MyCubit(sl<MyRepo>()));

// Lazy Singleton for Repos/APIs (persists across app life)
sl.registerLazySingleton<MyRepo>(() => MyRepoImpl(sl<MyApi>()));
```

## API & Networking

Uses **Dio** with a custom `DioClient` (`lib/shared/dio_client/dio_client.dart`).

### Interceptors
- **AppInfoInterceptor**: Adds device info, app version, and platform headers to every request.
- **LoggingInterceptor**: Handles request/response logging (dev mode).

### Result Wrapper
All repository methods return a `Result<T>` type to handle success and failure gracefully without throwing exceptions up the stack.

```dart
final result = await repo.getData();
result.fold(
  onSuccess: (data) => emit(Loaded(data)),
  onFailure: (error) => emit(Error(error.message)),
);
```

## Navigation & Routing

Uses **go_router** configured in `lib/core/services/routing/app_routes.dart`. Use context extensions for navigation:
- `context.pushNamed(Routes.productDetails, extra: product)`
- `context.goNamed(Routes.home)`

## UI & Styling

### Responsive Design
Uses **flutter_screenutil**. Always use `.w`, `.h`, `.sp`, and `.r` for sizes:
- `width: 100.w`
- `fontSize: 14.sp`
- `borderRadius: BorderRadius.circular(8.r)`

### Core Constants
- **AppColors**: `lib/config/app_colors.dart`
- **AppTextStyle**: `lib/config/text_styles.dart`
- **Extensions**: `lib/core/utls/utils.dart` (includes `sizeW`, `sizeH` helpers).

### Feedback
- **Toastification**: Used for consistent, beautiful toast messages.
- **Skeletonizer**: Used for shimmering loading states in lists.

## Agent Workflow Guidelines

When implementing a new feature, follow this order:
1.  **Define Model**: Create the data model in the feature's `data` folder.
2.  **API Layer**: Define the `Api` class using `DioClient`.
3.  **Repository**: Create the `Repo` abstract class and `RepoImpl`.
4.  **DI Registration**: Register the API and Repo in `locator.dart`.
5.  **State Management**: Create the BLoC/Cubit and its states.
6.  **UI**: Implement the screen and local widgets using the BLoC.
7.  **Routing**: Add the new route to `app_routes.dart`.

## Common Utilities
- `lib/core/utls/parse_utls.dart`: Safe parsing for dates and numbers.
- `lib/core/utls/debouncer.dart`: For search/input optimizing.
- `lib/core/utls/image_picker_helper.dart`: Unified image selection.
