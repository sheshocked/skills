---
name: kotlin-coroutines-flow
description: 
category: android
tags: [kotlin-coroutines-flow]
---

## When to Use
Use this skill when building Android apps with Kotlin coroutines, Flow, StateFlow, SharedFlow, or handling structured concurrency and asynchronous operations.

## Core Concepts
- **Structured concurrency**: Coroutines tied to lifecycle scopes (viewModelScope, lifecycleScope)
- **Flow**: Cold streams that emit values sequentially
- **StateFlow**: Hot flow that always has a value (replaces LiveData)
- **SharedFlow**: Hot flow for events (one-shot events like toasts)
- **Dispatchers**: IO (network/disk), Default (CPU-intensive), Main (UI)
- **SupervisorJob**: Prevents child coroutine failure from canceling siblings

## Workflow
1. **Scope selection**: viewModelScope for ViewModel, lifecycleScope for Activity/Fragment
2. **State management**: Use StateFlow<T> for UI state, SharedFlow<OneShotEvent> for events
3. **Error handling**: Use try/catch around suspend functions, runCatching for Result wrapping
4. **Cancellation**: Always check isActive or use ensureActive() in long loops
5. **Flow operators**: map, filter, combine, debounce, distinctUntilChanged, stateIn, shareIn

## Key Patterns
```kotlin
// ViewModel with StateFlow
class MyViewModel : ViewModel() {
    private val _uiState = MutableStateFlow(UiState.Loading)
    val uiState: StateFlow<UiState> = _uiState.asStateFlow()

    fun loadData() = viewModelScope.launch {
        try {
            val data = repository.fetch()
            _uiState.value = UiState.Success(data)
        } catch (e: Exception) {
            _uiState.value = UiState.Error(e.message)
        }
    }
}

// One-shot event with SharedFlow
private val _events = MutableSharedFlow<Event>()
val events = _events.asSharedFlow()

fun showToast(msg: String) = viewModelScope.launch {
    _events.emit(Event.ShowToast(msg))
}

// Collecting in Compose
val state by viewModel.uiState.collectAsStateWithLifecycle()
```

## Flow vs LiveData
| Feature | StateFlow | LiveData |
|---|---|---|
| Null values | Allowed | Allowed |
| Active collection | Always has value | May be null |
| Backpressure | Buffered/conflated | N/A |
| Operators | Rich set | Limited |
| Testing | Turbine library | InstantTaskExecutorRule |

## Pitfalls
- **collectAsState without lifecycle**: Use collectAsStateWithLifecycle() instead
- **Memory leaks**: Cancel scopes properly; don't leak Job references
- **StateFlow replay**: Default replay=1 means new collectors get last value (usually desired)
- **Sharing started**: use.WhileSubscribed(5000) for UI state to auto-stop when not visible
- **Backpressure**: StateFlow conflates, SharedFlow can buffer or drop

## Verification
- Use Turbine library for testing Flow emissions
- Test state transitions with StandardTestDispatcher
- Verify cancellation with runTest { ... advanceUntilIdle() }
- Check for memory leaks with LeakCanary in debug builds