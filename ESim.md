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
- **Client certificate** (`.p12`) for mTLS — provided by Gate2

## Installation

```kotlin
dependencies {
    implementation("travel.gate2:esim:1.0.2")
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
                .clientCertificate(
                    pkcs12 = resources.openRawResource(R.raw.client_cert),
                    password = BuildConfig.CLIENT_CERT_PASSWORD
                )
                .build()
        )
    }
}
```

Place your `.p12` file in `res/raw/`. Store the password in `BuildConfig` via `local.properties` or CI secrets.

### 2. Launch eSIM flow

```kotlin
import com.gate2.sdk.esim.Gate2Esim
import com.gate2.sdk.esim.EsimCallbacks
import java.util.UUID

val sessionId = UUID.randomUUID().toString()

Gate2Esim.start(
    activity = this,
    sessionId = sessionId,
    callbacks = EsimCallbacks(
        onComplete = { finish() },
        onCancel   = { finish() },
        onFail     = { error -> showError(error) },
        onProcessPayment = { request ->
            // Your app handles payment externally
            pendingOrderId = request.orderId
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
```

### 3. After payment — resume flow

```kotlin
fun onPaymentSuccess() {
    Gate2TravelSdk.notifyPaymentResult(orderId = pendingOrderId!!)

    Gate2Esim.resumeAfterPayment(
        activity = this,
        orderId = pendingOrderId!!,
        callbacks = EsimCallbacks(
            onComplete = { finish() },
            onCancel   = { finish() },
            onFail     = { error -> showError(error) }
        )
    )
}
```

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

## Support

- Docs: [docs.gate2.travel](https://docs.gate2.travel)
- Email: [support@gate2.travel](mailto:support@gate2.travel)
