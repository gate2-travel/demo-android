# Gate2 eSIM SDK

Drop-in eSIM data-plan purchasing flow for Android.

## Requirements

| | Minimum |
|---|---|
| Android minSdk | 24 (7.0) |
| Android compileSdk | 35 |
| Kotlin | 2.2.21 |
| Jetpack Compose | BOM 2025.05.00 |
| Java | 11 |

You will also need:
- **Gate2 API key** — obtain from [gate2.travel](https://gate2.travel)

> **Note:** The client certificate for mTLS is embedded within the SDK. No certificate management is required.

## Installation

```kotlin
dependencies {
    implementation("travel.gate2:esim:1.2.0")
}
```

Repository:

```kotlin
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
        maven { url = uri("https://maven.gate2.travel/releases") }
    }
}
```

The eSIM SDK is a standalone artifact — no separate `core` dependency is needed.

## Integration

### 1. Initialize (once, in Application)

```kotlin
import com.gate2.sdk.core.Gate2TravelSdk
import com.gate2.sdk.core.config.Gate2SdkConfig

class MyApp : Application() {
    override fun onCreate() {
        super.onCreate()

        Gate2TravelSdk.initialize(
            Gate2SdkConfig.builder()
                .context(this)
                .apiKey(BuildConfig.API_KEY)
                .enableLogging(BuildConfig.DEBUG)
                .build()
        )
    }
}
```

The SDK handles mTLS authentication internally - no certificate configuration needed.

### 2. Launch eSIM flow

```kotlin
import com.gate2.sdk.esim.Gate2Esim
import com.gate2.sdk.esim.EsimCallbacks
import com.gate2.sdk.esim.api.EsimError
import com.gate2.sdk.esim.api.EsimErrorCode
import java.util.UUID

val sessionId = UUID.randomUUID().toString()

Gate2Esim.start(
    activity = this,
    sessionId = sessionId,
    callbacks = EsimCallbacks(
        onComplete = { finish() },
        onCancel   = { finish() },
        onFail     = { error: EsimError ->
            // Structured error handling
            handleError(error)
        },
        onProcessPayment = { request ->
            // Store orderId and sessionId for resume after payment
            pendingOrderId = request.orderId
            pendingSessionId = request.sessionId
            navigateToPaymentProvider(
                orderId  = request.orderId,
                amount   = request.amount,
                currency = request.currency
            )
        }
    ),
    userId = "user-123",
    email  = "user@example.com",
    language = "en"
)

private fun handleError(error: EsimError) {
    // Log for analytics
    Log.e("eSIM", "Error: code=${error.code.name}, message=${error.message}")

    // Handle specific error codes
    when (error.code) {
        EsimErrorCode.NETWORK_ERROR -> {
            if (error.isRetryable) showRetryDialog()
        }
        EsimErrorCode.UNAUTHORIZED -> navigateToLogin()
        else -> showError(error.message)
    }
}
```

### 3. After payment — resume flow

```kotlin
fun onPaymentSuccess() {
    // Resume flow with both orderId and sessionId
    Gate2Esim.resumeAfterPayment(
        activity  = this,
        orderId   = pendingOrderId!!,
        sessionId = sessionId,  // Same sessionId from start()
        callbacks = EsimCallbacks(
            onComplete = { finish() },
            onCancel   = { finish() },
            onFail     = { error -> showError(error) }
        )
    )
}
```

**Important:** Store both `orderId` (from `onProcessPayment`) and `sessionId` (from `start()`) when payment is initiated. Both are required for proper order confirmation.

### 4. Customization (optional)

```kotlin
Gate2Esim.start(
    activity = this,
    sessionId = sessionId,
    callbacks = callbacks,
    accentColor = 0xFFE91E63.toInt(),  // ARGB accent color
    fontResId   = R.font.poppins,       // custom font from res/font/
    language    = "en"                   // "en", "ru", "az"
)
```

## ProGuard / R8

Consumer rules are applied automatically. If you encounter issues:

```proguard
-keep class com.gate2.sdk.** { *; }
-keepclassmembers class com.gate2.sdk.** { *; }
```
