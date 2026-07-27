---
name: retrofit-okhttp-networking
description: 
category: android
tags: [retrofit-okhttp-networking]
---

## When to Use
Use this skill when building HTTP clients with Retrofit/OkHttp: REST APIs, interceptors, auth token refresh, error mapping, and caching.

## Core Concepts
- **Retrofit**: Type-safe HTTP client with annotation-based API definitions
- **OkHttp**: Low-level HTTP client handling connection pooling, retries, caching
- **Interceptors**: Modify requests/responses (auth headers, logging, caching)
- **CallAdapter**: Handle suspend functions, RxJava, or custom return types

## Workflow
1. Define API interface with Retrofit annotations
2. Configure OkHttp client with interceptors
3. Build Retrofit instance with converters
4. Inject and use in Repository layer

## Key Patterns
```kotlin
// API Definition
interface UserApi {
    @GET("users/{id}")
    suspend fun getUser(@Path("id") id: Long): User

    @POST("users")
    suspend fun createUser(@Body user: CreateUserRequest): User

    @GET("users")
    suspend fun searchUsers(@Query("q") query: String): List<User>
}

// Auth Interceptor
class AuthInterceptor(private val tokenProvider: TokenProvider) : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val token = tokenProvider.getToken() ?: return chain.proceed(chain.request())
        val request = chain.request().newBuilder()
            .addHeader("Authorization", "Bearer $token")
            .build()
        return chain.proceed(request)
    }
}

// Token Refresh Interceptor
class AuthRefreshInterceptor(
    private val tokenProvider: TokenProvider,
    private val authService: AuthService
) : Interceptor {
    override fun intercept(chain: Interceptor.Chain): Response {
        val response = chain.proceed(chain.request())
        if (response.code == 401) {
            val newToken = tokenProvider.refreshToken()
            val newRequest = chain.request().newBuilder()
                .header("Authorization", "Bearer $newToken")
                .build()
            return chain.proceed(newRequest)
        }
        return response
    }
}
```

## Pitfalls
- **Missing @Body**: POST/PUT without @Body won't compile
- **Suspend functions**: Retrofit supports suspend directly — no need for Call<T>
- **Thread safety**: Retrofit is thread-safe; don't create new instances per request
- **Error parsing**: Use Response<T> or HttpException for error bodies
- **Timeout**: Set connectTimeout, readTimeout on OkHttp client

## Verification
- Use MockWebServer for integration tests
- Test interceptors with recorded requests
- Verify error handling with different HTTP status codes