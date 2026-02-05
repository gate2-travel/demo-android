# Gate2 eSIM SDK

Android SDK for integrating eSIM purchasing into your app. Browse regions and countries, select data plans, and complete purchases with a seamless user experience.

## Requirements

- Android 8.1+ (API 27+)
- Kotlin 1.9+
- Jetpack Compose

## Installation

Add the dependencies to your app's `build.gradle.kts`:

```kotlin
dependencies {
    implementation("travel.gate2:core:1.0.0")
    implementation("travel.gate2:esim:1.0.0")
}
```

## Quick Start

### 1. Initialize the SDK

Initialize the SDK in your `Application` class or `Activity.onCreate()`:

```kotlin
import com.gate2.sdk.core.Gate2TravelSdk
import com.gate2.sdk.core.config.Gate2SdkConfig

class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        Gate2TravelSdk.initialize(
            Gate2SdkConfig.builder()
                .context(this)
                .apiKey("your-api-key")
                .sessionId(UUID.randomUUID().toString())
                .userId("user-123")
                .email("user@example.com")
                .enableLogging(BuildConfig.DEBUG)
                .build()
        )
    }
}
```

### 2. Launch the eSIM Screen

Add the eSIM screen to your Compose UI:

```kotlin
import com.gate2.sdk.esim.Gate2EsimScreen

@Composable
fun MyScreen() {
    Gate2EsimScreen(
        onComplete = {
            // Order completed successfully
            // User will receive eSIM activation details via email
        },
        onCancel = {
            // User cancelled the flow
        },
        onFail = { errorMessage ->
            // Handle error (including access denied)
            Log.e("eSIM", "Error: $errorMessage")
        }
    )
}
```

## Configuration Options

| Parameter | Required | Description |
|-----------|----------|-------------|
| `context` | Yes | Application or Activity context |
| `apiKey` | Yes | Your Gate2 API key |
| `sessionId` | Yes | Unique session identifier (UUID recommended) |
| `userId` | Yes | Your user's unique identifier |
| `email` | No | User's email for order confirmation |
| `enableLogging` | No | Enable debug logging (default: false) |
| `theme` | No | Custom theme configuration |

## External Payment Integration (ABB)

For apps using external payment providers like ABB, use the `onProcessPayment` callback:

```kotlin
import com.gate2.sdk.esim.Gate2EsimScreen
import com.gate2.sdk.esim.Gate2EsimResumeScreen
import com.gate2.sdk.esim.api.EsimPaymentRequest

@Composable
fun MyEsimScreen() {
    var pendingPayment by remember { mutableStateOf<EsimPaymentRequest?>(null) }
    var showResumeScreen by remember { mutableStateOf(false) }

    if (showResumeScreen && pendingPayment != null) {
        // Step 3: Resume flow after payment
        Gate2EsimResumeScreen(
            orderId = pendingPayment!!.orderId,
            onComplete = {
                // Order confirmed successfully
                pendingPayment = null
                showResumeScreen = false
            },
            onCancel = {
                pendingPayment = null
                showResumeScreen = false
            },
            onFail = { error ->
                // Handle error
                pendingPayment = null
                showResumeScreen = false
            }
        )
    } else {
        Gate2EsimScreen(
            onComplete = { /* Direct flow completed */ },
            onCancel = { /* User cancelled */ },
            onFail = { error -> /* Handle error */ },
            onProcessPayment = { paymentRequest ->
                // Step 1: Store payment request
                pendingPayment = paymentRequest

                // Step 2: Navigate to your payment provider (ABB)
                navigateToAbbPayment(
                    orderId = paymentRequest.orderId,
                    amount = paymentRequest.amount,
                    currency = paymentRequest.currency
                )
            }
        )
    }
}

// After ABB payment completes (in your payment callback):
fun onAbbPaymentComplete(orderId: String) {
    // Notify SDK that payment processing is complete
    Gate2TravelSdk.notifyPaymentResult(orderId)

    // Navigate back to show Gate2EsimResumeScreen
    showResumeScreen = true
}
```

### EsimPaymentRequest Properties

| Property | Type | Description |
|----------|------|-------------|
| `orderId` | String | Unique order identifier |
| `sessionId` | String | Session ID from SDK config |
| `userId` | String | User ID from SDK config |
| `productId` | String | Selected eSIM product ID |
| `productName` | String | Product display name |
| `amount` | Double | Payment amount |
| `currency` | String | Currency code (e.g., "USD") |
| `dataLimit` | String | Data allowance description |
| `validityDays` | Int | Plan validity in days |
| `coverage` | List&lt;String&gt; | Countries covered |
| `email` | String | User email for confirmation |

## Theming

Customize the SDK appearance to match your app:

```kotlin
import com.gate2.sdk.core.config.Gate2Theme
import com.gate2.sdk.core.config.PrimaryColors
import com.gate2.sdk.core.config.SecondaryColors
import com.gate2.sdk.core.config.StatusColors

Gate2TravelSdk.initialize(
    Gate2SdkConfig.builder()
        .context(this)
        .apiKey("your-api-key")
        .sessionId(UUID.randomUUID().toString())
        .userId("user-123")
        .theme(
            Gate2Theme.builder()
                .primaryColors(
                    PrimaryColors.builder()
                        .main(0xFF1A73E8.toInt())      // Primary brand color
                        .onMain(0xFFFFFFFF.toInt())   // Text on primary
                        .build()
                )
                .secondaryColors(
                    SecondaryColors.builder()
                        .main(0xFF34A853.toInt())
                        .build()
                )
                .statusColors(
                    StatusColors.builder()
                        .success(0xFF34A853.toInt())
                        .error(0xFFEA4335.toInt())
                        .warning(0xFFFBBC04.toInt())
                        .build()
                )
                .build()
        )
        .build()
)
```

### Update Theme at Runtime

```kotlin
Gate2TravelSdk.updateTheme(
    Gate2Theme.builder()
        .primaryColors(PrimaryColors.builder().main(0xFF6366F1.toInt()).build())
        .build()
)
```

## Complete Example

```kotlin
class MainActivity : ComponentActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // Initialize SDK
        if (!Gate2TravelSdk.isInitialized()) {
            Gate2TravelSdk.initialize(
                Gate2SdkConfig.builder()
                    .context(this)
                    .apiKey("your-api-key")
                    .sessionId(UUID.randomUUID().toString())
                    .userId("user-123")
                    .email("user@example.com")
                    .enableLogging(true)
                    .build()
            )
        }

        enableEdgeToEdge()

        setContent {
            MaterialTheme {
                Gate2EsimScreen(
                    modifier = Modifier.fillMaxSize(),
                    onComplete = {
                        Toast.makeText(
                            this,
                            "eSIM purchased! Check your email.",
                            Toast.LENGTH_LONG
                        ).show()
                    },
                    onCancel = {
                        finish()
                    },
                    onFail = { error ->
                        Toast.makeText(this, "Error: $error", Toast.LENGTH_LONG).show()
                    }
                )
            }
        }
    }
}
```

## User Flow

```
Browse → Select Plan → Enter Email → Payment → Confirmation
```

1. **Browse** - User browses regions or searches countries
2. **Select** - User selects an eSIM plan and views details
3. **Order** - User enters email and reviews order
4. **Payment** - Payment is processed (internal or external)
5. **Confirmation** - User receives eSIM activation details (QR code + manual codes)

## Error Handling

Common error scenarios:

| Error | Cause | Solution |
|-------|-------|----------|
| "SDK not initialized" | `Gate2TravelSdk.initialize()` not called | Initialize SDK before using screens |
| "Access denied: eSIM module not available" | API key doesn't include eSIM access | Contact Gate2 for eSIM-enabled API key |
| "userId is required" | `userId` not set in config | Add `userId` to `Gate2SdkConfig` |
| "Network error" | No internet connection | Check connectivity and retry |

## ProGuard

If using ProGuard/R8, add these rules:

```proguard
-keep class com.gate2.sdk.** { *; }
-keepclassmembers class com.gate2.sdk.** { *; }
```
