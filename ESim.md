<div align="center">

# Gate2 eSIM SDK

### Complete eSIM Data Plan Solution for Android

[![SDK Version](https://img.shields.io/badge/SDK-1.0.0-blue.svg)](https://central.sonatype.com/search?q=travel.gate2)
[![Platform](https://img.shields.io/badge/Platform-Android-green.svg)](https://developer.android.com)
[![Min SDK](https://img.shields.io/badge/minSdk-27-orange.svg)](https://developer.android.com)
[![Kotlin](https://img.shields.io/badge/Kotlin-2.2.21-purple.svg)](https://kotlinlang.org)
[![Compose](https://img.shields.io/badge/Jetpack%20Compose-BOM%202025.12-teal.svg)](https://developer.android.com/jetpack/compose)

**Integrate complete eSIM purchasing into your Android app with just 10 lines of code.**

</div>

---

## Overview

The **Gate2 eSIM SDK** is a comprehensive, production-ready Android library that enables host applications to integrate complete eSIM data plan purchasing functionality with minimal effort. The SDK handles the entire purchase flow from plan browsing to eSIM activation.

### Key Capabilities

| Capability | Description |
|:-----------|:------------|
| **Plan Search** | Browse data plans by destination, duration, and data allowance |
| **Real-time Pricing** | Get accurate, up-to-date pricing with plan details |
| **Coverage Maps** | View network coverage for selected destinations |
| **Secure Payments** | PCI-compliant payment collection with flexible integration options |
| **eSIM Activation** | QR code and direct installation support |
| **Custom Theming** | Full brand customization with 20+ color properties |

### Integration at a Glance

```
┌─────────────────────────────────────────────────────────────────┐
│                      YOUR APPLICATION                           │
├─────────────────────────────────────────────────────────────────┤
│  ┌───────────────┐    ┌──────────────────────────────────────┐ │
│  │  Initialize   │───▶│         Gate2EsimScreen              │ │
│  │  (1 method)   │    │  ┌────────────────────────────────┐  │ │
│  └───────────────┘    │  │  Complete Purchase Flow        │  │ │
│                       │  │  • Destination Selection       │  │ │
│  ┌───────────────┐    │  │  • Plan Browsing               │  │ │
│  │   Callbacks   │◀───│  │  • Payment Processing          │  │ │
│  │  • onComplete │    │  │  • eSIM Activation             │  │ │
│  │  • onCancel   │    │  │  • Installation Guide          │  │ │
│  │  • onFail     │    │  └────────────────────────────────┘  │ │
│  │  • onPayment  │    └──────────────────────────────────────┘ │
│  └───────────────┘                                              │
└─────────────────────────────────────────────────────────────────┘
```

### Integration Complexity

| Aspect | Rating | Notes |
|:-------|:------:|:------|
| **Lines of Code** | ~10-20 | Minimal integration required |
| **API Surface** | 2 methods | `initialize()` + `Gate2EsimScreen()` |
| **Configuration** | Simple | Builder pattern with sensible defaults |
| **Theming** | Optional | Works out-of-the-box with defaults |
| **Time to Integrate** | < 1 hour | From setup to first purchase |

### Responsibilities Matrix

| SDK Handles | Host App Handles |
|:------------|:-----------------|
| Plan browsing UI and logic | SDK initialization with API key |
| Real-time pricing display | Implement callback handlers |
| Coverage map display | Optional: Custom theming |
| Payment collection UI | Optional: Payment processing |
| eSIM activation & QR code | Navigation after completion |
| Network security & retries | Error display (message provided) |
| State management | |
| Localization support | |

---

## Quick Start

> **Time to Complete**: 5 minutes

### Step 1: Add Dependencies

```kotlin
// app/build.gradle.kts
dependencies {
    implementation("travel.gate2:core:1.0.0")
    implementation("travel.gate2:esim:1.0.0")
}
```

### Step 2: Initialize SDK

```kotlin
import com.gate2.sdk.core.Gate2TravelSdk
import com.gate2.sdk.core.config.Gate2SdkConfig

class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        Gate2TravelSdk.initialize(
            Gate2SdkConfig.builder()
                .context(this)
                .apiKey("your-api-key-here")
                .sessionId(UUID.randomUUID().toString())
                .userId("user-123")
                .build()
        )
    }
}
```

### Step 3: Add the eSIM Screen

```kotlin
import com.gate2.sdk.esim.Gate2EsimScreen
import com.gate2.sdk.core.ui.theme.Gate2TravelTheme

class EsimActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContent {
            Gate2TravelTheme {
                Gate2EsimScreen(
                    onComplete = {
                        // Purchase successful - navigate to success screen
                        finish()
                    },
                    onCancel = {
                        // User cancelled - return to previous screen
                        finish()
                    },
                    onFail = { errorMessage ->
                        // Error occurred - show error to user
                        Toast.makeText(this, errorMessage, Toast.LENGTH_LONG).show()
                    },
                    onProcessPayment = { paymentRequest ->
                        // Optional: Handle payment in your app
                        processPayment(paymentRequest)
                    }
                )
            }
        }
    }

    private fun processPayment(request: EsimPaymentRequest) {
        // Your payment processing logic
        // Call Gate2TravelSdk.notifyPaymentResult() when done
    }
}
```

### That's It!

Your app now has complete eSIM purchasing functionality.

---

## Purchase Flow

The SDK implements a complete **5-screen purchase flow**:

```
┌──────────────────────────────────────────────────────────────────────────┐
│                           PURCHASE FLOW                                   │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐   │
│  │  DEST   │──▶│  PLANS  │──▶│ DETAILS │──▶│ PAYMENT │──▶│ACTIVATE │   │
│  │         │   │         │   │         │   │         │   │         │   │
│  │ Where?  │   │ Which   │   │ Confirm │   │ Pay &   │   │ Install │   │
│  │         │   │ plan?   │   │ details │   │ confirm │   │ eSIM    │   │
│  └─────────┘   └─────────┘   └─────────┘   └─────────┘   └─────────┘   │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

### Screen Details

#### 1. Destination Selection
- Country or region search
- Popular destinations
- Regional plans (Europe, Asia, etc.)

#### 2. Plan Selection
- Data allowance options (1GB, 3GB, 5GB, Unlimited)
- Validity periods (7, 15, 30 days)
- Network providers
- Speed (4G/5G)

#### 3. Plan Details
- Complete plan information
- Coverage details
- Network partners
- Fair usage policy

#### 4. Payment
- Card information collection
- PCI-compliant processing
- Secure transmission

#### 5. Activation
- QR code for installation
- Manual activation code
- Step-by-step guide
- Direct install option

---

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

---

## API Models

### EsimPaymentRequest

```kotlin
data class EsimPaymentRequest(
    val orderId: String,           // Unique order identifier
    val sessionId: String,         // Session ID from SDK config
    val userId: String,            // User ID from SDK config
    val productId: String,         // Selected eSIM product ID
    val productName: String,       // Product display name
    val amount: Double,            // Payment amount
    val currency: String,          // ISO 4217 currency code (e.g., "USD")
    val dataLimit: String,         // Data allowance description
    val validityDays: Int,         // Plan validity in days
    val coverage: List<String>,    // Countries covered
    val email: String              // User email for confirmation
)
```

---

## External Payment Integration

For apps using external payment providers, use the `onProcessPayment` callback:

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
                pendingPayment = null
                showResumeScreen = false
            },
            onCancel = {
                pendingPayment = null
                showResumeScreen = false
            },
            onFail = { error ->
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

                // Step 2: Navigate to your payment provider
                navigateToPayment(
                    orderId = paymentRequest.orderId,
                    amount = paymentRequest.amount,
                    currency = paymentRequest.currency
                )
            }
        )
    }
}

// After payment completes (in your payment callback):
fun onPaymentComplete(orderId: String) {
    // Notify SDK that payment processing is complete
    Gate2TravelSdk.notifyPaymentResult(orderId)

    // Navigate back to show Gate2EsimResumeScreen
    showResumeScreen = true
}
```

---

## Theming & Branding

### Quick Theming

For simple brand color customization:

```kotlin
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

---

## Supported Destinations

The SDK supports eSIM data plans for **190+ countries and regions**:

| Region | Countries |
|--------|-----------|
| **Europe** | France, Germany, Italy, Spain, UK, Netherlands, Switzerland, etc. |
| **Asia** | Japan, South Korea, Thailand, Singapore, Malaysia, Indonesia, etc. |
| **Americas** | USA, Canada, Mexico, Brazil, Argentina, etc. |
| **Middle East** | UAE, Saudi Arabia, Qatar, Turkey, Israel, etc. |
| **Oceania** | Australia, New Zealand, Fiji, etc. |

---

## Sample Code

### Complete Example

```kotlin
// Application.kt
class MyApp : Application() {
    override fun onCreate() {
        super.onCreate()
        Gate2TravelSdk.initialize(
            Gate2SdkConfig.builder()
                .context(this)
                .apiKey(BuildConfig.GATE2_API_KEY)
                .sessionId(UUID.randomUUID().toString())
                .userId("user-123")
                .email("user@example.com")
                .enableLogging(BuildConfig.DEBUG)
                .build()
        )
    }
}

// EsimActivity.kt
class EsimActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            Gate2TravelTheme {
                Gate2EsimScreen(
                    onComplete = { finish() },
                    onCancel = { finish() },
                    onFail = { error ->
                        Toast.makeText(this, error, Toast.LENGTH_LONG).show()
                    },
                    onProcessPayment = { request ->
                        handlePayment(request)
                    }
                )
            }
        }
    }

    private fun handlePayment(request: EsimPaymentRequest) {
        // Process with your payment provider
        yourPaymentProvider.charge(request.amount, request.currency) { result ->
            when (result) {
                is Success -> Gate2TravelSdk.notifyPaymentResult(request.orderId)
                is Failed -> { /* Handle failure */ }
            }
        }
    }
}
```

---

## System Requirements

| Requirement | Version | Notes |
|-------------|---------|-------|
| **Android minSdk** | 27 | Android 8.1 Oreo |
| **Android compileSdk** | 36 | Android 16 |
| **Kotlin** | 2.2.21 | Required for language features |
| **Java** | 11 | JVM target |
| **Jetpack Compose** | BOM 2025.12.00 | Material3 components |

### Device Compatibility

| Feature | Requirement |
|---------|-------------|
| **Screen Size** | Phone and Tablet optimized |
| **Orientation** | Portrait (primary), Landscape (supported) |
| **Network** | Internet connection required |
| **eSIM Support** | Device must support eSIM for activation |
| **Play Services** | Not required |

---

## Error Handling

### Common Error Scenarios

| Scenario | User Message Example | Recommended Action |
|----------|---------------------|-------------------|
| No internet | "Please check your internet connection" | Show retry option |
| Invalid API key | "Authentication failed" | Check configuration |
| Plan unavailable | "This plan is no longer available" | Return to search |
| Price changed | "The price has changed" | Show updated price |
| Payment declined | "Payment was declined" | Offer retry |
| Server error | "Something went wrong. Please try again" | Show retry option |

### Best Practice

```kotlin
onFail = { errorMessage ->
    if (BuildConfig.DEBUG) {
        Log.e("Gate2SDK", "Error: $errorMessage")
    }

    AlertDialog.Builder(context)
        .setTitle("Error")
        .setMessage(errorMessage)
        .setPositiveButton("Try Again") { _, _ -> /* retry */ }
        .setNegativeButton("Cancel") { _, _ -> finish() }
        .show()
}
```

---

## Troubleshooting

### Quick Diagnostic Checklist

- [ ] SDK initialized in `Application.onCreate()`
- [ ] Valid API key configured
- [ ] `userId` is set in configuration
- [ ] Internet connection available
- [ ] Using `Gate2TravelTheme` wrapper
- [ ] Compatible Android version (API 27+)
- [ ] All required callbacks implemented

### Common Issues

#### "SDK not initialized"

```kotlin
// Ensure initialization before use
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        Gate2TravelSdk.initialize(/*...*/)
    }
}

// Register in AndroidManifest.xml
<application android:name=".MyApplication">
```

#### "userId is required"

```kotlin
// Ensure userId is set in configuration
Gate2SdkConfig.builder()
    .context(this)
    .apiKey("your-api-key")
    .sessionId(UUID.randomUUID().toString())
    .userId("user-123")  // Required for eSIM orders
    .build()
```

---
