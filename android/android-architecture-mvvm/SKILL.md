---
name: android-architecture-mvvm
description: 
category: android
tags: [android-architecture-mvvm]
---

## When to Use
Use this skill when designing layered Android architecture with ViewModels, repositories, use-cases, and unidirectional data flow.

## Architecture Layers
```
UI Layer (Composable/Activity/Fragment)
  ↓ observes StateFlow
ViewModel Layer (state + events)
  ↓ calls
Domain Layer (optional Use-Cases)
  ↓ calls
Data Layer (Repository → Local/Remote sources)
```

## Workflow
1. **UI**: Only observes state, calls ViewModel methods on user actions
2. **ViewModel**: Manages UI state, delegates to repository, handles business logic
3. **Repository**: Single source of truth, coordinates local and remote data
4. **DataSource**: Local (Room/SharedPrefs), Remote (Retrofit)

## Key Patterns
```kotlin
// Repository pattern
class UserRepository(
    private val local: UserDao,
    private val remote: UserApi
) {
    fun getUsers(): Flow<List<User>> = local.getAllUsers()
        .onStart { refreshFromRemote() }
        .flowOn(Dispatchers.IO)

    private suspend fun refreshFromRemote() {
        val response = remote.fetchUsers()
        local.insertAll(response)
    }
}

// Use-Case (optional, for complex business logic)
class GetUsersUseCase(private val repo: UserRepository) {
    operator fun invoke(): Flow<List<User>> = repo.getUsers()
}

// ViewModel
class UsersViewModel(getUsers: GetUsersUseCase) : ViewModel() {
    val users: StateFlow<List<User>> = getUsers()
        .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5000), emptyList())
}
```

## Rules
- UI never accesses Repository directly
- ViewModel never references Android framework classes (no Context, no View)
- Repository decides cache vs network strategy
- Use-Cases are optional — skip if business logic is simple

## Pitfalls
- **God ViewModel**: Too many responsibilities → split into multiple ViewModels
- **Missing error state**: Always handle loading/error/success states
- **Leaking context**: ViewModel outlives Activity — never hold Activity reference
- **Coroutine scope**: Always use viewModelScope for ViewModel work

## Verification
- Unit test each layer independently with fakes
- UI test verifies state transitions
- Integration test verifies repository data flow