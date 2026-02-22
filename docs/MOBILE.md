# Mobile SDK Guide

Complete guide to the mobile SDK, mobile app features, and mobile-specific considerations for ResonantGenesis.

## Overview

ResonantGenesis provides native mobile SDKs for iOS and Android, enabling developers to integrate AI agents into mobile applications. This guide covers SDK installation, configuration, and mobile-specific features.

## Platform Support

### iOS

| Requirement | Version |
|-------------|---------|
| iOS | 14.0+ |
| Swift | 5.5+ |
| Xcode | 14.0+ |

### Android

| Requirement | Version |
|-------------|---------|
| Android | API 24+ (Android 7.0) |
| Kotlin | 1.8+ |
| Java | 11+ |

## Installation

### iOS (Swift Package Manager)

```swift
// Package.swift
dependencies: [
    .package(url: "https://github.com/resonantgenesis/ios-sdk.git", from: "1.0.0")
]
```

### iOS (CocoaPods)

```ruby
# Podfile
pod 'ResonantGenesis', '~> 1.0'
```

### Android (Gradle)

```kotlin
// build.gradle.kts
dependencies {
    implementation("xyz.resonantgenesis:sdk:1.0.0")
}
```

### Android (Maven)

```xml
<dependency>
    <groupId>xyz.resonantgenesis</groupId>
    <artifactId>sdk</artifactId>
    <version>1.0.0</version>
</dependency>
```

## Configuration

### iOS Configuration

```swift
import ResonantGenesis

// Initialize SDK
let config = ResonantConfig(
    apiKey: "your_api_key",
    environment: .production
)

ResonantGenesis.initialize(config: config)
```

### Android Configuration

```kotlin
import xyz.resonantgenesis.sdk.ResonantGenesis
import xyz.resonantgenesis.sdk.ResonantConfig

// Initialize SDK
val config = ResonantConfig.Builder()
    .apiKey("your_api_key")
    .environment(Environment.PRODUCTION)
    .build()

ResonantGenesis.initialize(context, config)
```

## Agent Integration

### iOS - Create Agent

```swift
// Create agent
let agent = try await ResonantGenesis.agents.create(
    name: "Mobile Assistant",
    systemPrompt: "You are a helpful mobile assistant.",
    model: .gpt4Turbo
)

// Execute agent
let result = try await agent.execute(goal: "Help me with my task")
print(result.output)
```

### Android - Create Agent

```kotlin
// Create agent
val agent = ResonantGenesis.agents.create(
    name = "Mobile Assistant",
    systemPrompt = "You are a helpful mobile assistant.",
    model = Model.GPT4_TURBO
)

// Execute agent
val result = agent.execute(goal = "Help me with my task")
println(result.output)
```

## Streaming Responses

### iOS Streaming

```swift
// Stream responses
for try await chunk in agent.executeStream(goal: "Write a story") {
    switch chunk {
    case .text(let text):
        updateUI(text: text)
    case .toolCall(let tool):
        showToolIndicator(tool: tool)
    case .complete(let result):
        showFinalResult(result: result)
    }
}
```

### Android Streaming

```kotlin
// Stream responses
agent.executeStream(goal = "Write a story")
    .collect { chunk ->
        when (chunk) {
            is StreamChunk.Text -> updateUI(chunk.text)
            is StreamChunk.ToolCall -> showToolIndicator(chunk.tool)
            is StreamChunk.Complete -> showFinalResult(chunk.result)
        }
    }
```

## Chat Interface

### iOS Chat View

```swift
import ResonantGenesisUI

struct ChatView: View {
    @StateObject var chatManager = ChatManager()
    
    var body: some View {
        ResonantChatView(
            agent: chatManager.agent,
            theme: .default,
            onMessage: { message in
                // Handle new message
            }
        )
    }
}
```

### Android Chat View

```kotlin
import xyz.resonantgenesis.ui.ResonantChatView

class ChatActivity : AppCompatActivity() {
    private lateinit var chatView: ResonantChatView
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        chatView = ResonantChatView(this).apply {
            setAgent(agent)
            setTheme(ChatTheme.DEFAULT)
            setOnMessageListener { message ->
                // Handle new message
            }
        }
        
        setContentView(chatView)
    }
}
```

## Offline Support

### Caching Configuration

```swift
// iOS - Configure caching
let config = ResonantConfig(
    apiKey: "your_api_key",
    cacheConfig: CacheConfig(
        enabled: true,
        maxSize: 50 * 1024 * 1024, // 50MB
        ttl: 86400 // 24 hours
    )
)
```

```kotlin
// Android - Configure caching
val config = ResonantConfig.Builder()
    .apiKey("your_api_key")
    .cacheConfig(
        CacheConfig(
            enabled = true,
            maxSize = 50 * 1024 * 1024, // 50MB
            ttl = 86400 // 24 hours
        )
    )
    .build()
```

### Offline Queue

```swift
// iOS - Queue requests when offline
ResonantGenesis.offlineQueue.enqueue(
    request: AgentExecuteRequest(
        agentId: agent.id,
        goal: "Process this when online"
    ),
    priority: .high
)

// Process queue when online
ResonantGenesis.offlineQueue.processWhenOnline()
```

## Push Notifications

### iOS Push Setup

```swift
// Register for push notifications
ResonantGenesis.notifications.register(
    deviceToken: deviceToken,
    events: [.sessionComplete, .agentError, .systemAlert]
)

// Handle notification
func userNotificationCenter(
    _ center: UNUserNotificationCenter,
    didReceive response: UNNotificationResponse
) {
    ResonantGenesis.notifications.handle(response: response)
}
```

### Android Push Setup

```kotlin
// Register for push notifications
ResonantGenesis.notifications.register(
    fcmToken = fcmToken,
    events = listOf(
        NotificationEvent.SESSION_COMPLETE,
        NotificationEvent.AGENT_ERROR,
        NotificationEvent.SYSTEM_ALERT
    )
)

// Handle notification
class ResonantMessagingService : FirebaseMessagingService() {
    override fun onMessageReceived(message: RemoteMessage) {
        ResonantGenesis.notifications.handle(message)
    }
}
```

## Voice Integration

### iOS Voice Input

```swift
import Speech

// Configure voice input
let voiceConfig = VoiceConfig(
    language: .english,
    continuous: true,
    interimResults: true
)

// Start voice session
let voiceSession = try await agent.startVoiceSession(config: voiceConfig)

voiceSession.onTranscript { transcript in
    updateTranscriptUI(transcript)
}

voiceSession.onResponse { response in
    speakResponse(response)
}
```

### Android Voice Input

```kotlin
// Configure voice input
val voiceConfig = VoiceConfig(
    language = Language.ENGLISH,
    continuous = true,
    interimResults = true
)

// Start voice session
val voiceSession = agent.startVoiceSession(voiceConfig)

voiceSession.onTranscript { transcript ->
    updateTranscriptUI(transcript)
}

voiceSession.onResponse { response ->
    speakResponse(response)
}
```

## Biometric Authentication

### iOS Biometrics

```swift
import LocalAuthentication

// Configure biometric auth
ResonantGenesis.auth.configureBiometrics(
    enabled: true,
    fallbackToPasscode: true,
    reason: "Authenticate to access your agents"
)

// Authenticate
let authenticated = try await ResonantGenesis.auth.authenticateWithBiometrics()
```

### Android Biometrics

```kotlin
import androidx.biometric.BiometricPrompt

// Configure biometric auth
ResonantGenesis.auth.configureBiometrics(
    enabled = true,
    fallbackToDeviceCredential = true,
    title = "Authenticate",
    subtitle = "Access your agents"
)

// Authenticate
ResonantGenesis.auth.authenticateWithBiometrics(
    activity = this,
    onSuccess = { /* Access granted */ },
    onError = { error -> /* Handle error */ }
)
```

## Background Processing

### iOS Background Tasks

```swift
import BackgroundTasks

// Register background task
BGTaskScheduler.shared.register(
    forTaskWithIdentifier: "xyz.resonantgenesis.sync",
    using: nil
) { task in
    ResonantGenesis.background.handleTask(task as! BGAppRefreshTask)
}

// Schedule background sync
ResonantGenesis.background.scheduleSync(
    interval: 3600, // 1 hour
    requiresNetwork: true
)
```

### Android Background Work

```kotlin
import androidx.work.WorkManager

// Schedule background sync
ResonantGenesis.background.scheduleSync(
    context = this,
    interval = 1, // hours
    constraints = Constraints.Builder()
        .setRequiredNetworkType(NetworkType.CONNECTED)
        .build()
)
```

## Performance Optimization

### Memory Management

```swift
// iOS - Configure memory limits
ResonantGenesis.performance.configure(
    maxMemory: 100 * 1024 * 1024, // 100MB
    cleanupThreshold: 0.8,
    aggressiveCleanup: true
)
```

```kotlin
// Android - Configure memory limits
ResonantGenesis.performance.configure(
    maxMemory = 100 * 1024 * 1024, // 100MB
    cleanupThreshold = 0.8f,
    aggressiveCleanup = true
)
```

### Battery Optimization

```swift
// iOS - Battery-aware configuration
ResonantGenesis.performance.setBatteryMode(
    lowPowerMode: ProcessInfo.processInfo.isLowPowerModeEnabled,
    reducedPolling: true,
    deferNonCritical: true
)
```

```kotlin
// Android - Battery-aware configuration
ResonantGenesis.performance.setBatteryMode(
    lowPowerMode = powerManager.isPowerSaveMode,
    reducedPolling = true,
    deferNonCritical = true
)
```

### Network Optimization

```swift
// iOS - Network configuration
ResonantGenesis.network.configure(
    timeout: 30,
    retryCount: 3,
    compressionEnabled: true,
    adaptiveQuality: true
)
```

## Security

### Secure Storage

```swift
// iOS - Store credentials securely
try ResonantGenesis.security.storeCredential(
    key: "api_key",
    value: apiKey,
    accessibility: .whenUnlockedThisDeviceOnly
)
```

```kotlin
// Android - Store credentials securely
ResonantGenesis.security.storeCredential(
    key = "api_key",
    value = apiKey,
    requireBiometric = true
)
```

### Certificate Pinning

```swift
// iOS - Enable certificate pinning
ResonantGenesis.security.enableCertificatePinning(
    pins: [
        "sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA="
    ]
)
```

```kotlin
// Android - Enable certificate pinning
ResonantGenesis.security.enableCertificatePinning(
    pins = listOf(
        "sha256/AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA="
    )
)
```

## Analytics

### Mobile Analytics

```swift
// iOS - Track events
ResonantGenesis.analytics.track(
    event: "agent_executed",
    properties: [
        "agent_id": agent.id,
        "duration_ms": duration
    ]
)
```

```kotlin
// Android - Track events
ResonantGenesis.analytics.track(
    event = "agent_executed",
    properties = mapOf(
        "agent_id" to agent.id,
        "duration_ms" to duration
    )
)
```

## Error Handling

### iOS Error Handling

```swift
do {
    let result = try await agent.execute(goal: "Task")
} catch ResonantError.networkError(let error) {
    showNetworkError(error)
} catch ResonantError.authenticationError {
    promptReauthentication()
} catch ResonantError.rateLimitExceeded(let retryAfter) {
    scheduleRetry(after: retryAfter)
} catch {
    showGenericError(error)
}
```

### Android Error Handling

```kotlin
try {
    val result = agent.execute(goal = "Task")
} catch (e: ResonantException.NetworkError) {
    showNetworkError(e)
} catch (e: ResonantException.AuthenticationError) {
    promptReauthentication()
} catch (e: ResonantException.RateLimitExceeded) {
    scheduleRetry(e.retryAfter)
} catch (e: Exception) {
    showGenericError(e)
}
```

## Testing

### iOS Testing

```swift
import XCTest
@testable import ResonantGenesis

class AgentTests: XCTestCase {
    func testAgentExecution() async throws {
        // Use mock client
        ResonantGenesis.useMock()
        
        let agent = try await ResonantGenesis.agents.create(
            name: "Test Agent",
            systemPrompt: "Test"
        )
        
        let result = try await agent.execute(goal: "Test")
        XCTAssertNotNil(result.output)
    }
}
```

### Android Testing

```kotlin
import org.junit.Test
import xyz.resonantgenesis.sdk.testing.MockResonantGenesis

class AgentTests {
    @Test
    fun testAgentExecution() = runTest {
        // Use mock client
        ResonantGenesis.useMock()
        
        val agent = ResonantGenesis.agents.create(
            name = "Test Agent",
            systemPrompt = "Test"
        )
        
        val result = agent.execute(goal = "Test")
        assertNotNull(result.output)
    }
}
```

## Best Practices

### For iOS

1. **Use async/await** - Modern concurrency
2. **Handle background states** - Save state properly
3. **Respect low power mode** - Reduce activity
4. **Test on real devices** - Simulators differ
5. **Use Instruments** - Profile performance

### For Android

1. **Use coroutines** - Structured concurrency
2. **Handle configuration changes** - Rotation, etc.
3. **Respect Doze mode** - Battery optimization
4. **Test on multiple devices** - Fragmentation
5. **Use Android Profiler** - Monitor resources

### General

1. **Cache aggressively** - Reduce network calls
2. **Handle offline gracefully** - Queue operations
3. **Minimize battery usage** - Batch requests
4. **Secure credentials** - Use secure storage
5. **Test thoroughly** - Unit and UI tests

## API Reference

### iOS SDK Reference

Full iOS SDK documentation: [docs.resonantgenesis.xyz/ios](https://docs.resonantgenesis.xyz/ios)

### Android SDK Reference

Full Android SDK documentation: [docs.resonantgenesis.xyz/android](https://docs.resonantgenesis.xyz/android)

---

**Need mobile help?** Contact mobile@resonantgenesis.xyz or join our [Discord](https://discord.gg/resonantgenesis).
