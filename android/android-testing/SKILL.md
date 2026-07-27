---
name: android-testing
description: 
category: android
tags: [android-testing]
---

## When to Use
Use this skill when setting up Android testing: unit tests, Compose UI tests, instrumentation tests, screenshot tests.

## Test Types
- **Unit tests**: JVM-based, fast, test business logic (JUnit + MockK)
- **Compose UI tests**: Test Compose components in isolation
- **Instrumentation tests**: Run on device/emulator, test UI flows
- **Screenshot tests**: Visual regression testing (Paparazzi/Roborazzi)

## Key Patterns
```kotlin
// Unit test with MockK
class UserRepositoryTest {
    private val api = mockk<UserApi>()
    private val dao = mockk<UserDao>(relaxed = true)
    private val repo = UserRepository(dao, api)

    @Test
    fun `fetch users saves to database`() = runTest {
        coEvery { api.fetchUsers() } returns listOf(user1, user2)
        repo.refreshFromRemote()
        coVerify { dao.insertAll(listOf(user1, user2)) }
    }
}

// Compose UI test
@Composable
fun UserListTest() {
    ComposeTestRule.setContent {
        UserList(users = listOf(user1, user2))
    }
    onNodeWithText("Alice").assertIsDisplayed()
    onNodeWithTag("loading_indicator").assertDoesNotExist()
}

// ViewModel test with Turbine
@Test
fun `viewmodel emits loading then success`() = runTest {
    viewModel.uiState.test {
        viewModel.loadData()
        assertEquals(UiState.Loading, awaitItem())
        assertEquals(UiState.Success(data), awaitItem())
    }
}
```

## Test Structure
```
src/test/          → Unit tests (JVM)
src/androidTest/   → Instrumentation tests
src/testDebug/     → Debug-specific tests
```

## Pitfalls
- **Compose test timing**: Use waitUntil { } for async operations
- **Flaky tests**: Use idling resources or Espresso's IdlingResource
- **Mock overuse**: Prefer fakes over mocks for better test confidence
- **Test doubles**: Use Turbine for Flow testing

## Verification
- All unit tests pass: ./gradlew test
- Instrumentation tests pass: ./gradlew connectedAndroidTest
- Coverage report: ./gradlew jacocoTestReport