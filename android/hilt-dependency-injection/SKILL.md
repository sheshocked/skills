---
name: hilt-dependency-injection
description: 
category: android
tags: [hilt-dependency-injection]
---

## When to Use
Use this skill when setting up dependency injection in Android with Hilt/Dagger: modules, scoping, qualifiers, and testing.

## Core Concepts
- **@HiltAndroidApp**: Application class entry point
- **@AndroidEntryPoint**: Inject into Activity/Fragment/Service
- **@HiltViewModel**: Inject into ViewModel
- **@Module + @InstallIn**: Provide dependencies for a component scope
- **@Singleton, @ViewModelScoped, @ActivityScoped**: Lifecycle scoping

## Workflow
1. Add Hilt plugin to build.gradle
2. Annotate Application with @HiltAndroidApp
3. Create modules for complex dependencies
4. Inject via constructor or @Inject fields
5. Use @Provides or @Binds in modules

## Key Patterns
```kotlin
// Module
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    @Provides @Singleton
    fun provideRetrofit(): Retrofit = Retrofit.Builder()
        .baseUrl("https://api.example.com/")
        .addConverterFactory(GsonConverterFactory.create())
        .build()

    @Provides @Singleton
    fun provideUserApi(retrofit: Retrofit): UserApi =
        retrofit.create(UserApi::class.java)
}

// Binds interface to implementation
@Module
@InstallIn(SingletonComponent::class)
abstract class RepositoryModule {
    @Binds abstract fun bindUserRepository(impl: UserRepositoryImpl): UserRepository
}

// ViewModel injection
@HiltViewModel
class UsersViewModel @Inject constructor(
    private val getUsersUseCase: GetUsersUseCase
) : ViewModel()

// Activity
@AndroidEntryPoint
class MainActivity : ComponentActivity() {
    private val viewModel: UsersViewModel by viewModels()
}
```

## Pitfalls
- **Missing @InstallIn**: Module won't be found if not installed in correct component
- **Circular dependency**: Break with lazy<T> or provider<T>
- **Wrong scope**: @Singleton lives forever, @ViewModelScoped per ViewModel
- **Context injection**: Use @ApplicationContext for Application context
- **Testing**: Use @TestInstallIn to swap modules in tests

## Verification
- Compile-time errors catch most DI issues
- Use Hilt testing APIs: @UninstallModules, @TestInstallIn
- Run with ./gradlew connectedAndroidTest