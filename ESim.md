<div align="center">

# Gate2 eSIM SDK

### Complete eSIM Data Plan Solution for Android

[![SDK Version](https://img.shields.io/badge/SDK-1.0.5-blue.svg)](https://central.sonatype.com/search?q=travel.gate2)
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
    implementation("travel.gate2:esim:1.0.5")
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
                .apiKey(BuildConfig.API_KEY)
                .enableLogging(BuildConfig.DEBUG)
                .build()
        )
    }
}
```

> **Note:** The client certificate for mTLS is embedded within the SDK and configured automatically. No certificate management is required.

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
        // Store request.orderId and request.sessionId
        // Call Gate2Esim.resumeAfterPayment(orderId, sessionId) after payment
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
| `enableLogging` | No | Enable debug logging (default: false) |
| `theme` | No | Custom theme configuration |

> **mTLS Security:** The client certificate is embedded within the SDK and configured automatically. No certificate management is required by the host app.

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
    val isTopUp: Boolean,          // Whether this is a top-up purchase
    val iccid: String?             // eSIM ICCID (for top-ups)
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
                // Step 1: Store orderId and sessionId for resume
                pendingPayment = paymentRequest

                // Step 2: Navigate to your payment provider
                navigateToPayment(
                    orderId   = paymentRequest.orderId,
                    sessionId = paymentRequest.sessionId,
                    amount    = paymentRequest.amount,
                    currency  = paymentRequest.currency
                )
            }
        )
    }
}

// After payment completes (in your payment callback):
fun onPaymentComplete(orderId: String, sessionId: String) {
    // Resume the eSIM flow to show confirmation
    Gate2Esim.resumeAfterPayment(
        activity  = this,
        orderId   = orderId,
        sessionId = sessionId,
        callbacks = EsimCallbacks(
            onComplete = { finish() },
            onCancel   = { finish() },
            onFail     = { error -> showError(error) }
        )
    )
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
        .apiKey(BuildConfig.API_KEY)
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

### Complete Example (MainActivity.kt)

This is a complete working implementation from the demo app:

```kotlin
package com.gate2.travel

import android.os.Bundle
import android.widget.Toast
import androidx.activity.ComponentActivity
import com.gate2.sdk.core.Gate2TravelSdk
import com.gate2.sdk.core.config.Gate2SdkConfig
import com.gate2.sdk.esim.EsimCallbacks
import com.gate2.sdk.esim.Gate2Esim
import com.gate2.sdk.esim.api.EsimError
import com.gate2.sdk.esim.api.EsimErrorCode
import java.util.UUID

class MainActivity : ComponentActivity() {

    companion object {
        private const val KEY_SESSION_ID = "session_id"
        private const val KEY_PENDING_ORDER_ID = "pending_order_id"

        // Demo user ID - in production, get this from your auth system
        private const val DEMO_USER_ID = "9900923232922"
    }

    // Session state - survives configuration changes
    private lateinit var sessionId: String
    private var pendingOrderId: String? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // Restore or create session state
        sessionId = savedInstanceState?.getString(KEY_SESSION_ID)
            ?: UUID.randomUUID().toString()
        pendingOrderId = savedInstanceState?.getString(KEY_PENDING_ORDER_ID)

        // Initialize SDK (app-level config only)
        initializeSdk()

        // Launch eSIM flow
        launchEsimFlow()
    }

    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        outState.putString(KEY_SESSION_ID, sessionId)
        pendingOrderId?.let { outState.putString(KEY_PENDING_ORDER_ID, it) }
    }

    /**
     * Initialize SDK with app-level configuration.
     * Note: Client certificate for mTLS is handled internally by the SDK.
     */
    private fun initializeSdk() {
        if (Gate2TravelSdk.isInitialized()) return

        Gate2TravelSdk.initialize(
            Gate2SdkConfig.builder()
                .context(this)
                .apiKey(BuildConfig.API_KEY)
                .enableLogging(true)
                .enableCertificatePinning(true)
                .build()
        )
    }

    /**
     * Launch the eSIM browsing and purchasing flow.
     */
    private fun launchEsimFlow() {
        Gate2Esim.start(
            title = "ABB eSIM",
            activity = this,
            sessionId = sessionId,
            callbacks = createCallbacks(),
            userId = DEMO_USER_ID,
            language = "az"
        )
    }

    /**
     * Create callbacks for the eSIM flow with structured error handling.
     */
    private fun createCallbacks() = EsimCallbacks(
        onComplete = {
            showToast("eSIM order completed! Check your email for activation details.")
            finish()
        },
        onCancel = {
            showToast("eSIM purchase cancelled")
            finish()
        },
        onFail = { error: EsimError ->
            handleError(error)
        },
        onProcessPayment = { paymentRequest ->
            // Store orderId for payment flow
            pendingOrderId = paymentRequest.orderId

            showToast(
                "Order created: ${paymentRequest.orderId}\n" +
                    "Amount: ${paymentRequest.amount} ${paymentRequest.currency}\n" +
                    "Redirecting to payment..."
            )

            // In production:
            // 1. Store orderId and sessionId
            // 2. Navigate to your payment provider (Stripe, PayPal, etc.)
            // 3. After payment completes, call Gate2Esim.resumeAfterPayment()

            // Demo: simulate immediate payment completion
            handlePaymentComplete()
        }
    )

    /**
     * Handle errors with structured error information.
     */
    private fun handleError(error: EsimError) {
        // Log error for analytics
        val logMessage = buildString {
            append("Error: code=${error.code.name}")
            append(", message=${error.message}")
            error.httpStatusCode?.let { append(", httpStatus=$it") }
            append(", retryable=${error.isRetryable}")
        }
        android.util.Log.e("eSIM", logMessage)

        // Handle specific error codes
        when (error.code) {
            EsimErrorCode.NETWORK_ERROR -> {
                if (error.isRetryable) {
                    showToast("Network error. Please check your connection and try again.")
                } else {
                    showToast("Network error: ${error.message}")
                }
            }
            EsimErrorCode.UNAUTHORIZED -> {
                showToast("Authentication failed. Please check your API key.")
            }
            EsimErrorCode.MODULE_ACCESS_DENIED -> {
                showToast("eSIM module not available with your subscription.")
            }
            EsimErrorCode.RATE_LIMITED -> {
                showToast("Too many requests. Please try again later.")
            }
            EsimErrorCode.SERVER_ERROR -> {
                if (error.isRetryable) {
                    showToast("Server error. Please try again.")
                } else {
                    showToast("Server error: ${error.message}")
                }
            }
            else -> {
                showToast("Error: ${error.message}")
            }
        }
    }

    /**
     * Handle payment completion and resume the eSIM flow.
     */
    private fun handlePaymentComplete() {
        val orderId = pendingOrderId ?: return

        Gate2Esim.resumeAfterPayment(
            activity = this,
            orderId = orderId,
            sessionId = sessionId,
            callbacks = EsimCallbacks(
                onComplete = {
                    showToast("eSIM activated! Check your email for details.")
                    clearPaymentState()
                    finish()
                },
                onCancel = {
                    clearPaymentState()
                    finish()
                },
                onFail = { error: EsimError ->
                    val message = when (error.code) {
                        EsimErrorCode.ORDER_CONFIRMATION_FAILED -> "Payment confirmation failed"
                        EsimErrorCode.NOT_FOUND -> "Order not found"
                        else -> "Payment failed"
                    }
                    showToast("$message: ${error.message}")
                    clearPaymentState()
                }
            ),
            language = "az"
        )
    }

    private fun clearPaymentState() {
        pendingOrderId = null
    }

    private fun showToast(message: String) {
        Toast.makeText(this, message, Toast.LENGTH_LONG).show()
    }
}
```

---

## My eSIMs

Display the list of purchased eSIMs with detail view:

```kotlin
Gate2Esim.showMyEsims(
    activity  = this,
    sessionId = sessionId,
    callbacks = EsimCallbacks(
        onComplete = { finish() },
        onCancel   = { finish() },
        onFail     = { error -> showError(error.message) }
    ),
    userId   = "user-123",
    language = "en"
)
```

---

## Device eSIM Check

Check if the device supports eSIM before showing eSIM features:

```kotlin
if (Gate2Esim.isDeviceEsimCapable(context)) {
    // Show eSIM purchasing option
} else {
    // Hide eSIM features or show informational message
}
```

> **Note:** Returns `false` on devices running Android 8.1 or below (API < 28) where `EuiccManager` is not available.

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

### Structured Error API (v1.0.3+)

The SDK provides structured error information via `EsimError`:

```kotlin
import com.gate2.sdk.esim.api.EsimError
import com.gate2.sdk.esim.api.EsimErrorCode

onFail = { error: EsimError ->
    // Programmatic handling based on error code
    when (error.code) {
        EsimErrorCode.NETWORK_ERROR -> {
            if (error.isRetryable) showRetryDialog()
        }
        EsimErrorCode.UNAUTHORIZED -> navigateToLogin()
        EsimErrorCode.RATE_LIMITED -> showToast("Too many requests. Try again later.")
        else -> showError(error.message)
    }

    // Log for analytics
    analytics.logError(
        code = error.code.name,
        message = error.message,
        httpStatus = error.httpStatusCode,
        retryable = error.isRetryable
    )
}
```

### EsimError Properties

| Property | Type | Description |
|----------|------|-------------|
| `code` | `EsimErrorCode` | Error category for programmatic handling |
| `message` | `String` | Human-readable message for display |
| `isRetryable` | `Boolean` | Whether retry logic should be offered |
| `httpStatusCode` | `Int?` | HTTP status code (for HTTP errors) |
| `cause` | `Throwable?` | Underlying exception (for debugging) |

### Error Codes

| Code | Description | Retryable |
|------|-------------|-----------|
| `NETWORK_ERROR` | No internet, timeout | Yes |
| `UNAUTHORIZED` | Invalid API key (401) | No |
| `FORBIDDEN` | Access denied (403) | No |
| `NOT_FOUND` | Resource not found (404) | No |
| `RATE_LIMITED` | Too many requests (429) | Yes |
| `SERVER_ERROR` | Server error (5xx) | Yes |
| `SDK_NOT_INITIALIZED` | SDK not initialized | No |
| `MODULE_ACCESS_DENIED` | eSIM module not in subscription | No |
| `INVALID_SESSION_ID` | Session ID is blank | No |
| `INVALID_ORDER_ID` | Order ID validation failed | No |
| `PAYMENT_FAILED` | Payment processing failed | Depends |
| `ORDER_CONFIRMATION_FAILED` | Order confirmation failed | Depends |

### Backward Compatibility

Existing code using `(String) -> Unit` for `onFail` continues to work:

```kotlin
// Still works! The message is extracted automatically.
onFail = { message: String -> showError(message) }
```

### Common Error Scenarios

| Scenario | Error Code | Recommended Action |
|----------|------------|-------------------|
| No internet | `NETWORK_ERROR` | Show retry option |
| Invalid API key | `UNAUTHORIZED` | Check configuration |
| Plan unavailable | `NOT_FOUND` | Return to search |
| Payment declined | `PAYMENT_FAILED` | Offer retry |
| Server error | `SERVER_ERROR` | Show retry option |

### Best Practice

```kotlin
onFail = { error: EsimError ->
    if (BuildConfig.DEBUG) {
        Log.e("Gate2SDK", "Error: ${error.code.name} - ${error.message}")
    }

    if (error.isRetryable) {
        AlertDialog.Builder(context)
            .setTitle("Error")
            .setMessage(error.message)
            .setPositiveButton("Try Again") { _, _ -> /* retry */ }
            .setNegativeButton("Cancel") { _, _ -> finish() }
            .show()
    } else {
        Toast.makeText(context, error.message, Toast.LENGTH_LONG).show()
    }
}
```

---

## Troubleshooting

### Quick Diagnostic Checklist

- [ ] SDK initialized in `Application.onCreate()`
- [ ] Valid API key configured
- [ ] Internet connection available
- [ ] Using `Gate2TravelTheme` wrapper
- [ ] Compatible Android version (API 27+)
- [ ] All required callbacks implemented

### Debug Logging

Enable SDK logging during development:

```kotlin
Gate2TravelSdk.initialize(
    Gate2SdkConfig.builder()
        .context(this)
        .apiKey(BuildConfig.API_KEY)
        .enableLogging(BuildConfig.DEBUG)  // Enable in debug builds
        .build()
)
```

Log tag: `Gate2-esim`

Key log messages:
| Message | Meaning |
|---------|---------|
| `SDK initialized (v1.0.5)` | Initialization successful |
| `eSIM flow started` | `Gate2Esim.start()` called |
| `Resuming eSIM flow after payment` | `resumeAfterPayment()` called |
| `Confirming order: xxx` | Order confirmation in progress |
| `Order confirmed successfully` | Payment verification complete |
| `Order confirmation failed` | Check error details |

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

#### "Authentication failed"

```kotlin
// Ensure API key is set correctly
Gate2SdkConfig.builder()
    .context(this)
    .apiKey(BuildConfig.API_KEY)
    .build()
```

If authentication still fails, verify your API key is valid and not expired.

---

## License

Commercial License - See https://gate2.travel/license

---

<div align="center">

**Gate2 eSIM SDK** - Complete eSIM Solution for Android

© 2026 Gate2 Travel. All rights reserved.

</div>
