<div align="center">

# Gate2 eSIM SDK

### eSIM Data Plan Integration for Android

[![SDK Version](https://img.shields.io/badge/SDK-1.0.6-blue.svg)](https://github.com/gate2-travel/demo-android)
[![Platform](https://img.shields.io/badge/Platform-Android-green.svg)](https://developer.android.com)
[![Min SDK](https://img.shields.io/badge/minSdk-27-orange.svg)](https://developer.android.com)
[![Kotlin](https://img.shields.io/badge/Kotlin-2.2.21-purple.svg)](https://kotlinlang.org)
[![Compose](https://img.shields.io/badge/Jetpack%20Compose-BOM%202025.12-teal.svg)](https://developer.android.com/jetpack/compose)

**Integrate complete eSIM data plan purchasing into your Android app.**

</div>

---

> **Document Version**: 1.0.6
> **Last Updated**: February 25, 2026
> **Compatibility**: Android 8.1+ (API 27+)

---

## Table of Contents

- [1. Overview](#1-overview)
- [2. Quick Start](#2-quick-start)
- [3. Purchase Flow](#3-purchase-flow)
- [4. API Reference](#4-api-reference)
  - [4.1 Gate2Esim Methods](#41-gate2esim-methods)
  - [4.2 EsimCallbacks](#42-esimcallbacks)
  - [4.3 EsimPaymentRequest](#43-esimpaymentrequest)
  - [4.4 EsimError & EsimErrorCode](#44-esimerror--esimerrorcode)
  - [4.5 EsimInstallationResult](#45-esiminstallationresult)
  - [4.6 EsimProductSummary](#46-esimproductsummary)
  - [4.7 EsimException](#47-esimexception)
- [5. Sample Code](#5-sample-code)
- [6. My eSIMs](#6-my-esims)
- [7. Device eSIM Check](#7-device-esim-check)
- [8. Error Handling](#8-error-handling)
- [9. Theming & Branding](#9-theming--branding)
- [10. System Requirements](#10-system-requirements)
- [11. Troubleshooting](#11-troubleshooting)
- [12. Support](#12-support)
- [Changelog](#changelog)

---

## 1. Overview

### What is Gate2 eSIM SDK?

The **Gate2 eSIM SDK** (v1.0.6) is a production-ready Android library that enables host applications to integrate complete eSIM data plan purchasing with minimal effort. The SDK handles the entire flow from destination browsing to eSIM activation, plus a "My eSIMs" management screen.

### Key Capabilities

| Capability | Description |
|:-----------|:------------|
| **Plan Search** | Browse data plans by destination, duration, and data allowance |
| **Real-time Pricing** | Accurate, up-to-date pricing with plan details |
| **External Payments** | Host app processes payment, then resumes the SDK flow |
| **eSIM Activation** | QR code and direct device installation (Android 9+) |
| **My eSIMs** | View and manage previously purchased eSIMs |
| **Top-Up** | Add data to existing eSIMs |
| **Custom Theming** | Accent color and custom font support |
| **Localization** | English, Azerbaijani, Russian |

### Integration at a Glance

```
┌─────────────────────────────────────────────────────────────────┐
│                      YOUR APPLICATION                           │
├─────────────────────────────────────────────────────────────────┤
│  ┌───────────────┐    ┌──────────────────────────────────────┐ │
│  │  Initialize   │───▶│         Gate2Esim.start()            │ │
│  │  (1 method)   │    │  ┌────────────────────────────────┐  │ │
│  └───────────────┘    │  │  Complete Purchase Flow        │  │ │
│                       │  │  • Destination Selection       │  │ │
│  ┌───────────────┐    │  │  • Plan Browsing               │  │ │
│  │  EsimCallbacks│◀───│  │  • Payment Processing          │  │ │
│  │  • onComplete │    │  │  • eSIM Activation             │  │ │
│  │  • onCancel   │    │  │  • Installation Guide          │  │ │
│  │  • onFail     │    │  └────────────────────────────────┘  │ │
│  │  • onPayment  │    └──────────────────────────────────────┘ │
│  │  • onInstalled│                                              │
│  └───────────────┘                                              │
└─────────────────────────────────────────────────────────────────┘
```

### API Surface

| Method | Description |
|:-------|:------------|
| `Gate2Esim.start()` | Launch the eSIM purchase flow |
| `Gate2Esim.resumeAfterPayment()` | Resume after external payment |
| `Gate2Esim.showMyEsims()` | Show purchased eSIMs list |
| `Gate2Esim.isAvailable()` | Check if eSIM module is accessible |
| `Gate2Esim.isDeviceEsimCapable()` | Check device eSIM hardware support |

### Responsibilities Matrix

| SDK Handles | Host App Handles |
|:------------|:-----------------|
| Plan browsing UI and logic | SDK initialization with API key |
| Real-time pricing display | Implement `EsimCallbacks` (5 callbacks) |
| eSIM activation & QR code | External payment processing |
| My eSIMs management | Store `orderId` + `sessionId` during payment |
| Network security (mTLS + cert pinning) | Call `resumeAfterPayment()` after payment |
| State management | Navigation after completion |
| Localization (en, az, ru) | Optional: Custom accent color and font |

---

## 2. Quick Start

> **Time to Complete**: 5 minutes

### Step 1: Add Dependency

```kotlin
// app/build.gradle.kts
dependencies {
    implementation("travel.gate2:esim:1.0.6")
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
                .build()
        )
    }
}
```

> **Note**: The eSIM SDK includes an embedded mTLS client certificate for secure API communication. No additional certificate configuration is needed.

### Step 3: Launch the eSIM Flow

```kotlin
import com.gate2.sdk.esim.Gate2Esim
import com.gate2.sdk.esim.EsimCallbacks
import com.gate2.sdk.esim.api.EsimPaymentRequest

class EsimActivity : ComponentActivity() {

    // Store these during onProcessPayment for resumeAfterPayment()
    private var pendingOrderId: String? = null
    private var pendingSessionId: String? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        Gate2Esim.start(
            activity = this,
            sessionId = "your-session-id",
            callbacks = EsimCallbacks(
                onComplete = {
                    // Purchase successful
                    finish()
                },
                onCancel = {
                    // User cancelled
                    finish()
                },
                onFail = { error ->
                    // Structured error — see Section 8
                    showError(error.message)
                },
                onProcessPayment = { paymentRequest ->
                    // Store for resumeAfterPayment()
                    pendingOrderId = paymentRequest.orderId
                    pendingSessionId = paymentRequest.sessionId
                    processPayment(paymentRequest)
                },
                onEsimInstalled = { result ->
                    // Optional: eSIM installation result
                    if (result.success) {
                        Log.d("eSIM", "Installed for order ${result.orderId}")
                    }
                }
            ),
            userId = "user-123",     // Optional
            language = "en"          // Default "en"
        )
    }

    private fun processPayment(request: EsimPaymentRequest) {
        // Process with your payment provider, then:
        Gate2Esim.resumeAfterPayment(
            activity = this,
            orderId = pendingOrderId!!,
            sessionId = pendingSessionId!!,
            callbacks = callbacks // same callbacks
        )
    }
}
```

### That's It!

Your app now has complete eSIM purchasing functionality.

---

## 3. Purchase Flow

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
- SDK calls `onProcessPayment` with `EsimPaymentRequest`
- Host app processes payment externally
- Host app calls `Gate2Esim.resumeAfterPayment()` to continue

#### 5. Activation
- QR code for installation
- Direct install option (Android 9+)
- Step-by-step guide

---

## 4. API Reference

### 4.1 Gate2Esim Methods

#### `start()`

Launch the eSIM browsing and purchasing flow.

```kotlin
fun start(
    activity: Activity,
    sessionId: String,                          // Required — backend correlation
    callbacks: EsimCallbacks,                   // Required — flow outcome handlers
    userId: String? = null,                     // Optional — order tracking
    language: String = "en",                    // Optional — UI language
    accentColor: Int = 0xFF1B63ED.toInt(),      // Optional — brand color (ARGB)
    fontResId: Int = 0,                         // Optional — @FontRes (0 = default Inter)
    title: String = "eSIM"                      // Optional — toolbar title
)
```

**Throws**: Calls `onFail` if SDK is not initialized, session ID is blank, or module access is denied.

---

#### `resumeAfterPayment()`

Resume the eSIM flow after external payment processing.

```kotlin
fun resumeAfterPayment(
    activity: Activity,
    orderId: String,                            // Required — from EsimPaymentRequest.orderId
    sessionId: String,                          // Required — from EsimPaymentRequest.sessionId
    callbacks: EsimCallbacks,                   // Required — flow outcome handlers
    language: String = "en"                     // Optional — UI language
)
```

**Throws**: Calls `onFail` if SDK is not initialized, orderId/sessionId is blank or invalid, or module access is denied.

**Validation**: `orderId` must be alphanumeric with hyphens/underscores, max 100 characters.

---

#### `showMyEsims()`

Show the My eSIMs management screen.

```kotlin
fun showMyEsims(
    activity: Activity,
    sessionId: String,                          // Required
    callbacks: EsimCallbacks,                   // Required
    userId: String,                             // Required
    language: String = "en",                    // Optional
    accentColor: Int = 0xFF1B63ED.toInt(),      // Optional
    fontResId: Int = 0,                         // Optional
    title: String = "eSIM"                      // Optional
)
```

---

#### `isAvailable()`

Check if the eSIM module is available (SDK initialized + token grants access).

```kotlin
fun isAvailable(): Boolean
```

---

#### `isDeviceEsimCapable()`

Check if the device supports eSIM hardware.

```kotlin
fun isDeviceEsimCapable(context: Context): Boolean
```

Returns `false` on Android 8.1 and below (API < 28).

---

### 4.2 EsimCallbacks

```kotlin
data class EsimCallbacks(
    val onComplete: () -> Unit,                             // Purchase completed successfully
    val onCancel: () -> Unit,                               // User cancelled the flow
    val onFail: (EsimError) -> Unit,                        // Error occurred (structured)
    val onProcessPayment: (EsimPaymentRequest) -> Unit,     // Payment processing required
    val onEsimInstalled: (EsimInstallationResult) -> Unit = {} // eSIM installation result (optional)
)
```

**Backward compatibility**: Existing code using `(String) -> Unit` for `onFail` still works:

```kotlin
// String-based error handling (backward compatible)
EsimCallbacks(
    onComplete = { /* ... */ },
    onCancel = { /* ... */ },
    onFail = { message: String -> showError(message) },
    onProcessPayment = { request -> handlePayment(request) }
)
```

Or use the factory method:

```kotlin
EsimCallbacks.withStringError(
    onComplete = { /* ... */ },
    onCancel = { /* ... */ },
    onFailMessage = { message -> showError(message) },
    onProcessPayment = { request -> handlePayment(request) }
)
```

---

### 4.3 EsimPaymentRequest

Received via `onProcessPayment` when the user confirms a plan purchase.

```kotlin
data class EsimPaymentRequest(
    val orderId: String,          // Unique order ID — STORE for resumeAfterPayment()
    val sessionId: String,        // Session ID — STORE for resumeAfterPayment()
    val userId: String,           // User ID from SDK configuration
    val productId: String,        // The eSIM product ID
    val productName: String,      // Name of the eSIM product
    val amount: Double,           // Total amount to charge
    val currency: String,         // ISO 4217 currency code (e.g., "USD", "EUR")
    val dataLimit: String,        // Data allowance (e.g., "5GB", "Unlimited")
    val validityDays: Int,        // Validity period in days
    val coverage: List<String>,   // List of covered countries/regions
    val isTopUp: Boolean,         // Whether this is a top-up order
    val iccid: String?            // ICCID for top-up orders (null for new purchases)
)
```

> **Important**: You **must** store both `orderId` and `sessionId` from this request. They are required parameters when calling `resumeAfterPayment()`.

---

### 4.4 EsimError & EsimErrorCode

#### EsimError

```kotlin
data class EsimError(
    val code: EsimErrorCode,      // Error category for programmatic handling
    val message: String,          // Human-readable message safe to display
    val cause: Throwable?,        // Underlying exception (optional, for debugging)
    val isRetryable: Boolean,     // Whether retry might resolve the error
    val httpStatusCode: Int?      // HTTP status code for HTTP errors (optional)
)
```

#### EsimErrorCode

| Code | Category | Description |
|------|----------|-------------|
| `SDK_NOT_INITIALIZED` | SDK State | Call `Gate2TravelSdk.initialize()` first |
| `MODULE_ACCESS_DENIED` | SDK State | Token doesn't grant eSIM permission |
| `DEVICE_NOT_SUPPORTED` | SDK State | No EuiccManager or not enabled |
| `INVALID_SESSION_ID` | Validation | Session ID is blank or invalid |
| `INVALID_ORDER_ID` | Validation | Order ID is blank, too long, or invalid chars |
| `INVALID_USER_ID` | Validation | User ID required but not provided |
| `VALIDATION_ERROR` | Validation | General parameter validation error |
| `NETWORK_ERROR` | Network | No internet, timeout (usually retryable) |
| `UNAUTHORIZED` | HTTP | 401 — check API key |
| `FORBIDDEN` | HTTP | 403 — access denied |
| `NOT_FOUND` | HTTP | 404 — e.g., invalid order ID |
| `RATE_LIMITED` | HTTP | 429 — try again later |
| `SERVER_ERROR` | HTTP | 5xx (usually retryable) |
| `NO_PRODUCT_SELECTED` | Business | No product selected for order |
| `ORDER_CREATION_FAILED` | Business | Order creation failed |
| `PAYMENT_FAILED` | Business | Payment processing failed |
| `ORDER_CONFIRMATION_FAILED` | Business | Order confirmation failed |
| `ESIM_INSTALLATION_FAILED` | Installation | Installation failed on device |
| `ESIM_EUICC_DISABLED` | Installation | eSIM hardware disabled in settings |
| `PARSE_ERROR` | Other | Failed to parse API response |
| `UNKNOWN` | Other | Unexpected error |

---

### 4.5 EsimInstallationResult

Received via `onEsimInstalled` callback after a device installation attempt.

```kotlin
data class EsimInstallationResult(
    val success: Boolean,         // Whether the eSIM was installed
    val orderId: String?,         // Associated order ID
    val errorMessage: String?,    // Error details if failed (null on success)
    val status: Status            // Detailed status enum
) {
    enum class Status {
        INSTALLED,                // eSIM installed successfully
        NOT_SUPPORTED,            // Device doesn't support eSIM (API < 28)
        EUICC_DISABLED,           // eSIM hardware disabled in settings
        USER_CANCELLED,           // User dismissed system consent dialog
        FAILED                    // Installation failed (see errorMessage)
    }
}
```

---

### 4.6 EsimProductSummary

```kotlin
data class EsimProductSummary(
    val productName: String,      // Name of the eSIM product
    val dataLimit: String,        // Data allowance
    val validityDays: Int,        // Validity period in days
    val coverageCount: Int        // Number of countries covered
)
```

---

### 4.7 EsimException

Exception wrapper for `EsimError`, useful for try/catch patterns.

```kotlin
class EsimException(val error: EsimError) : Exception(error.message, error.cause) {
    val code: EsimErrorCode           // Delegates to error.code
    val isRetryable: Boolean          // Delegates to error.isRetryable
    val httpStatusCode: Int?          // Delegates to error.httpStatusCode
}
```

---

## 5. Sample Code

### Complete Integration with Payment Resume

```kotlin
// Application.kt
class MyApp : Application() {
    override fun onCreate() {
        super.onCreate()
        Gate2TravelSdk.initialize(
            Gate2SdkConfig.builder()
                .context(this)
                .apiKey(BuildConfig.GATE2_API_KEY)
                .build()
        )
    }
}

// EsimActivity.kt
class EsimActivity : ComponentActivity() {

    private var pendingOrderId: String? = null
    private var pendingSessionId: String? = null

    private val callbacks by lazy {
        EsimCallbacks(
            onComplete = {
                Toast.makeText(this, "eSIM activated!", Toast.LENGTH_SHORT).show()
                finish()
            },
            onCancel = {
                finish()
            },
            onFail = { error ->
                when (error.code) {
                    EsimErrorCode.NETWORK_ERROR -> {
                        if (error.isRetryable) showRetryDialog()
                        else showError(error.message)
                    }
                    EsimErrorCode.UNAUTHORIZED -> {
                        showError("Authentication failed. Check your API key.")
                    }
                    else -> showError(error.message)
                }
            },
            onProcessPayment = { request ->
                // Store both values for resumeAfterPayment()
                pendingOrderId = request.orderId
                pendingSessionId = request.sessionId
                handlePayment(request)
            },
            onEsimInstalled = { result ->
                when (result.status) {
                    EsimInstallationResult.Status.INSTALLED ->
                        Log.d("eSIM", "Successfully installed for order ${result.orderId}")
                    EsimInstallationResult.Status.USER_CANCELLED ->
                        Log.d("eSIM", "User cancelled installation dialog")
                    else ->
                        Log.w("eSIM", "Installation failed: ${result.errorMessage}")
                }
            }
        )
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        Gate2Esim.start(
            activity = this,
            sessionId = "your-session-id",
            callbacks = callbacks,
            userId = "user-123",
            language = "en",
            accentColor = 0xFF1B63ED.toInt(),
            title = "eSIM"
        )
    }

    private fun handlePayment(request: EsimPaymentRequest) {
        // Process with your payment provider
        yourPaymentProvider.charge(request.amount, request.currency) { result ->
            when (result) {
                is Success -> {
                    // Resume SDK flow after successful payment
                    Gate2Esim.resumeAfterPayment(
                        activity = this,
                        orderId = pendingOrderId!!,
                        sessionId = pendingSessionId!!,
                        callbacks = callbacks,
                        language = "en"
                    )
                }
                is Failed -> {
                    showError("Payment failed: ${result.error}")
                }
            }
        }
    }
}
```

### Supported Destinations

The SDK supports eSIM data plans for **190+ countries and regions**:

| Region | Countries |
|--------|-----------|
| **Europe** | France, Germany, Italy, Spain, UK, Netherlands, Switzerland, etc. |
| **Asia** | Japan, South Korea, Thailand, Singapore, Malaysia, Indonesia, etc. |
| **Americas** | USA, Canada, Mexico, Brazil, Argentina, etc. |
| **Middle East** | UAE, Saudi Arabia, Qatar, Turkey, Israel, etc. |
| **Oceania** | Australia, New Zealand, Fiji, etc. |

---

## 6. My eSIMs

Display the list of purchased eSIMs for a user.

```kotlin
Gate2Esim.showMyEsims(
    activity = this,
    sessionId = sessionId,
    callbacks = EsimCallbacks(
        onComplete = { finish() },
        onCancel = { finish() },
        onFail = { error -> showError(error.message) },
        onProcessPayment = { request ->
            // Handle top-up payments (request.isTopUp == true)
            pendingOrderId = request.orderId
            pendingSessionId = request.sessionId
            handlePayment(request)
        },
        onEsimInstalled = { result ->
            if (result.success) {
                Toast.makeText(this, "eSIM activated!", Toast.LENGTH_SHORT).show()
            }
        }
    ),
    userId = "user-123",
    language = "en"
)
```

> **Note**: The My eSIMs screen supports top-up purchases. The `onProcessPayment` callback will fire with `isTopUp = true` and the existing `iccid` when the user tops up an existing eSIM.

---

## 7. Device eSIM Check

Check if the device supports eSIM before showing eSIM features:

```kotlin
if (Gate2Esim.isDeviceEsimCapable(context)) {
    // Show eSIM purchasing option
} else {
    // Hide eSIM features or show informational message
}
```

> Returns `false` on devices running Android 8.1 or below (API < 28) where `EuiccManager` is not available.

### Module Availability Check

```kotlin
if (Gate2Esim.isAvailable()) {
    // SDK is initialized AND eSIM module access is granted by token
    showEsimButton()
}
```

---

## 8. Error Handling

### Structured Errors

The `onFail` callback receives an `EsimError` object with structured information:

```kotlin
onFail = { error ->
    when (error.code) {
        EsimErrorCode.NETWORK_ERROR -> {
            if (error.isRetryable) showRetryDialog()
            else showError(error.message)
        }
        EsimErrorCode.UNAUTHORIZED -> navigateToLogin()
        EsimErrorCode.RATE_LIMITED -> showError("Too many requests. Please wait.")
        EsimErrorCode.SERVER_ERROR -> {
            if (error.isRetryable) showRetryDialog()
        }
        else -> showError(error.message)
    }

    // Log for analytics
    analytics.logError(
        code = error.code.name,
        statusCode = error.httpStatusCode,
        message = error.message,
        retryable = error.isRetryable
    )
}
```

### Backward Compatibility

Existing integrations using `(String) -> Unit` for `onFail` continue to work:

```kotlin
EsimCallbacks(
    onComplete = { /* ... */ },
    onCancel = { /* ... */ },
    onFail = { message: String -> showError(message) },
    onProcessPayment = { request -> handlePayment(request) }
)
```

### Common Error Scenarios

| Error Code | Typical Cause | Recommended Action |
|------------|--------------|-------------------|
| `NETWORK_ERROR` | No internet / timeout | Show retry option |
| `UNAUTHORIZED` | Invalid API key | Check configuration |
| `MODULE_ACCESS_DENIED` | Token missing eSIM permission | Contact Gate2 support |
| `INVALID_SESSION_ID` | Blank session ID passed | Fix caller code |
| `INVALID_ORDER_ID` | Malformed order ID | Fix caller code |
| `SERVER_ERROR` | Backend issue | Show retry option |
| `ESIM_INSTALLATION_FAILED` | Device installation error | Show manual QR option |
| `ESIM_EUICC_DISABLED` | eSIM hardware off | Prompt user to enable |

---

## 9. Theming & Branding

The eSIM SDK supports accent color and custom font customization through `Gate2Esim.start()` and `Gate2Esim.showMyEsims()` parameters:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `accentColor` | `Int` (ARGB) | `0xFF1B63ED` (blue) | Brand accent color for buttons, highlights |
| `fontResId` | `Int` (`@FontRes`) | `0` (Inter) | Custom font resource ID |
| `title` | `String` | `"eSIM"` | Toolbar title |

### Example

```kotlin
Gate2Esim.start(
    activity = this,
    sessionId = sessionId,
    callbacks = callbacks,
    accentColor = 0xFF6200EE.toInt(),    // Purple brand
    fontResId = R.font.my_custom_font,    // Custom font
    title = "Data Plans"                  // Custom title
)
```

---

## 10. System Requirements

### Minimum Requirements

| Requirement | Version | Notes |
|-------------|---------|-------|
| **Android minSdk** | 27 | Android 8.1 Oreo |
| **Android compileSdk** | 35 | Android 15 |
| **Kotlin** | 2.2.21 | Required for language features |
| **Java** | 11 | JVM target |
| **Jetpack Compose** | BOM 2025.12.00 | Material3 components |

### Device Compatibility

| Feature | Requirement |
|---------|-------------|
| **eSIM Hardware** | Required for direct installation (Android 9+) |
| **QR Code** | Works on all devices (user scans with Settings) |
| **Network** | Internet connection required |
| **Play Services** | Not required |

### Gradle Setup

```kotlin
// app/build.gradle.kts
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
    id("org.jetbrains.kotlin.plugin.compose")
}

android {
    namespace = "com.yourcompany.app"
    compileSdk = 35

    defaultConfig {
        minSdk = 27
        targetSdk = 35
    }

    buildFeatures {
        compose = true
        buildConfig = true
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_11
        targetCompatibility = JavaVersion.VERSION_11
    }

    kotlinOptions {
        jvmTarget = "11"
    }
}

dependencies {
    // Gate2 eSIM SDK
    implementation("travel.gate2:esim:1.0.6")

    // AndroidX Core
    implementation("androidx.core:core-ktx:1.17.0")
    implementation("androidx.activity:activity-compose:1.12.1")

    // Compose
    implementation(platform("androidx.compose:compose-bom:2025.12.00"))
    implementation("androidx.compose.ui:ui")
    implementation("androidx.compose.material3:material3")
}
```

### Repository Configuration

```kotlin
// settings.gradle.kts
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
    }
}
```

---

## 11. Troubleshooting

### Quick Diagnostic Checklist

- [ ] SDK initialized in `Application.onCreate()` via `Gate2TravelSdk.initialize()`
- [ ] Valid API key configured
- [ ] Internet connection available
- [ ] Compatible Android version (API 27+)
- [ ] All required callbacks implemented in `EsimCallbacks`
- [ ] Storing `orderId` and `sessionId` from `onProcessPayment`
- [ ] Calling `resumeAfterPayment()` after payment completes

### Common Issues

#### "SDK not initialized"

```kotlin
// Ensure initialization in Application.onCreate()
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        Gate2TravelSdk.initialize(
            Gate2SdkConfig.builder()
                .context(this)
                .apiKey("your-api-key")
                .build()
        )
    }
}

// Register in AndroidManifest.xml
<application android:name=".MyApplication">
```

#### Payment resume not working

Ensure you are storing **both** `orderId` and `sessionId` from `EsimPaymentRequest`:

```kotlin
onProcessPayment = { request ->
    // MUST store both values
    pendingOrderId = request.orderId
    pendingSessionId = request.sessionId
    processPayment(request)
}
```

#### eSIM installation fails

- Check `Gate2Esim.isDeviceEsimCapable(context)` returns `true`
- Verify eSIM is enabled in device Settings > Network > SIM
- Android 9+ (API 28) is required for direct installation

---

## 12. Support

| Resource | Contact |
|----------|---------|
| **SDK Documentation** | [GitHub Page](https://github.com/gate2-travel/demo-android/blob/main/README.md) |
| **API Documentation** | [Postman Collection](https://documenter.getpostman.com/view/7713462/2sB3dWq6UM) |
| **Developer Portal** | [developers.gate2.travel](https://developers.gate2.travel) |
| **GitHub Issues** | [github.com/gate2-travel/demo-android](https://github.com/gate2-travel/demo-android) |
| **Email Support** | [support@gate2.travel](mailto:support@gate2.travel) |

---

## Glossary

| Term | Definition |
|------|------------|
| **eSIM** | Embedded SIM — digital SIM without physical card |
| **SM-DP+** | Subscription Manager for eSIM profiles |
| **QR Code** | Quick Response code for eSIM activation |
| **EID** | eSIM Identifier |
| **ICCID** | Integrated Circuit Card Identifier — unique SIM card number |
| **eUICC** | Embedded Universal Integrated Circuit Card — eSIM hardware chip |
| **mTLS** | Mutual TLS — client + server certificate authentication |
| **Top-Up** | Adding additional data to an existing eSIM profile |

---

## Changelog

### v1.0.6 (February 2026)
- New API: `Gate2Esim.start()`, `resumeAfterPayment()`, `showMyEsims()`
- New API: `isAvailable()`, `isDeviceEsimCapable()`
- Structured error handling with `EsimError` and `EsimErrorCode` (20 error codes)
- eSIM device installation via `onEsimInstalled` callback
- My eSIMs management screen with top-up support
- `EsimPaymentRequest` with `sessionId`, `isTopUp`, `iccid` fields
- Maven coordinates: `travel.gate2:esim`
- R8 log stripping, AES-256-GCM encrypted mTLS certificate
- Backward-compatible string-based `onFail` support

### v1.0.0 (January 2026)
- Initial release
- eSIM data plan purchasing and activation
- Certificate pinning
- Debug logging support

---

<div align="center">

**Gate2 eSIM SDK** v1.0.6 — eSIM Data Plan Integration for Android

&copy; 2026 Gate2 Travel. All rights reserved.

</div>
