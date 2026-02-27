---
name: testing-expert
description: Elite KMP testing expertise covering unit tests, presenter testing with Turbine, screenshot tests with Compose Preview Screenshot Testing/Roborazzi/Paparazzi, repository tests, and test architecture. Use when writing tests, designing test strategies, setting up test infrastructure, or debugging test failures. Triggers on test creation, test patterns, fake implementations, or test infrastructure questions.
---

# Testing Expert Skill

## Testing Philosophy

1. **Test behavior, not implementation** - Focus on what, not how
2. **Arrange-Act-Assert** - Clear test structure
3. **No mocking libraries** - Use test doubles that mirror production interfaces (inspired by Now in Android)
4. **Fast and deterministic** - No flaky tests
5. **Test pyramid** - Many unit tests, fewer integration tests

## Why No Mocking?

Following the Now in Android approach, avoid mocking libraries like MockK/Mockito:
- Test doubles that implement real interfaces are more maintainable
- They catch breaking changes at compile time
- They make tests more readable and behavior-focused
- Use real components where feasible (e.g., actual DataStore with temp storage)

## Dependencies

```kotlin
// build.gradle.kts
commonTest.dependencies {
    implementation(kotlin("test"))
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-test:1.10.2")
    implementation("app.cash.turbine:turbine:1.2.0")
    implementation("io.kotest:kotest-assertions-core:5.9.1")
    implementation("com.slack.circuit:circuit-test:0.31.0")
}

androidUnitTest.dependencies {
    // Roborazzi (preferred for JVM screenshot tests)
    implementation("io.github.takahirom.roborazzi:roborazzi:1.51.0")
    implementation("io.github.takahirom.roborazzi:roborazzi-compose:1.51.0")

    // Paparazzi (alternative screenshot testing)
    implementation("app.cash.paparazzi:paparazzi:1.3.5")

    implementation("org.robolectric:robolectric:4.16")
}
```

## Circuit Presenter Testing

### Using presenterTestOf (Recommended)

The simplest way to test presenters using Circuit's built-in testing utilities:

```kotlin
class ProfilePresenterTest {

    @Test
    fun `initial state is loading`() = runTest {
        presenterTestOf(
            presentFunction = {
                profilePresenter(
                    screen = ProfileScreen(userId = "123"),
                    navigator = FakeNavigator(ProfileScreen(userId = "123")),
                    userRepository = FakeUserRepository(),
                )
            },
        ) {
            assertThat(awaitItem()).isInstanceOf<ProfileScreen.State.Loading>()
        }
    }

    @Test
    fun `successful load shows user profile`() = runTest {
        val fakeRepository = FakeUserRepository().apply {
            emitUser(testUser)
        }

        presenterTestOf(
            presentFunction = {
                profilePresenter(
                    screen = ProfileScreen(userId = "123"),
                    navigator = FakeNavigator(ProfileScreen(userId = "123")),
                    userRepository = fakeRepository,
                )
            },
        ) {
            assertThat(awaitItem()).isInstanceOf<ProfileScreen.State.Loading>()

            val successState = awaitItem() as ProfileScreen.State.Success
            assertThat(successState.user.name).isEqualTo(testUser.name)
        }
    }

    @Test
    fun `navigation events are captured`() = runTest {
        val navigator = FakeNavigator(ProfileScreen(userId = "123"))
        val fakeRepository = FakeUserRepository().apply {
            emitUser(testUser)
        }

        presenterTestOf(
            presentFunction = {
                profilePresenter(
                    screen = ProfileScreen(userId = "123"),
                    navigator = navigator,
                    userRepository = fakeRepository,
                )
            },
        ) {
            skipItems(1) // Skip loading

            val state = awaitItem() as ProfileScreen.State.Success
            state.eventSink(ProfileScreen.Event.BackClick)

            // Assert navigation happened
            assertThat(navigator.awaitPop()).isNotNull()
        }
    }
}
```

### Alternative: Manual Presenter Test

```kotlin
class ProfilePresenterTest {

    @Test
    fun `initial state is loading`() = runTest {
        val presenter = createPresenter()

        presenter.test {
            val state = awaitItem()
            assertThat(state).isInstanceOf<ProfileScreen.State.Loading>()
        }
    }
    
    @Test
    fun `successful load shows user profile`() = runTest {
        val fakeRepository = FakeUserRepository()
        val presenter = createPresenter(userRepository = fakeRepository)
        
        presenter.test {
            // Initial loading
            assertThat(awaitItem()).isInstanceOf<ProfileScreen.State.Loading>()
            
            // Emit user from repository
            fakeRepository.emitUser(testUser)
            
            // Verify success state
            val successState = awaitItem() as ProfileScreen.State.Success
            assertThat(successState.user.name).isEqualTo(testUser.name)
            assertThat(successState.user.email).isEqualTo(testUser.email)
        }
    }
    
    @Test
    fun `error state shows retry option`() = runTest {
        val fakeRepository = FakeUserRepository()
        val presenter = createPresenter(userRepository = fakeRepository)
        
        presenter.test {
            skipItems(1) // Skip loading
            
            fakeRepository.emitError(DomainError.Network.NoConnection)
            
            val errorState = awaitItem() as ProfileScreen.State.Error
            assertThat(errorState.message).contains("connection")
            assertThat(errorState.canRetry).isTrue()
        }
    }
    
    @Test
    fun `refresh event triggers repository refresh`() = runTest {
        val fakeRepository = FakeUserRepository()
        val presenter = createPresenter(userRepository = fakeRepository)
        
        presenter.test {
            skipItems(1) // Skip loading
            fakeRepository.emitUser(testUser)
            
            val state = awaitItem() as ProfileScreen.State.Success
            
            // Trigger refresh
            state.eventSink(ProfileScreen.Event.Refresh)
            
            // Verify refresh was called
            assertThat(fakeRepository.refreshCallCount).isEqualTo(1)
        }
    }
    
    @Test
    fun `back navigation pops screen`() = runTest {
        val navigator = FakeNavigator(ProfileScreen(userId = "123"))
        val fakeRepository = FakeUserRepository()
        val presenter = createPresenter(
            navigator = navigator,
            userRepository = fakeRepository,
        )
        
        presenter.test {
            skipItems(1)
            fakeRepository.emitUser(testUser)
            
            val state = awaitItem() as ProfileScreen.State.Success
            state.eventSink(ProfileScreen.Event.BackClick)
            
            // Verify navigation
            assertThat(navigator.awaitPop()).isNotNull()
        }
    }
    
    private fun createPresenter(
        screen: ProfileScreen = ProfileScreen(userId = "123"),
        navigator: Navigator = FakeNavigator(screen),
        userRepository: UserRepository = FakeUserRepository(),
    ): Presenter<ProfileScreen.State> {
        return Presenter { 
            ProfilePresenter(
                screen = screen,
                navigator = navigator,
                userRepository = userRepository,
            )
        }
    }
    
    companion object {
        val testUser = User(
            id = "123",
            name = "Test User",
            email = "test@example.com",
            avatarUrl = null,
            createdAt = Clock.System.now(),
        )
    }
}
```

### Testing State Transitions

```kotlin
@Test
fun `loading to success to refreshing flow`() = runTest {
    val fakeRepository = FakeUserRepository()
    val presenter = createPresenter(userRepository = fakeRepository)
    
    presenter.test {
        // 1. Initial loading
        assertThat(awaitItem()).isInstanceOf<State.Loading>()
        
        // 2. Data loaded
        fakeRepository.emitUser(testUser)
        val success1 = awaitItem() as State.Success
        assertThat(success1.isRefreshing).isFalse()
        
        // 3. Trigger refresh
        success1.eventSink(Event.Refresh)
        
        // 4. Refreshing state
        val refreshing = awaitItem() as State.Success
        assertThat(refreshing.isRefreshing).isTrue()
        
        // 5. Refresh complete
        fakeRepository.emitUser(updatedUser)
        val success2 = awaitItem() as State.Success
        assertThat(success2.isRefreshing).isFalse()
        assertThat(success2.user).isEqualTo(updatedUser)
    }
}
```

## Fake Implementations

### Fake Repository

```kotlin
class FakeUserRepository : UserRepository {
    
    private val userFlow = MutableSharedFlow<Either<DomainError, User>>(replay = 1)
    private val usersFlow = MutableSharedFlow<Either<DomainError, List<User>>>(replay = 1)
    
    var refreshCallCount = 0
        private set
    
    var lastUpdatedUser: User? = null
        private set
    
    // Emit methods for tests
    suspend fun emitUser(user: User) {
        userFlow.emit(user.right())
    }
    
    suspend fun emitError(error: DomainError) {
        userFlow.emit(error.left())
    }
    
    suspend fun emitUsers(users: List<User>) {
        usersFlow.emit(users.right())
    }
    
    // Repository implementation
    override fun observeUser(id: String): Flow<Either<DomainError, User>> = userFlow
    
    override fun observeUsers(): Flow<Either<DomainError, List<User>>> = usersFlow
    
    override suspend fun refreshUser(id: String): Either<DomainError, User> {
        refreshCallCount++
        return userFlow.first()
    }
    
    override suspend fun updateUser(user: User): Either<DomainError, Unit> {
        lastUpdatedUser = user
        return Unit.right()
    }
    
    fun reset() {
        refreshCallCount = 0
        lastUpdatedUser = null
    }
}
```

### Fake Navigator (Circuit)

```kotlin
class FakeNavigator(
    initialScreen: Screen,
) : Navigator {
    
    private val backStack = mutableListOf<Screen>(initialScreen)
    private val goToChannel = Channel<Screen>(Channel.UNLIMITED)
    private val popChannel = Channel<PopResult?>(Channel.UNLIMITED)
    private val resetRootChannel = Channel<Screen>(Channel.UNLIMITED)
    
    override val onRootPop: () -> Unit = {}
    
    override fun goTo(screen: Screen) {
        backStack.add(screen)
        goToChannel.trySend(screen)
    }
    
    override fun pop(result: PopResult?): Screen? {
        popChannel.trySend(result)
        return if (backStack.size > 1) {
            backStack.removeLast()
        } else null
    }
    
    override fun resetRoot(
        newRoot: Screen,
        saveState: Boolean,
        restoreState: Boolean,
    ): List<Screen> {
        resetRootChannel.trySend(newRoot)
        val old = backStack.toList()
        backStack.clear()
        backStack.add(newRoot)
        return old
    }
    
    override fun peek(): Screen? = backStack.lastOrNull()
    
    override fun peekBackStack(): ImmutableList<Screen> = backStack.toImmutableList()
    
    // Assertion helpers
    suspend fun awaitGoTo(): Screen = goToChannel.receive()
    
    suspend fun awaitPop(): PopResult? = popChannel.receive()
    
    suspend fun awaitResetRoot(): Screen = resetRootChannel.receive()
    
    fun expectNoEvents() {
        assertThat(goToChannel.tryReceive().isSuccess).isFalse()
        assertThat(popChannel.tryReceive().isSuccess).isFalse()
    }
}
```

### Fake API

```kotlin
class FakeUserApi : UserApi {
    
    private var userResponse: Either<NetworkError, UserResponse>? = null
    private var usersResponse: Either<NetworkError, List<UserResponse>>? = null
    
    var createUserCallCount = 0
        private set
    var lastCreateRequest: CreateUserRequest? = null
        private set
    
    fun setUserResponse(response: Either<NetworkError, UserResponse>) {
        userResponse = response
    }
    
    fun setUsersResponse(response: Either<NetworkError, List<UserResponse>>) {
        usersResponse = response
    }
    
    fun setError(error: NetworkError) {
        userResponse = error.left()
        usersResponse = error.left()
    }
    
    override suspend fun getUser(id: String): Either<NetworkError, UserResponse> =
        userResponse ?: NetworkError.Unknown(Exception("Not configured")).left()
    
    override suspend fun getUsers(
        page: Int,
        size: Int,
    ): Either<NetworkError, PaginatedResponse<UserResponse>> =
        usersResponse?.map { users ->
            PaginatedResponse(
                data = users,
                page = page,
                pageSize = size,
                totalCount = users.size,
                hasMore = false,
            )
        } ?: NetworkError.Unknown(Exception("Not configured")).left()
    
    override suspend fun createUser(
        request: CreateUserRequest,
    ): Either<NetworkError, UserResponse> {
        createUserCallCount++
        lastCreateRequest = request
        return userResponse ?: NetworkError.Unknown(Exception("Not configured")).left()
    }
    
    fun reset() {
        userResponse = null
        usersResponse = null
        createUserCallCount = 0
        lastCreateRequest = null
    }
}
```

## Flow Testing with Turbine

### Basic Flow Test

```kotlin
@Test
fun `repository emits cached then fresh data`() = runTest {
    val repository = UserRepositoryImpl(fakeApi, testDatabase)
    
    // Seed database
    testDatabase.userQueries.upsert(cachedUser.toEntity())
    fakeApi.setUserResponse(freshUser.right())
    
    repository.observeUser("123").test {
        // First emission: cached data
        val cached = awaitItem()
        assertThat(cached.isRight()).isTrue()
        assertThat(cached.getOrNull()?.name).isEqualTo(cachedUser.name)
        
        // Second emission: fresh data
        val fresh = awaitItem()
        assertThat(fresh.isRight()).isTrue()
        assertThat(fresh.getOrNull()?.name).isEqualTo(freshUser.name)
        
        // No more emissions expected
        cancelAndIgnoreRemainingEvents()
    }
}
```

### Testing Flow Transformations

```kotlin
@Test
fun `combined flow emits when any source changes`() = runTest {
    val userFlow = MutableSharedFlow<User>()
    val prefsFlow = MutableSharedFlow<Preferences>()
    
    val combinedFlow = combine(userFlow, prefsFlow) { user, prefs ->
        UserWithPrefs(user, prefs)
    }
    
    combinedFlow.test {
        // Emit both values
        userFlow.emit(testUser)
        prefsFlow.emit(testPrefs)
        
        val combined = awaitItem()
        assertThat(combined.user).isEqualTo(testUser)
        assertThat(combined.prefs).isEqualTo(testPrefs)
        
        // Update user only
        userFlow.emit(updatedUser)
        
        val updated = awaitItem()
        assertThat(updated.user).isEqualTo(updatedUser)
        assertThat(updated.prefs).isEqualTo(testPrefs) // Unchanged
    }
}
```

### Testing Error Scenarios

```kotlin
@Test
fun `flow emits error and recovers`() = runTest {
    val repository = FakeUserRepository()
    
    repository.observeUser("123").test {
        // Success
        repository.emitUser(testUser)
        assertThat(awaitItem().isRight()).isTrue()
        
        // Error
        repository.emitError(DomainError.Network.NoConnection)
        assertThat(awaitItem().isLeft()).isTrue()
        
        // Recovery
        repository.emitUser(testUser)
        assertThat(awaitItem().isRight()).isTrue()
    }
}
```

## Use Case Testing

```kotlin
class GetUserProfileUseCaseTest {
    
    private lateinit var userRepository: FakeUserRepository
    private lateinit var prefsRepository: FakePreferencesRepository
    private lateinit var useCase: GetUserProfileUseCase
    
    @BeforeTest
    fun setup() {
        userRepository = FakeUserRepository()
        prefsRepository = FakePreferencesRepository()
        useCase = GetUserProfileUseCase(userRepository, prefsRepository)
    }
    
    @Test
    fun `returns profile when both repositories succeed`() = runTest {
        userRepository.setUser(testUser)
        prefsRepository.setPreferences(testPrefs)
        
        val result = useCase("123")
        
        assertThat(result.isRight()).isTrue()
        val profile = result.getOrNull()!!
        assertThat(profile.user).isEqualTo(testUser)
        assertThat(profile.preferences).isEqualTo(testPrefs)
    }
    
    @Test
    fun `returns error when user not found`() = runTest {
        userRepository.setError(DomainError.Business.UserNotFound)
        prefsRepository.setPreferences(testPrefs)
        
        val result = useCase("unknown")
        
        assertThat(result.isLeft()).isTrue()
        assertThat(result.leftOrNull()).isEqualTo(DomainError.Business.UserNotFound)
    }
    
    @Test
    fun `short-circuits on first error`() = runTest {
        userRepository.setError(DomainError.Network.NoConnection)
        // Don't configure prefs - should not be called
        
        val result = useCase("123")
        
        assertThat(result.isLeft()).isTrue()
        assertThat(prefsRepository.getPreferencesCallCount).isEqualTo(0)
    }
}
```

## Screenshot Testing Options

This project supports three screenshot testing approaches:

| Tool | Type | Best For |
|------|------|----------|
| **Compose Preview Screenshot Testing** | Official Google tool | Simple preview-based tests, teams new to screenshot testing |
| **Roborazzi** | JVM/Robolectric | Advanced control, parameterized tests, CI consistency |
| **Paparazzi** | JVM layoutlib | Fast iteration, no Robolectric dependency |

---

## Compose Preview Screenshot Testing (Official Google Tool)

Google's official screenshot testing tool that converts `@Preview` composables into automated tests. Ideal for leveraging existing previews.

### Requirements
- AGP 8.5.0+
- Kotlin 1.9.20+ (2.0+ recommended)
- JDK 23 or lower

### Setup

**1. Enable in `gradle.properties`:**
```properties
android.experimental.enableScreenshotTest=true
```

**2. Add to version catalog (`libs.versions.toml`):**
```toml
[versions]
screenshot = "0.0.1-alpha12"

[plugins]
screenshot = { id = "com.android.compose.screenshot", version.ref = "screenshot" }

[libraries]
screenshot-validation-api = { group = "com.android.tools.screenshot", name = "screenshot-validation-api", version.ref = "screenshot" }
```

**3. Configure module `build.gradle.kts`:**
```kotlin
plugins {
    alias(libs.plugins.screenshot)
}

android {
    experimentalProperties["android.experimental.enableScreenshotTest"] = true

    testOptions {
        screenshotTests {
            imageDifferenceThreshold = 0.0001f // 0.01% tolerance
        }
    }
}

dependencies {
    screenshotTestImplementation(libs.screenshot.validation.api)
    screenshotTestImplementation(libs.androidx.ui.tooling)
}
```

### Writing Screenshot Tests

Create test files in `src/screenshotTest/kotlin/`:

```kotlin
package com.example.app

import androidx.compose.runtime.Composable
import androidx.compose.ui.tooling.preview.Preview
import com.android.tools.screenshot.PreviewTest
import com.example.app.ui.theme.AppTheme

// Basic screenshot test
@PreviewTest
@Preview(showBackground = true)
@Composable
fun ProfileCardPreview() {
    AppTheme {
        ProfileCard(
            user = PreviewData.user,
            onClick = {},
        )
    }
}

// Multiple configurations with multi-preview
@PreviewTest
@Preview(name = "Light", showBackground = true)
@Preview(name = "Dark", uiMode = android.content.res.Configuration.UI_MODE_NIGHT_YES)
@Composable
fun ProfileCardThemeVariants() {
    AppTheme {
        ProfileCard(user = PreviewData.user, onClick = {})
    }
}

// Font scale variants
@PreviewTest
@Preview(name = "Normal", fontScale = 1.0f)
@Preview(name = "Large", fontScale = 1.5f)
@Preview(name = "Small", fontScale = 0.85f)
@Composable
fun ProfileCardFontScales() {
    AppTheme {
        ProfileCard(user = PreviewData.user, onClick = {})
    }
}

// Different states
@PreviewTest
@Preview(showBackground = true)
@Composable
fun ProfileScreenLoadingPreview() {
    AppTheme {
        ProfileUi(state = ProfileScreen.State.Loading)
    }
}

@PreviewTest
@Preview(showBackground = true)
@Composable
fun ProfileScreenSuccessPreview() {
    AppTheme {
        ProfileUi(
            state = ProfileScreen.State.Success(
                user = PreviewData.user,
                isRefreshing = false,
                eventSink = {},
            ),
        )
    }
}

@PreviewTest
@Preview(showBackground = true)
@Composable
fun ProfileScreenErrorPreview() {
    AppTheme {
        ProfileUi(
            state = ProfileScreen.State.Error(
                message = "Unable to load profile",
                canRetry = true,
                eventSink = {},
            ),
        )
    }
}
```

### Using Built-in Multi-Preview Annotations

Leverage standard multi-preview annotations for comprehensive coverage:

```kotlin
import androidx.compose.ui.tooling.preview.PreviewScreenSizes
import androidx.compose.ui.tooling.preview.PreviewFontScale
import androidx.compose.ui.tooling.preview.PreviewLightDark
import androidx.compose.ui.tooling.preview.PreviewDynamicColors

// Custom combined annotation
@PreviewTest
@PreviewLightDark
@PreviewFontScale
annotation class StandardScreenshotTests

@StandardScreenshotTests
@Composable
fun ButtonPreview() {
    AppTheme {
        PrimaryButton(text = "Submit", onClick = {})
    }
}
```

### Gradle Commands

```bash
# Generate/update reference screenshots
./gradlew updateDebugScreenshotTest
./gradlew :feature:profile:updateDebugScreenshotTest

# Validate against references (fails on differences)
./gradlew validateDebugScreenshotTest
./gradlew :feature:profile:validateDebugScreenshotTest
```

**Output locations:**
- References: `{module}/src/screenshotTest{Variant}/reference/`
- Reports: `{module}/build/reports/screenshotTest/preview/{variant}/index.html`

### CI Best Practices

```yaml
# GitHub Actions example
jobs:
  screenshot-tests:
    runs-on: ubuntu-latest  # Use consistent OS
    steps:
      - uses: actions/checkout@v4
      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'

      - name: Validate screenshots
        run: ./gradlew validateDebugScreenshotTest

      - name: Upload report on failure
        if: failure()
        uses: actions/upload-artifact@v4
        with:
          name: screenshot-report
          path: '**/build/reports/screenshotTest/'
```

**Important CI considerations:**
- ⚠️ Different OS platforms may produce slightly different screenshots
- Run `updateDebugScreenshotTest` **manually only** when UI intentionally changes
- Run `validateDebugScreenshotTest` on every PR
- Use consistent JDK and OS version in CI
- Flag screenshot changes in PRs for reviewer attention

### Directory Structure

```
module/
├── src/
│   ├── main/kotlin/           # Production code
│   ├── test/kotlin/           # Unit tests
│   └── screenshotTest/        # Screenshot tests
│       └── kotlin/
│           └── com/app/feature/
│               ├── ProfileScreenshotTest.kt
│               └── ComponentScreenshotTests.kt
│       └── reference/         # Generated reference images (commit these)
│           └── debug/
│               └── com.app.feature.ProfileCardPreview_xxx.png
```

### Troubleshooting

| Issue | Solution |
|-------|----------|
| Tests not found | Ensure `@PreviewTest` annotation is present (required since v0.0.1-alpha10) |
| Different results on CI | Lock OS, JDK version; consider using Docker |
| Memory issues | Configure heap size in plugin v0.0.1-alpha12+ |
| Missing dependencies | Add both `screenshot-validation-api` and `ui-tooling` |

---

## Screenshot Testing (Roborazzi - Recommended for Advanced Use)

Roborazzi is preferred for JVM-based screenshot tests as it integrates with Robolectric:

### Setup

```kotlin
// build.gradle.kts
plugins {
    id("io.github.takahirom.roborazzi") version "1.51.0"
}

android {
    testOptions {
        unitTests {
            isIncludeAndroidResources = true
            all { test ->
                test.systemProperty("robolectric.graphicsMode", "NATIVE")
            }
        }
    }
}

dependencies {
    testImplementation("io.github.takahirom.roborazzi:roborazzi:1.51.0")
    testImplementation("io.github.takahirom.roborazzi:roborazzi-compose:1.51.0")
    testImplementation("io.github.takahirom.roborazzi:roborazzi-junit-rule:1.51.0")
    testImplementation("org.robolectric:robolectric:4.16")
}
```

### Basic Roborazzi Test

```kotlin
@RunWith(RobolectricTestRunner::class)
@GraphicsMode(GraphicsMode.Mode.NATIVE)
@Config(sdk = [33])
class ProfileUiRoborazziTest {

    @get:Rule
    val composeTestRule = createComposeRule()

    @Test
    fun profileLoading() {
        composeTestRule.setContent {
            AppTheme {
                ProfileUi(
                    state = ProfileScreen.State.Loading,
                    modifier = Modifier.fillMaxSize(),
                )
            }
        }

        composeTestRule.onRoot().captureRoboImage()
    }

    @Test
    fun profileSuccess() {
        composeTestRule.setContent {
            AppTheme {
                ProfileUi(
                    state = ProfileScreen.State.Success(
                        user = PreviewData.user,
                        isRefreshing = false,
                        eventSink = {},
                    ),
                    modifier = Modifier.fillMaxSize(),
                )
            }
        }

        composeTestRule.onRoot().captureRoboImage()
    }

    @Test
    fun profileError() {
        composeTestRule.setContent {
            AppTheme {
                ProfileUi(
                    state = ProfileScreen.State.Error(
                        message = "Something went wrong",
                        canRetry = true,
                        eventSink = {},
                    ),
                    modifier = Modifier.fillMaxSize(),
                )
            }
        }

        composeTestRule.onRoot().captureRoboImage()
    }
}
```

### Device and Theme Variants

```kotlin
@RunWith(ParameterizedRobolectricTestRunner::class)
@GraphicsMode(GraphicsMode.Mode.NATIVE)
class ProfileUiScreenshotVariantsTest(
    private val device: RobolectricDeviceQualifiers,
    private val darkTheme: Boolean,
) {

    companion object {
        @JvmStatic
        @ParameterizedRobolectricTestRunner.Parameters(name = "{0}, dark={1}")
        fun params() = listOf(
            arrayOf(RobolectricDeviceQualifiers.Pixel5, false),
            arrayOf(RobolectricDeviceQualifiers.Pixel5, true),
            arrayOf(RobolectricDeviceQualifiers.MediumTablet, false),
            arrayOf(RobolectricDeviceQualifiers.MediumTablet, true),
        )
    }

    @get:Rule
    val composeTestRule = createComposeRule()

    @Test
    @Config(qualifiers = "w411dp-h891dp") // Will be overridden by device
    fun profileSuccess() {
        composeTestRule.setContent {
            AppTheme(darkTheme = darkTheme) {
                ProfileUi(
                    state = ProfileScreen.State.Success(
                        user = PreviewData.user,
                        isRefreshing = false,
                        eventSink = {},
                    ),
                    modifier = Modifier.fillMaxSize(),
                )
            }
        }

        composeTestRule.onRoot().captureRoboImage(
            filePath = "screenshots/${device}_dark$darkTheme.png",
        )
    }
}
```

### Gradle Commands

```bash
# Record baseline screenshots
./gradlew recordRoborazzi

# Compare against baseline
./gradlew compareRoborazzi

# Verify screenshots (fails if different)
./gradlew verifyRoborazzi

# Record for specific module
./gradlew :feature:profile:recordRoborazzi
```

## Screenshot Testing (Paparazzi)

### Basic Screenshot Test

```kotlin
class ProfileUiScreenshotTest {
    
    @get:Rule
    val paparazzi = Paparazzi(
        deviceConfig = DeviceConfig.PIXEL_5,
        theme = "android:Theme.Material3.DayNight.NoActionBar",
    )
    
    @Test
    fun profileLoading() {
        paparazzi.snapshot {
            AppTheme {
                ProfileUi(
                    state = ProfileScreen.State.Loading,
                    modifier = Modifier.fillMaxSize(),
                )
            }
        }
    }
    
    @Test
    fun profileSuccess() {
        paparazzi.snapshot {
            AppTheme {
                ProfileUi(
                    state = ProfileScreen.State.Success(
                        user = PreviewData.user,
                        isRefreshing = false,
                        eventSink = {},
                    ),
                    modifier = Modifier.fillMaxSize(),
                )
            }
        }
    }
    
    @Test
    fun profileError() {
        paparazzi.snapshot {
            AppTheme {
                ProfileUi(
                    state = ProfileScreen.State.Error(
                        message = "Something went wrong",
                        canRetry = true,
                        eventSink = {},
                    ),
                    modifier = Modifier.fillMaxSize(),
                )
            }
        }
    }
    
    @Test
    fun profileRefreshing() {
        paparazzi.snapshot {
            AppTheme {
                ProfileUi(
                    state = ProfileScreen.State.Success(
                        user = PreviewData.user,
                        isRefreshing = true,
                        eventSink = {},
                    ),
                    modifier = Modifier.fillMaxSize(),
                )
            }
        }
    }
}
```

### Device Configurations

```kotlin
class ResponsiveLayoutScreenshotTest {
    
    @Test
    fun phone() {
        Paparazzi(deviceConfig = DeviceConfig.PIXEL_5).snapshot {
            AppTheme { ContentToTest() }
        }
    }
    
    @Test
    fun tablet() {
        Paparazzi(
            deviceConfig = DeviceConfig.NEXUS_10.copy(
                orientation = ScreenOrientation.LANDSCAPE,
            ),
        ).snapshot {
            AppTheme { ContentToTest() }
        }
    }
    
    @Test
    fun foldable() {
        Paparazzi(
            deviceConfig = DeviceConfig.PIXEL_FOLD,
        ).snapshot {
            AppTheme { ContentToTest() }
        }
    }
}
```

### Dark Mode Testing

```kotlin
class DarkModeScreenshotTest {
    
    @get:Rule
    val paparazzi = Paparazzi(deviceConfig = DeviceConfig.PIXEL_5)
    
    @Test
    fun lightMode() {
        paparazzi.snapshot {
            AppTheme(darkTheme = false) {
                ProfileUi(state = successState)
            }
        }
    }
    
    @Test
    fun darkMode() {
        paparazzi.snapshot {
            AppTheme(darkTheme = true) {
                ProfileUi(state = successState)
            }
        }
    }
}
```

## Database Testing

```kotlin
class UserDatabaseTest {
    
    private lateinit var database: AppDatabase
    private lateinit var queries: UserQueries
    
    @BeforeTest
    fun setup() {
        val driver = JdbcSqliteDriver(JdbcSqliteDriver.IN_MEMORY)
        AppDatabase.Schema.create(driver)
        database = AppDatabase(driver)
        queries = database.userQueries
    }
    
    @AfterTest
    fun teardown() {
        database.close()
    }
    
    @Test
    fun `insert and retrieve user`() {
        queries.upsert(
            id = "123",
            email = "test@example.com",
            name = "Test User",
            avatar_url = null,
            created_at = 0L,
            updated_at = 0L,
        )
        
        val user = queries.getById("123").executeAsOne()
        
        assertThat(user.id).isEqualTo("123")
        assertThat(user.email).isEqualTo("test@example.com")
    }
    
    @Test
    fun `upsert updates existing user`() {
        queries.upsert("123", "old@example.com", "Old Name", null, 0L, 0L)
        queries.upsert("123", "new@example.com", "New Name", null, 0L, 1L)
        
        val users = queries.getAll().executeAsList()
        
        assertThat(users).hasSize(1)
        assertThat(users.first().email).isEqualTo("new@example.com")
    }
    
    @Test
    fun `observe users emits on changes`() = runTest {
        queries.getAll().asFlow().mapToList().test {
            assertThat(awaitItem()).isEmpty()
            
            queries.upsert("1", "a@test.com", "A", null, 0L, 0L)
            assertThat(awaitItem()).hasSize(1)
            
            queries.upsert("2", "b@test.com", "B", null, 0L, 0L)
            assertThat(awaitItem()).hasSize(2)
            
            queries.deleteById("1")
            assertThat(awaitItem()).hasSize(1)
            
            cancelAndIgnoreRemainingEvents()
        }
    }
}
```

## Test Utilities

### Test Data

```kotlin
// TestFixtures.kt in :core:testing module

object TestData {
    val testUser = User(
        id = "test-user-123",
        name = "Test User",
        email = "test@example.com",
        avatarUrl = null,
        createdAt = Instant.parse("2024-01-01T00:00:00Z"),
        updatedAt = Instant.parse("2024-01-01T00:00:00Z"),
    )
    
    val testUsers = listOf(
        testUser,
        testUser.copy(id = "test-user-456", name = "Another User"),
        testUser.copy(id = "test-user-789", name = "Third User"),
    )
    
    val testPreferences = Preferences(
        theme = Theme.SYSTEM,
        notificationsEnabled = true,
        language = "en",
    )
}

// Builder pattern for complex objects
fun user(
    id: String = "test-123",
    name: String = "Test User",
    email: String = "test@example.com",
    block: UserBuilder.() -> Unit = {},
): User = UserBuilder(id, name, email).apply(block).build()

class UserBuilder(
    var id: String,
    var name: String,
    var email: String,
) {
    var avatarUrl: String? = null
    var createdAt: Instant = Clock.System.now()
    var updatedAt: Instant = Clock.System.now()
    
    fun build() = User(id, name, email, avatarUrl, createdAt, updatedAt)
}
```

### Coroutine Test Extensions

```kotlin
// TestExtensions.kt

fun runTest(block: suspend TestScope.() -> Unit) =
    kotlinx.coroutines.test.runTest(testBody = block)

// For testing with specific dispatcher
fun runTestWithMain(block: suspend TestScope.() -> Unit) = runTest {
    Dispatchers.setMain(StandardTestDispatcher(testScheduler))
    try {
        block()
    } finally {
        Dispatchers.resetMain()
    }
}
```

### Assertion Extensions

```kotlin
// AssertionExtensions.kt

inline fun <reified T> Any?.assertIsInstance(): T {
    assertThat(this).isInstanceOf(T::class.java)
    return this as T
}

fun <L, R> Either<L, R>.assertRight(): R {
    assertThat(this.isRight()).isTrue()
    return this.getOrNull()!!
}

fun <L, R> Either<L, R>.assertLeft(): L {
    assertThat(this.isLeft()).isTrue()
    return this.leftOrNull()!!
}

fun <T> Flow<T>.testCollect(
    scope: CoroutineScope,
): MutableList<T> {
    val results = mutableListOf<T>()
    scope.launch { collect { results.add(it) } }
    return results
}
```

## Test Organization

### Directory Structure

```
module/
├── src/
│   ├── commonMain/kotlin/
│   │   └── com/app/feature/
│   │       ├── ProfileScreen.kt
│   │       ├── ProfilePresenter.kt
│   │       └── ProfileUi.kt
│   ├── commonTest/kotlin/
│   │   └── com/app/feature/
│   │       ├── ProfilePresenterTest.kt
│   │       └── fakes/
│   │           └── FakeUserRepository.kt
│   └── androidUnitTest/kotlin/
│       └── com/app/feature/
│           └── ProfileUiScreenshotTest.kt
```

### Naming Conventions

```kotlin
// Test class: {ClassUnderTest}Test
class ProfilePresenterTest

// Test method: backtick style with description
@Test
fun `initial state is loading`()

@Test
fun `successful load shows user profile`()

@Test
fun `error state shows retry button when retryable`()

// Or: given_when_then style
@Test
fun givenUserExists_whenLoadProfile_thenShowsUser()
```

## Anti-Patterns

❌ **Don't test implementation details**
```kotlin
// WRONG - testing internal state
assertThat(presenter.internalLoadingState).isTrue()

// RIGHT - test observable behavior
presenter.test {
    assertThat(awaitItem()).isInstanceOf<State.Loading>()
}
```

❌ **Don't use real dependencies in unit tests**
```kotlin
// WRONG
val presenter = ProfilePresenter(RealUserRepository())

// RIGHT
val presenter = ProfilePresenter(FakeUserRepository())
```

❌ **Don't ignore flaky tests**
```kotlin
// WRONG - retry until pass
@Test
@FlakyTest
fun flaky_test() { ... }

// RIGHT - fix the root cause or use proper synchronization
@Test
fun deterministic_test() = runTest {
    advanceUntilIdle()
    // assertions
}
```

## References

- Compose Preview Screenshot Testing: https://developer.android.com/studio/preview/compose-screenshot-testing
- Turbine: https://github.com/cashapp/turbine
- Roborazzi: https://github.com/takahirom/roborazzi
- Paparazzi: https://github.com/cashapp/paparazzi
- Kotest: https://kotest.io/
- Circuit Testing: https://slackhq.github.io/circuit/testing/
- Coroutines Testing: https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-test/
- Now in Android Testing: https://github.com/android/nowinandroid
