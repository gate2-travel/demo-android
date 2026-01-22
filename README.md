<div align="center">

# Gate2 SDK Suite

### Complete Travel Services Solution for Android

[![SDK Version](https://img.shields.io/badge/SDK-1.0.0-blue.svg)](https://github.com/gate2-travel/demo-android)
[![Platform](https://img.shields.io/badge/Platform-Android-green.svg)](https://developer.android.com)
[![Min SDK](https://img.shields.io/badge/minSdk-27-orange.svg)](https://developer.android.com)
[![Kotlin](https://img.shields.io/badge/Kotlin-2.2.21-purple.svg)](https://kotlinlang.org)
[![Compose](https://img.shields.io/badge/Jetpack%20Compose-BOM%202025.12-teal.svg)](https://developer.android.com/jetpack/compose)

**Integrate complete travel services into your Android app with just 10 lines of code per service.**

| [Travel SDK](#part-1-gate2-travel-sdk) | [eSIM SDK](#part-2-gate2-esim-sdk) | [Hotel SDK](#part-3-gate2-hotel-sdk) |
|:---:|:---:|:---:|
| Flight Booking | Data Plans | Hotel Reservations |

</div>

---

> **Document Version**: 1.0.0
> **Last Updated**: January 22, 2026
> **Compatibility**: Android 8.1+ (API 27+)

---

## Master Table of Contents

### Part 1: Gate2 Travel SDK
- [1.1 Overview](#11-travel-sdk-overview)
- [1.2 Quick Start](#12-travel-quick-start)
- [1.3 Booking Flow](#13-travel-booking-flow)
- [1.4 API Models](#14-travel-api-models)
- [1.5 Sample Code](#15-travel-sample-code)

### Part 2: Gate2 eSIM SDK
- [2.1 Overview](#21-esim-sdk-overview)
- [2.2 Quick Start](#22-esim-quick-start)
- [2.3 Purchase Flow](#23-esim-purchase-flow)
- [2.4 API Models](#24-esim-api-models)
- [2.5 Sample Code](#25-esim-sample-code)

### Part 3: Gate2 Hotel SDK
- [3.1 Overview](#31-hotel-sdk-overview)
- [3.2 Quick Start](#32-hotel-quick-start)
- [3.3 Booking Flow](#33-hotel-booking-flow)
- [3.4 API Models](#34-hotel-api-models)
- [3.5 Sample Code](#35-hotel-sample-code)

### Part 4: Common Components
- [4.1 System Requirements](#41-system-requirements)
- [4.2 Installation & Setup](#42-installation--setup)
- [4.3 Theming & Branding](#43-theming--branding)
- [4.4 Payment Integration](#44-payment-integration)
- [4.5 Error Handling](#45-error-handling)
- [4.6 Network & Security](#46-network--security)
- [4.7 Troubleshooting](#47-troubleshooting)
- [4.8 Support Contacts](#48-support-contacts)

---

# Part 1: Gate2 Travel SDK

## 1.1 Travel SDK Overview

### What is Gate2 Travel SDK?

The **Gate2 Travel SDK** is a comprehensive, production-ready Android library that enables host applications to integrate complete flight booking functionality with minimal effort. Built with modern Android technologies including **Jetpack Compose** and **Kotlin Coroutines**, the SDK handles the entire booking flow from flight search to confirmation.

### Key Capabilities

| Capability | Description |
|:-----------|:------------|
| **Flight Search** | Search for flights by origin, destination, dates, passengers, and travel class |
| **Real-time Pricing** | Get accurate, up-to-date pricing with fare rules before booking |
| **Traveler Management** | Collect and validate traveler information with passport/document support |
| **Secure Payments** | PCI-compliant payment collection with flexible integration options |
| **Booking Confirmation** | Complete booking with PNR generation and confirmation details |
| **Custom Theming** | Full brand customization with 20+ color properties |

### Integration at a Glance

```
┌─────────────────────────────────────────────────────────────────┐
│                      YOUR APPLICATION                           │
├─────────────────────────────────────────────────────────────────┤
│  ┌───────────────┐    ┌──────────────────────────────────────┐ │
│  │  Initialize   │───▶│         Gate2TravelScreen            │ │
│  │  (1 method)   │    │  ┌────────────────────────────────┐  │ │
│  └───────────────┘    │  │  Complete Booking Flow         │  │ │
│                       │  │  • Flight Search               │  │ │
│  ┌───────────────┐    │  │  • Pricing & Selection         │  │ │
│  │   Callbacks   │◀───│  │  • Traveler Information        │  │ │
│  │  • onComplete │    │  │  • Payment Processing          │  │ │
│  │  • onCancel   │    │  │  • Booking Confirmation        │  │ │
│  │  • onFail     │    │  └────────────────────────────────┘  │ │
│  │  • onPayment  │    └──────────────────────────────────────┘ │
│  └───────────────┘                                              │
└─────────────────────────────────────────────────────────────────┘
```

### Integration Complexity

| Aspect | Rating | Notes |
|:-------|:------:|:------|
| **Lines of Code** | ~10-20 | Minimal integration required |
| **API Surface** | 2 methods | `initialize()` + `Gate2TravelScreen()` |
| **Configuration** | Simple | Builder pattern with sensible defaults |
| **Theming** | Optional | Works out-of-the-box with defaults |
| **Time to Integrate** | < 1 hour | From setup to first booking |

### Responsibilities Matrix

| SDK Handles | Host App Handles |
|:------------|:-----------------|
| Flight search UI and logic | SDK initialization with API key |
| Real-time pricing display | Implement callback handlers |
| Traveler form validation | Optional: Custom theming |
| Payment collection UI | Optional: Payment processing |
| Booking creation & confirmation | Navigation after completion |
| Network security & retries | Error display (message provided) |
| State management | |
| Localization support | |

---

## 1.2 Travel Quick Start

> **Time to Complete**: 5 minutes

### Step 1: Add Dependency

```kotlin
// app/build.gradle.kts
dependencies {
    implementation("com.gate2.sdk:travel:1.0.0")
}
```

### Step 2: Initialize SDK

```kotlin
import com.gate2.sdk.travel.Gate2TravelSdk
import com.gate2.sdk.travel.config.Gate2SdkConfig

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

### Step 3: Add the Booking Screen

```kotlin
import com.gate2.sdk.travel.ui.Gate2TravelScreen
import com.gate2.sdk.travel.ui.theme.Gate2TravelTheme

class BookingActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContent {
            Gate2TravelTheme {
                Gate2TravelScreen(
                    onComplete = {
                        // Booking successful - navigate to success screen
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

    private fun processPayment(request: PaymentRequest) {
        // Your payment processing logic
        // Call Gate2TravelSdk.notifyPaymentResult() when done
    }
}
```

### That's It!

Your app now has complete flight booking functionality.

---

## 1.3 Travel Booking Flow

The SDK implements a complete **6-screen booking flow**:

```
┌──────────────────────────────────────────────────────────────────────────┐
│                           BOOKING FLOW                                    │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐   │
│  │ SEARCH  │──▶│ PRICING │──▶│TRAVELERS│──▶│ REVIEW  │──▶│ PAYMENT │   │
│  │         │   │         │   │         │   │         │   │         │   │
│  │ Where?  │   │ Confirm │   │ Who's   │   │ Verify  │   │ Pay &   │   │
│  │ When?   │   │ price   │   │ flying? │   │ details │   │ confirm │   │
│  │ Who?    │   │         │   │         │   │         │   │         │   │
│  └─────────┘   └─────────┘   └─────────┘   └─────────┘   └─────────┘   │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

### Screen Details

#### 1. Flight Search
- Origin/destination airport selection
- Departure/return dates
- Passenger count (adults, children, infants)
- Travel class (Economy, Business, First)

#### 2. Flight Pricing
- Selected flight details
- Current price confirmation
- Fare rules and conditions
- Baggage allowance

#### 3. Travelers
- Traveler information collection
- Name, gender, date of birth
- Passport/ID details (optional)
- Contact information

#### 4. Booking Review
- Complete itinerary review
- All traveler details
- Price breakdown
- Terms and conditions

#### 5. Payment
- Card information collection
- PCI-compliant processing
- Secure transmission

#### 6. Confirmation
- PNR (Passenger Name Record)
- Booking reference
- Complete itinerary
- Next steps

---

## 1.4 Travel API Models

### PaymentRequest

```kotlin
data class PaymentRequest(
    val orderId: String,              // Unique order identifier
    val pnr: String,                  // Passenger Name Record
    val amount: Double,               // Total amount to charge
    val currency: String,             // ISO 4217 currency code
    val travelers: List<TravelerSummary>,
    val flightDetails: FlightSummary,
    val contactEmail: String,
    val contactPhone: String?
)
```

### TravelerSummary

```kotlin
data class TravelerSummary(
    val firstName: String,
    val lastName: String,
    val type: TravelerType            // ADULT, CHILD, INFANT
)
```

### FlightSummary

```kotlin
data class FlightSummary(
    val origin: String,               // Origin airport IATA code
    val destination: String,          // Destination airport IATA code
    val departureDate: String,        // YYYY-MM-DD
    val returnDate: String?,          // For round-trips
    val airlineCode: String,
    val flightNumber: String
)
```

### PaymentResult

```kotlin
sealed class PaymentResult {
    data class Success(
        val transactionId: String,
        val authorizationCode: String?
    ) : PaymentResult()

    data class Failure(
        val errorCode: String?,
        val message: String,
        val canRetry: Boolean
    ) : PaymentResult()

    data object Cancelled : PaymentResult()

    data class RequiresAction(
        val actionType: String,
        val actionData: Map<String, String>
    ) : PaymentResult()
}
```

---

## 1.5 Travel Sample Code

### Minimal Integration

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

// BookingActivity.kt
class BookingActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            Gate2TravelTheme {
                Gate2TravelScreen(
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

    private fun handlePayment(request: PaymentRequest) {
        // Process with your payment provider
        yourPaymentProvider.charge(request.amount, request.currency) { result ->
            when (result) {
                is Success -> Gate2TravelSdk.notifyPaymentResult(
                    PaymentResult.Success(transactionId = result.id)
                )
                is Failed -> Gate2TravelSdk.notifyPaymentResult(
                    PaymentResult.Failure(message = result.error, canRetry = true)
                )
            }
        }
    }
}
```

---

# Part 2: Gate2 eSIM SDK

## 2.1 eSIM SDK Overview

### What is Gate2 eSIM SDK?

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

## 2.2 eSIM Quick Start

> **Time to Complete**: 5 minutes

### Step 1: Add Dependency

```kotlin
// app/build.gradle.kts
dependencies {
    implementation("com.gate2.sdk:esim:1.0.0")
}
```

### Step 2: Initialize SDK

```kotlin
import com.gate2.sdk.esim.Gate2EsimSdk
import com.gate2.sdk.esim.config.Gate2SdkConfig

class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        Gate2EsimSdk.initialize(
            Gate2SdkConfig.builder()
                .context(this)
                .apiKey("your-api-key-here")
                .build()
        )
    }
}
```

### Step 3: Add the eSIM Screen

```kotlin
import com.gate2.sdk.esim.ui.Gate2EsimScreen
import com.gate2.sdk.esim.ui.theme.Gate2EsimTheme

class EsimActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContent {
            Gate2EsimTheme {
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

    private fun processPayment(request: PaymentRequest) {
        // Your payment processing logic
        // Call Gate2EsimSdk.notifyPaymentResult() when done
    }
}
```

### That's It!

Your app now has complete eSIM purchasing functionality.

---

## 2.3 eSIM Purchase Flow

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

## 2.4 eSIM API Models

### PaymentRequest

```kotlin
data class PaymentRequest(
    val orderId: String,              // Unique order identifier
    val amount: Double,               // Total amount to charge
    val currency: String,             // ISO 4217 currency code
    val planDetails: PlanSummary,     // eSIM plan information
    val contactEmail: String,
    val contactPhone: String?
)
```

### PlanSummary

```kotlin
data class PlanSummary(
    val planId: String,               // Unique plan identifier
    val destination: String,          // Destination country/region
    val dataAllowance: String,        // e.g., "5GB", "Unlimited"
    val validity: String,             // e.g., "30 days"
    val networkType: String           // e.g., "4G/LTE", "5G"
)
```

### PaymentResult

```kotlin
sealed class PaymentResult {
    data class Success(
        val transactionId: String,
        val authorizationCode: String?
    ) : PaymentResult()

    data class Failure(
        val errorCode: String?,
        val message: String,
        val canRetry: Boolean
    ) : PaymentResult()

    data object Cancelled : PaymentResult()

    data class RequiresAction(
        val actionType: String,
        val actionData: Map<String, String>
    ) : PaymentResult()
}
```

---

## 2.5 eSIM Sample Code

### Minimal Integration

```kotlin
// Application.kt
class MyApp : Application() {
    override fun onCreate() {
        super.onCreate()
        Gate2EsimSdk.initialize(
            Gate2SdkConfig.builder()
                .context(this)
                .apiKey(BuildConfig.GATE2_API_KEY)
                .build()
        )
    }
}

// EsimActivity.kt
class EsimActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            Gate2EsimTheme {
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

    private fun handlePayment(request: PaymentRequest) {
        // Process with your payment provider
        yourPaymentProvider.charge(request.amount, request.currency) { result ->
            when (result) {
                is Success -> Gate2EsimSdk.notifyPaymentResult(
                    PaymentResult.Success(transactionId = result.id)
                )
                is Failed -> Gate2EsimSdk.notifyPaymentResult(
                    PaymentResult.Failure(message = result.error, canRetry = true)
                )
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

# Part 3: Gate2 Hotel SDK

## 3.1 Hotel SDK Overview

### What is Gate2 Hotel SDK?

The **Gate2 Hotel SDK** is a comprehensive, production-ready Android library that enables host applications to integrate complete hotel booking functionality with minimal effort. The SDK handles the entire booking flow from hotel search to confirmation.

### Key Capabilities

| Capability | Description |
|:-----------|:------------|
| **Hotel Search** | Search hotels by destination, dates, guests, and room requirements |
| **Real-time Pricing** | Get accurate, up-to-date pricing with room details and availability |
| **Rich Content** | High-quality photos, amenities, reviews, and location maps |
| **Guest Management** | Collect and validate guest information with preferences |
| **Secure Payments** | PCI-compliant payment collection with flexible integration options |
| **Booking Confirmation** | Complete booking with confirmation number and details |
| **Custom Theming** | Full brand customization with 20+ color properties |

### Integration at a Glance

```
┌─────────────────────────────────────────────────────────────────┐
│                      YOUR APPLICATION                           │
├─────────────────────────────────────────────────────────────────┤
│  ┌───────────────┐    ┌──────────────────────────────────────┐ │
│  │  Initialize   │───▶│         Gate2HotelScreen             │ │
│  │  (1 method)   │    │  ┌────────────────────────────────┐  │ │
│  └───────────────┘    │  │  Complete Booking Flow         │  │ │
│                       │  │  • Hotel Search                │  │ │
│  ┌───────────────┐    │  │  • Room Selection              │  │ │
│  │   Callbacks   │◀───│  │  • Guest Information           │  │ │
│  │  • onComplete │    │  │  • Payment Processing          │  │ │
│  │  • onCancel   │    │  │  • Booking Confirmation        │  │ │
│  │  • onFail     │    │  └────────────────────────────────┘  │ │
│  │  • onPayment  │    └──────────────────────────────────────┘ │
│  └───────────────┘                                              │
└─────────────────────────────────────────────────────────────────┘
```

### Integration Complexity

| Aspect | Rating | Notes |
|:-------|:------:|:------|
| **Lines of Code** | ~10-20 | Minimal integration required |
| **API Surface** | 2 methods | `initialize()` + `Gate2HotelScreen()` |
| **Configuration** | Simple | Builder pattern with sensible defaults |
| **Theming** | Optional | Works out-of-the-box with defaults |
| **Time to Integrate** | < 1 hour | From setup to first booking |

### Responsibilities Matrix

| SDK Handles | Host App Handles |
|:------------|:-----------------|
| Hotel search UI and logic | SDK initialization with API key |
| Real-time pricing display | Implement callback handlers |
| Photo gallery and maps | Optional: Custom theming |
| Guest form validation | Optional: Payment processing |
| Payment collection UI | Navigation after completion |
| Booking creation & confirmation | Error display (message provided) |
| State management | |
| Localization support | |

---

## 3.2 Hotel Quick Start

> **Time to Complete**: 5 minutes

### Step 1: Add Dependency

```kotlin
// app/build.gradle.kts
dependencies {
    implementation("com.gate2.sdk:hotel:1.0.0")
}
```

### Step 2: Initialize SDK

```kotlin
import com.gate2.sdk.hotel.Gate2HotelSdk
import com.gate2.sdk.hotel.config.Gate2SdkConfig

class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        Gate2HotelSdk.initialize(
            Gate2SdkConfig.builder()
                .context(this)
                .apiKey("your-api-key-here")
                .build()
        )
    }
}
```

### Step 3: Add the Booking Screen

```kotlin
import com.gate2.sdk.hotel.ui.Gate2HotelScreen
import com.gate2.sdk.hotel.ui.theme.Gate2HotelTheme

class HotelActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        setContent {
            Gate2HotelTheme {
                Gate2HotelScreen(
                    onComplete = {
                        // Booking successful - navigate to success screen
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

    private fun processPayment(request: PaymentRequest) {
        // Your payment processing logic
        // Call Gate2HotelSdk.notifyPaymentResult() when done
    }
}
```

### That's It!

Your app now has complete hotel booking functionality.

---

## 3.3 Hotel Booking Flow

The SDK implements a complete **6-screen booking flow**:

```
┌──────────────────────────────────────────────────────────────────────────┐
│                           BOOKING FLOW                                    │
├──────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐   ┌─────────┐   │
│  │ SEARCH  │──▶│  LIST   │──▶│ DETAILS │──▶│  ROOM   │──▶│  GUEST  │   │
│  │         │   │         │   │         │   │         │   │         │   │
│  │ Where?  │   │ Browse  │   │ Photos  │   │ Select  │   │ Who's   │   │
│  │ When?   │   │ hotels  │   │ Amenity │   │ room    │   │ staying?│   │
│  │ Guests? │   │         │   │ Reviews │   │         │   │         │   │
│  └─────────┘   └─────────┘   └─────────┘   └─────────┘   └─────────┘   │
│                                                     │                    │
│                                    ┌─────────┐   ┌─────────┐            │
│                                    │ PAYMENT │──▶│ CONFIRM │            │
│                                    │         │   │         │            │
│                                    │ Pay &   │   │ Success │            │
│                                    │ confirm │   │ details │            │
│                                    └─────────┘   └─────────┘            │
│                                                                          │
└──────────────────────────────────────────────────────────────────────────┘
```

### Screen Details

#### 1. Hotel Search
- Destination search
- Check-in/Check-out dates
- Number of rooms and guests

#### 2. Hotel List
- Browse search results
- Filter by price, rating, amenities
- Sort options

#### 3. Hotel Details
- Photo gallery
- Full description
- Amenities list
- Location map
- Guest reviews

#### 4. Room Selection
- Available room types
- Bed configurations
- Price per night
- Cancellation policy

#### 5. Guest Information
- Guest name and contact
- Special requests
- Arrival time

#### 6. Payment
- Card information
- PCI-compliant processing

#### 7. Confirmation
- Confirmation number
- Booking details
- Hotel contact info

---

## 3.4 Hotel API Models

### PaymentRequest

```kotlin
data class PaymentRequest(
    val orderId: String,              // Unique order identifier
    val confirmationNumber: String,   // Booking confirmation number
    val amount: Double,               // Total amount to charge
    val currency: String,             // ISO 4217 currency code
    val guests: List<GuestSummary>,
    val hotelDetails: HotelSummary,
    val contactEmail: String,
    val contactPhone: String?
)
```

### GuestSummary

```kotlin
data class GuestSummary(
    val firstName: String,
    val lastName: String,
    val isMainGuest: Boolean
)
```

### HotelSummary

```kotlin
data class HotelSummary(
    val hotelId: String,
    val hotelName: String,
    val address: String,
    val city: String,
    val country: String,
    val checkInDate: String,          // YYYY-MM-DD
    val checkOutDate: String,         // YYYY-MM-DD
    val roomType: String,
    val numberOfRooms: Int,
    val numberOfNights: Int
)
```

### PaymentResult

```kotlin
sealed class PaymentResult {
    data class Success(
        val transactionId: String,
        val authorizationCode: String?
    ) : PaymentResult()

    data class Failure(
        val errorCode: String?,
        val message: String,
        val canRetry: Boolean
    ) : PaymentResult()

    data object Cancelled : PaymentResult()

    data class RequiresAction(
        val actionType: String,
        val actionData: Map<String, String>
    ) : PaymentResult()
}
```

---

## 3.5 Hotel Sample Code

### Minimal Integration

```kotlin
// Application.kt
class MyApp : Application() {
    override fun onCreate() {
        super.onCreate()
        Gate2HotelSdk.initialize(
            Gate2SdkConfig.builder()
                .context(this)
                .apiKey(BuildConfig.GATE2_API_KEY)
                .build()
        )
    }
}

// HotelActivity.kt
class HotelActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            Gate2HotelTheme {
                Gate2HotelScreen(
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

    private fun handlePayment(request: PaymentRequest) {
        // Process with your payment provider
        yourPaymentProvider.charge(request.amount, request.currency) { result ->
            when (result) {
                is Success -> Gate2HotelSdk.notifyPaymentResult(
                    PaymentResult.Success(transactionId = result.id)
                )
                is Failed -> Gate2HotelSdk.notifyPaymentResult(
                    PaymentResult.Failure(message = result.error, canRetry = true)
                )
            }
        }
    }
}
```

---

# Part 4: Common Components

## 4.1 System Requirements

### Minimum Requirements

| Requirement | Version | Notes |
|-------------|---------|-------|
| **Android minSdk** | 27 | Android 8.1 Oreo |
| **Android compileSdk** | 36 | Android 16 |
| **Kotlin** | 2.2.21 | Required for language features |
| **Java** | 11 | JVM target |
| **Jetpack Compose** | BOM 2025.12.00 | Material3 components |

### Gradle Plugin Requirements

```kotlin
// build.gradle.kts (project level)
plugins {
    id("com.android.application") version "8.7.3" apply false
    id("org.jetbrains.kotlin.android") version "2.2.21" apply false
    id("org.jetbrains.kotlin.plugin.compose") version "2.2.21" apply false
}
```

### Device Compatibility

| Feature | Requirement |
|---------|-------------|
| **Screen Size** | Phone and Tablet optimized |
| **Orientation** | Portrait (primary), Landscape (supported) |
| **Network** | Internet connection required |
| **eSIM Support** | Device must support eSIM for eSIM SDK activation |
| **Play Services** | Not required |

### Permissions

```xml
<!-- Automatically merged from SDK manifest -->
<uses-permission android:name="android.permission.INTERNET" />
```

---

## 4.2 Installation & Setup

### Adding Dependencies

```kotlin
// app/build.gradle.kts
dependencies {
    // Choose the SDKs you need:
    implementation("com.gate2.sdk:travel:1.0.0")  // Flight booking
    implementation("com.gate2.sdk:esim:1.0.0")    // eSIM data plans
    implementation("com.gate2.sdk:hotel:1.0.0")   // Hotel booking
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
        mavenLocal()  // For local development
        // Or use Gate2 Maven repository for production:
        maven {
            url = uri("https://maven.gate2.travel/releases")
            credentials {
                username = providers.gradleProperty("gate2.username").orNull
                password = providers.gradleProperty("gate2.token").orNull
            }
        }
    }
}
```

### Compose Configuration

```kotlin
// app/build.gradle.kts
android {
    buildFeatures {
        compose = true
    }
}

plugins {
    id("org.jetbrains.kotlin.plugin.compose")
}
```

### Complete Gradle Setup

```kotlin
// app/build.gradle.kts
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")
    id("org.jetbrains.kotlin.plugin.compose")
}

android {
    namespace = "com.yourcompany.app"
    compileSdk = 36

    defaultConfig {
        applicationId = "com.yourcompany.app"
        minSdk = 27
        targetSdk = 36
        versionCode = 1
        versionName = "1.0.0"
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
    // Gate2 SDKs
    implementation("com.gate2.sdk:travel:1.0.0")
    implementation("com.gate2.sdk:esim:1.0.0")
    implementation("com.gate2.sdk:hotel:1.0.0")

    // AndroidX Core
    implementation("androidx.core:core-ktx:1.17.0")
    implementation("androidx.activity:activity-compose:1.12.1")

    // Compose
    implementation(platform("androidx.compose:compose-bom:2025.12.00"))
    implementation("androidx.compose.ui:ui")
    implementation("androidx.compose.material3:material3")

    // Lifecycle
    implementation("androidx.lifecycle:lifecycle-runtime-compose:2.10.0")
}
```

---

## 4.3 Theming & Branding

The SDK provides comprehensive customization options to match your brand identity. Every visual element can be configured including colors, shapes, buttons, inputs, cards, and typography.

### Theme Architecture

```
Gate2Theme
├── Colors
│   ├── PrimaryColors (main, dark, light, onPrimary)
│   ├── SecondaryColors (main, dark, light, onSecondary)
│   ├── BackgroundColors (background, surface, card, divider, border)
│   ├── TextColors (primary, secondary, hint, disabled, link)
│   └── StatusColors (success, warning, error, info)
├── Shapes
│   ├── ButtonShape (cornerRadius, borderWidth)
│   ├── CardShape (cornerRadius, elevation, shadowColor)
│   ├── InputShape (cornerRadius, borderWidth, borderColor)
│   └── DialogShape (cornerRadius)
├── ButtonStyles
│   ├── PrimaryButton (background, text, icon, disabled, pressed, ripple)
│   ├── SecondaryButton (background, text, border, disabled)
│   └── TextButton (text, disabled, ripple)
├── InputStyles
│   ├── TextField (background, text, hint, border, focused, error)
│   └── Dropdown (background, text, icon, border)
├── CardStyles
│   └── Card (background, border, shadow)
└── Typography
    ├── Fonts (heading, body, button)
    └── Sizes (title, subtitle, body, caption)
```

---

### Quick Theming

For simple brand color customization:

```kotlin
Gate2Theme.simple(
    primaryColor = 0xFF1A73E8.toInt(),   // Your brand primary
    secondaryColor = 0xFF34A853.toInt()  // Your brand secondary
)
```

---

### Button Customization

#### Button Corner Radius

```kotlin
val theme = Gate2Theme.builder()
    .shapes(
        Gate2Shapes.builder()
            // Rounded buttons (pill shape)
            .buttonCornerRadius(24.dp)
            // Or square buttons
            // .buttonCornerRadius(0.dp)
            // Or slightly rounded
            // .buttonCornerRadius(8.dp)
            .build()
    )
    .build()
```

#### Button Colors

```kotlin
val theme = Gate2Theme.builder()
    .buttonStyles(
        ButtonStyles.builder()
            // Primary Button (main CTA buttons)
            .primaryButton(
                PrimaryButtonStyle.builder()
                    .backgroundColor(0xFF1A73E8.toInt())      // Button background
                    .backgroundColorPressed(0xFF1557B0.toInt()) // When pressed
                    .backgroundColorDisabled(0xFFE8EAED.toInt()) // When disabled
                    .textColor(0xFFFFFFFF.toInt())            // Button text
                    .textColorDisabled(0xFF9AA0A6.toInt())    // Disabled text
                    .iconColor(0xFFFFFFFF.toInt())            // Icon tint
                    .iconColorDisabled(0xFF9AA0A6.toInt())    // Disabled icon
                    .rippleColor(0x401A73E8.toInt())          // Ripple effect
                    .elevation(4.dp)                           // Shadow elevation
                    .elevationPressed(8.dp)                    // Elevation when pressed
                    .build()
            )
            // Secondary Button (outline buttons)
            .secondaryButton(
                SecondaryButtonStyle.builder()
                    .backgroundColor(0x00000000.toInt())      // Transparent
                    .backgroundColorPressed(0x1A1A73E8.toInt()) // Light fill when pressed
                    .borderColor(0xFF1A73E8.toInt())          // Border color
                    .borderColorDisabled(0xFFE8EAED.toInt())  // Disabled border
                    .borderWidth(1.5.dp)                       // Border thickness
                    .textColor(0xFF1A73E8.toInt())            // Text color
                    .textColorDisabled(0xFF9AA0A6.toInt())    // Disabled text
                    .build()
            )
            // Text Button (minimal buttons)
            .textButton(
                TextButtonStyle.builder()
                    .textColor(0xFF1A73E8.toInt())
                    .textColorPressed(0xFF1557B0.toInt())
                    .textColorDisabled(0xFF9AA0A6.toInt())
                    .rippleColor(0x1A1A73E8.toInt())
                    .build()
            )
            .build()
    )
    .build()
```

#### Button Shapes Reference

| Shape Type | Corner Radius | Example Use Case |
|------------|---------------|------------------|
| **Square** | `0.dp` | Modern, sharp design |
| **Slightly Rounded** | `4.dp` - `8.dp` | Professional, subtle |
| **Rounded** | `12.dp` - `16.dp` | Friendly, approachable |
| **Pill / Capsule** | `24.dp` - `50.dp` | Playful, modern |
| **Circle** | `50%` or large value | Icon buttons |

---

### Input Field Customization

```kotlin
val theme = Gate2Theme.builder()
    .inputStyles(
        InputStyles.builder()
            .textField(
                TextFieldStyle.builder()
                    // Background
                    .backgroundColor(0xFFFFFFFF.toInt())
                    .backgroundColorFocused(0xFFF8F9FA.toInt())
                    .backgroundColorError(0xFFFCE8E6.toInt())
                    .backgroundColorDisabled(0xFFF1F3F4.toInt())
                    // Border
                    .borderColor(0xFFDADCE0.toInt())
                    .borderColorFocused(0xFF1A73E8.toInt())
                    .borderColorError(0xFFEA4335.toInt())
                    .borderColorDisabled(0xFFE8EAED.toInt())
                    .borderWidth(1.dp)
                    .borderWidthFocused(2.dp)
                    // Corner Radius
                    .cornerRadius(8.dp)
                    // Text Colors
                    .textColor(0xFF202124.toInt())
                    .textColorDisabled(0xFF9AA0A6.toInt())
                    .hintColor(0xFF9AA0A6.toInt())
                    .labelColor(0xFF5F6368.toInt())
                    .labelColorFocused(0xFF1A73E8.toInt())
                    .labelColorError(0xFFEA4335.toInt())
                    // Helper & Error Text
                    .helperTextColor(0xFF5F6368.toInt())
                    .errorTextColor(0xFFEA4335.toInt())
                    // Cursor
                    .cursorColor(0xFF1A73E8.toInt())
                    // Selection
                    .selectionHandleColor(0xFF1A73E8.toInt())
                    .selectionBackgroundColor(0x401A73E8.toInt())
                    .build()
            )
            .dropdown(
                DropdownStyle.builder()
                    .backgroundColor(0xFFFFFFFF.toInt())
                    .textColor(0xFF202124.toInt())
                    .hintColor(0xFF9AA0A6.toInt())
                    .iconColor(0xFF5F6368.toInt())
                    .borderColor(0xFFDADCE0.toInt())
                    .borderColorFocused(0xFF1A73E8.toInt())
                    .cornerRadius(8.dp)
                    // Dropdown Menu
                    .menuBackgroundColor(0xFFFFFFFF.toInt())
                    .menuItemTextColor(0xFF202124.toInt())
                    .menuItemSelectedBackgroundColor(0xFFE8F0FE.toInt())
                    .menuItemHoverBackgroundColor(0xFFF8F9FA.toInt())
                    .menuElevation(8.dp)
                    .menuCornerRadius(8.dp)
                    .build()
            )
            .build()
    )
    .build()
```

---

### Card Customization

```kotlin
val theme = Gate2Theme.builder()
    .cardStyles(
        CardStyles.builder()
            .card(
                CardStyle.builder()
                    // Background
                    .backgroundColor(0xFFFFFFFF.toInt())
                    .backgroundColorPressed(0xFFF8F9FA.toInt())
                    .backgroundColorSelected(0xFFE8F0FE.toInt())
                    // Border
                    .borderColor(0xFFE8EAED.toInt())
                    .borderColorSelected(0xFF1A73E8.toInt())
                    .borderWidth(1.dp)
                    // Corner Radius
                    .cornerRadius(12.dp)
                    // Elevation & Shadow
                    .elevation(2.dp)
                    .elevationPressed(4.dp)
                    .shadowColor(0x1A000000.toInt())
                    // Ripple
                    .rippleColor(0x1A000000.toInt())
                    .build()
            )
            // Flight/Hotel/Plan Result Cards
            .resultCard(
                ResultCardStyle.builder()
                    .backgroundColor(0xFFFFFFFF.toInt())
                    .borderColor(0xFFE8EAED.toInt())
                    .cornerRadius(16.dp)
                    .elevation(4.dp)
                    // Price Badge
                    .priceBadgeBackgroundColor(0xFF1A73E8.toInt())
                    .priceBadgeTextColor(0xFFFFFFFF.toInt())
                    .priceBadgeCornerRadius(8.dp)
                    .build()
            )
            .build()
    )
    .build()
```

---

### Shape Configuration (Global Corner Radius)

Configure corner radius for all components at once:

```kotlin
val theme = Gate2Theme.builder()
    .shapes(
        Gate2Shapes.builder()
            // Buttons
            .buttonCornerRadius(12.dp)
            .buttonBorderWidth(1.5.dp)
            // Cards
            .cardCornerRadius(16.dp)
            .cardElevation(4.dp)
            // Input Fields
            .inputCornerRadius(8.dp)
            .inputBorderWidth(1.dp)
            // Dialogs
            .dialogCornerRadius(24.dp)
            // Bottom Sheets
            .bottomSheetCornerRadius(24.dp)
            // Chips/Tags
            .chipCornerRadius(16.dp)
            // Images
            .imageCornerRadius(8.dp)
            // Snackbar
            .snackbarCornerRadius(8.dp)
            .build()
    )
    .build()
```

#### Preset Shape Styles

```kotlin
// Sharp/Square Design
Gate2Shapes.sharp()  // All corners = 0.dp

// Slightly Rounded (Default)
Gate2Shapes.rounded()  // Subtle rounding

// Very Rounded (Friendly)
Gate2Shapes.veryRounded()  // More pronounced rounding

// Pill Style
Gate2Shapes.pill()  // Maximum rounding for buttons
```

---

### Color Customization

#### Primary & Secondary Colors

```kotlin
val theme = Gate2Theme.builder()
    .primaryColors(
        PrimaryColors.builder()
            .main(0xFF1A73E8.toInt())        // Main brand color
            .dark(0xFF1557B0.toInt())        // Darker variant (pressed states)
            .light(0xFF4A90D9.toInt())       // Lighter variant (backgrounds)
            .onPrimary(0xFFFFFFFF.toInt())   // Text/icons on primary color
            .build()
    )
    .secondaryColors(
        SecondaryColors.builder()
            .main(0xFF34A853.toInt())        // Secondary brand color
            .dark(0xFF2E7D4A.toInt())        // Darker variant
            .light(0xFF5CBC6E.toInt())       // Lighter variant
            .onSecondary(0xFFFFFFFF.toInt()) // Text/icons on secondary color
            .build()
    )
    .build()
```

#### Background Colors

```kotlin
val theme = Gate2Theme.builder()
    .backgroundColors(
        BackgroundColors.builder()
            .background(0xFFFFFFFF.toInt())   // Main screen background
            .surface(0xFFF8F9FA.toInt())      // Elevated surface background
            .card(0xFFFFFFFF.toInt())         // Card background
            .divider(0xFFE8EAED.toInt())      // Divider lines
            .border(0xFFDADCE0.toInt())       // Border color
            .overlay(0x80000000.toInt())      // Modal overlay
            .scrim(0x52000000.toInt())        // Bottom sheet scrim
            .build()
    )
    .build()
```

#### Text Colors

```kotlin
val theme = Gate2Theme.builder()
    .textColors(
        TextColors.builder()
            .primary(0xFF202124.toInt())      // Main text
            .secondary(0xFF5F6368.toInt())    // Secondary text
            .tertiary(0xFF80868B.toInt())     // Tertiary/caption text
            .hint(0xFF9AA0A6.toInt())         // Placeholder text
            .disabled(0xFFBDC1C6.toInt())     // Disabled text
            .link(0xFF1A73E8.toInt())         // Clickable links
            .linkVisited(0xFF7B1FA2.toInt())  // Visited links
            .onDark(0xFFFFFFFF.toInt())       // Text on dark backgrounds
            .onLight(0xFF202124.toInt())      // Text on light backgrounds
            .build()
    )
    .build()
```

#### Status Colors

```kotlin
val theme = Gate2Theme.builder()
    .statusColors(
        StatusColors.builder()
            // Success (green)
            .success(0xFF34A853.toInt())
            .successLight(0xFFE6F4EA.toInt())
            .onSuccess(0xFFFFFFFF.toInt())
            // Warning (yellow/amber)
            .warning(0xFFFBBC04.toInt())
            .warningLight(0xFFFEF7E0.toInt())
            .onWarning(0xFF202124.toInt())
            // Error (red)
            .error(0xFFEA4335.toInt())
            .errorLight(0xFFFCE8E6.toInt())
            .onError(0xFFFFFFFF.toInt())
            // Info (blue)
            .info(0xFF4285F4.toInt())
            .infoLight(0xFFE8F0FE.toInt())
            .onInfo(0xFFFFFFFF.toInt())
            .build()
    )
    .build()
```

---

### Icon Customization

```kotlin
val theme = Gate2Theme.builder()
    .iconColors(
        IconColors.builder()
            .primary(0xFF5F6368.toInt())      // Default icon color
            .secondary(0xFF9AA0A6.toInt())    // Secondary icons
            .active(0xFF1A73E8.toInt())       // Active/selected icons
            .disabled(0xFFBDC1C6.toInt())     // Disabled icons
            .onPrimary(0xFFFFFFFF.toInt())    // Icons on primary background
            .onSurface(0xFF5F6368.toInt())    // Icons on surface
            .build()
    )
    .build()
```

---

### Typography Customization

```kotlin
val theme = Gate2Theme.builder()
    .typography(
        Gate2Typography.builder()
            // Font Families
            .headingFontFamily(FontFamily.SansSerif)
            .bodyFontFamily(FontFamily.SansSerif)
            .buttonFontFamily(FontFamily.SansSerif)
            // Or custom fonts:
            // .headingFontFamily(FontFamily(Font(R.font.your_heading_font)))

            // Font Sizes
            .displayLarge(32.sp)
            .displayMedium(28.sp)
            .headlineLarge(24.sp)
            .headlineMedium(20.sp)
            .titleLarge(18.sp)
            .titleMedium(16.sp)
            .bodyLarge(16.sp)
            .bodyMedium(14.sp)
            .bodySmall(12.sp)
            .labelLarge(14.sp)
            .labelMedium(12.sp)
            .labelSmall(10.sp)

            // Font Weights
            .headingWeight(FontWeight.Bold)
            .titleWeight(FontWeight.SemiBold)
            .bodyWeight(FontWeight.Normal)
            .buttonWeight(FontWeight.Medium)

            // Letter Spacing
            .headingLetterSpacing((-0.5).sp)
            .bodyLetterSpacing(0.sp)
            .buttonLetterSpacing(0.5.sp)

            // Line Heights
            .headingLineHeight(1.2.em)
            .bodyLineHeight(1.5.em)
            .build()
    )
    .build()
```

---

### Complete Theme Example

Here's a complete example with all customization options:

```kotlin
val myBrandTheme = Gate2Theme.builder()
    // Colors
    .primaryColors(
        PrimaryColors.builder()
            .main(0xFF6200EE.toInt())         // Purple brand
            .dark(0xFF3700B3.toInt())
            .light(0xFFBB86FC.toInt())
            .onPrimary(0xFFFFFFFF.toInt())
            .build()
    )
    .secondaryColors(
        SecondaryColors.builder()
            .main(0xFF03DAC6.toInt())         // Teal accent
            .dark(0xFF018786.toInt())
            .light(0xFF66FFF9.toInt())
            .onSecondary(0xFF000000.toInt())
            .build()
    )
    .backgroundColors(
        BackgroundColors.builder()
            .background(0xFFFAFAFA.toInt())
            .surface(0xFFFFFFFF.toInt())
            .card(0xFFFFFFFF.toInt())
            .divider(0xFFE0E0E0.toInt())
            .border(0xFFBDBDBD.toInt())
            .build()
    )
    .textColors(
        TextColors.builder()
            .primary(0xFF212121.toInt())
            .secondary(0xFF757575.toInt())
            .hint(0xFF9E9E9E.toInt())
            .disabled(0xFFBDBDBD.toInt())
            .link(0xFF6200EE.toInt())
            .build()
    )
    .statusColors(
        StatusColors.builder()
            .success(0xFF4CAF50.toInt())
            .warning(0xFFFF9800.toInt())
            .error(0xFFF44336.toInt())
            .info(0xFF2196F3.toInt())
            .build()
    )
    // Shapes
    .shapes(
        Gate2Shapes.builder()
            .buttonCornerRadius(24.dp)        // Pill-shaped buttons
            .cardCornerRadius(16.dp)          // Rounded cards
            .inputCornerRadius(12.dp)         // Rounded inputs
            .dialogCornerRadius(28.dp)        // Rounded dialogs
            .build()
    )
    // Button Styles
    .buttonStyles(
        ButtonStyles.builder()
            .primaryButton(
                PrimaryButtonStyle.builder()
                    .backgroundColor(0xFF6200EE.toInt())
                    .backgroundColorPressed(0xFF3700B3.toInt())
                    .textColor(0xFFFFFFFF.toInt())
                    .elevation(6.dp)
                    .build()
            )
            .secondaryButton(
                SecondaryButtonStyle.builder()
                    .backgroundColor(0x00000000.toInt())
                    .borderColor(0xFF6200EE.toInt())
                    .borderWidth(2.dp)
                    .textColor(0xFF6200EE.toInt())
                    .build()
            )
            .build()
    )
    // Input Styles
    .inputStyles(
        InputStyles.builder()
            .textField(
                TextFieldStyle.builder()
                    .backgroundColor(0xFFFFFFFF.toInt())
                    .borderColor(0xFFBDBDBD.toInt())
                    .borderColorFocused(0xFF6200EE.toInt())
                    .cornerRadius(12.dp)
                    .cursorColor(0xFF6200EE.toInt())
                    .build()
            )
            .build()
    )
    // Card Styles
    .cardStyles(
        CardStyles.builder()
            .card(
                CardStyle.builder()
                    .backgroundColor(0xFFFFFFFF.toInt())
                    .cornerRadius(16.dp)
                    .elevation(4.dp)
                    .borderColor(0xFFE0E0E0.toInt())
                    .build()
            )
            .build()
    )
    .build()

// Apply to SDK
setContent {
    Gate2TravelTheme(theme = myBrandTheme) {
        Gate2TravelScreen(
            onComplete = { finish() },
            onCancel = { finish() },
            onFail = { error -> showError(error) },
            onProcessPayment = { request -> processPayment(request) }
        )
    }
}
```

---

### Color Categories Reference

| Category | Properties | Usage |
|----------|------------|-------|
| **Primary** | main, dark, light, onPrimary | Buttons, headers, active states |
| **Secondary** | main, dark, light, onSecondary | Accents, highlights |
| **Background** | background, surface, card, divider, border | Screens, cards |
| **Text** | primary, secondary, hint, disabled, link | All text content |
| **Status** | success, warning, error, info | Feedback indicators |
| **Button** | backgroundColor, textColor, borderColor | All button types |
| **Input** | backgroundColor, borderColor, textColor | Form fields |
| **Card** | backgroundColor, borderColor, elevation | Cards and panels |
| **Icon** | primary, active, disabled | All icons |

### Shape Reference

| Component | Property | Default | Description |
|-----------|----------|---------|-------------|
| **Button** | cornerRadius | 12.dp | Button corner rounding |
| **Card** | cornerRadius | 12.dp | Card corner rounding |
| **Input** | cornerRadius | 8.dp | Input field corner rounding |
| **Dialog** | cornerRadius | 24.dp | Dialog corner rounding |
| **Chip** | cornerRadius | 16.dp | Tag/chip corner rounding |
| **Image** | cornerRadius | 8.dp | Image corner rounding |

---

## 4.4 Payment Integration

### Payment Modes

All three SDKs support the same payment integration patterns:

```
Payment Required
├── Mode 1: Host Payment Processing (Recommended)
│   └── You handle payment with your provider
└── Mode 2: Internal Payment (Demo/Test)
    └── SDK handles everything
```

### Host Payment Processing

```kotlin
// Works the same for all three SDKs
onProcessPayment = { paymentRequest ->
    yourPaymentProvider.charge(
        amount = paymentRequest.amount,
        currency = paymentRequest.currency,
        orderId = paymentRequest.orderId
    ) { result ->
        when (result) {
            is Success -> {
                // Notify SDK (use appropriate SDK)
                Gate2TravelSdk.notifyPaymentResult(
                    PaymentResult.Success(
                        transactionId = result.transactionId,
                        authorizationCode = result.authCode
                    )
                )
            }
            is Failed -> {
                Gate2TravelSdk.notifyPaymentResult(
                    PaymentResult.Failure(
                        errorCode = result.code,
                        message = result.message,
                        canRetry = true
                    )
                )
            }
            is Cancelled -> {
                Gate2TravelSdk.notifyPaymentResult(PaymentResult.Cancelled)
            }
        }
    }
}
```

### Payment Flow

```
┌────────────────────────────────────────────────────────────────┐
│                     PAYMENT FLOW                                │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ┌─────────┐    ┌─────────────┐    ┌─────────────┐           │
│  │   SDK   │───▶│  Host App   │───▶│  Payment    │           │
│  │         │    │             │    │  Provider   │           │
│  └─────────┘    └─────────────┘    └─────────────┘           │
│       │              │                    │                   │
│       │              │                    │                   │
│       │         onProcessPayment          │                   │
│       │◀─────────────┘                    │                   │
│       │                                   │                   │
│       │              notifyPaymentResult  │                   │
│       │◀──────────────────────────────────┘                   │
│       │                                                       │
│       ▼                                                       │
│  ┌─────────┐                                                  │
│  │  Show   │                                                  │
│  │ Result  │                                                  │
│  └─────────┘                                                  │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### PCI DSS Compliance

| Aspect | Implementation |
|--------|----------------|
| **Card Storage** | Never stored |
| **Transmission** | TLS 1.3 encrypted |
| **Logging** | Card data never logged |
| **Memory** | Cleared after use |

---

## 4.5 Error Handling

### Error Handling Pattern

All SDKs use the same error callback pattern:

```kotlin
onFail = { errorMessage ->
    // errorMessage is user-friendly and safe to display
    showErrorDialog(errorMessage)
}
```

### Common Error Scenarios

| Scenario | User Message Example | Recommended Action |
|----------|---------------------|-------------------|
| No internet | "Please check your internet connection" | Show retry option |
| Invalid API key | "Authentication failed" | Check configuration |
| Item unavailable | "This item is no longer available" | Return to search |
| Price changed | "The price has changed" | Show updated price |
| Payment declined | "Payment was declined" | Offer retry |
| Server error | "Something went wrong. Please try again" | Show retry option |

### Best Practice

```kotlin
onFail = { errorMessage ->
    // Log in debug builds
    if (BuildConfig.DEBUG) {
        Log.e("Gate2SDK", "Error: $errorMessage")
    }

    // Show user-friendly dialog with options
    AlertDialog.Builder(context)
        .setTitle("Error")
        .setMessage(errorMessage)
        .setPositiveButton("Try Again") { _, _ -> /* retry */ }
        .setNegativeButton("Cancel") { _, _ -> finish() }
        .show()
}
```

---

## 4.6 Network & Security

### Security Architecture

```
┌────────────────────────────────────────────────────────────────┐
│                    SECURITY ARCHITECTURE                        │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐       │
│  │   TLS 1.3   │    │ Certificate │    │  API Key    │       │
│  │  Encryption │    │   Pinning   │    │   Auth      │       │
│  └─────────────┘    └─────────────┘    └─────────────┘       │
│                                                                │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐       │
│  │   No Card   │    │  Sensitive  │    │   Secure    │       │
│  │   Storage   │    │  Data Redact│    │   Memory    │       │
│  └─────────────┘    └─────────────┘    └─────────────┘       │
│                                                                │
└────────────────────────────────────────────────────────────────┘
```

### Base Configuration

| Setting | Value |
|:--------|:------|
| **Base URL** | `https://api.gate2.travel/` |
| **Protocol** | HTTPS only (TLS 1.3) |
| **Authentication** | API Key header |

### API Headers

```
X-API-Key: your-api-key-here
X-SDK-Version: 1.0.0
X-Platform: Android
Content-Type: application/json
Accept: application/json
```

### Network Timeouts

| Timeout | Value |
|---------|-------|
| Connect | 30 seconds |
| Read | 30 seconds |
| Write | 30 seconds |

---

## 4.7 Troubleshooting

### Quick Diagnostic Checklist

- [ ] SDK initialized in `Application.onCreate()`
- [ ] Valid API key configured
- [ ] Internet connection available
- [ ] Using appropriate Theme wrapper
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
        Gate2EsimSdk.initialize(/*...*/)
        Gate2HotelSdk.initialize(/*...*/)
    }
}

// Register in AndroidManifest.xml
<application android:name=".MyApplication">
```

#### Theme Not Applied

```kotlin
// Ensure theme wrapper is used
Gate2TravelTheme {  // or Gate2EsimTheme or Gate2HotelTheme
    Gate2TravelScreen(/*...*/)
}
```

#### Debug Logging

```kotlin
Gate2SdkConfig.builder()
    .enableLogging(BuildConfig.DEBUG)
    .build()
```

---

## 4.8 Support Contacts

| Resource | Contact |
|----------|---------|
| **SDK Documentation** | [GitHub Page](https://github.com/gate2-travel/demo-android/blob/main/README.md) |
| **API Documentation** | [Postman Collection](https://documenter.getpostman.com/view/7713462/2sB3dWq6UM) |
| **Developer Portal** | [developers.gate2.travel](https://developers.gate2.travel) |
| **GitHub Issues** | [github.com/gate2-travel/demo-android](https://github.com/gate2-travel/demo-android) |
| **Email Support** | [support@gate2.travel](mailto:support@gate2.travel) |

---

## Glossary

### Travel Terms
| Term | Definition |
|------|------------|
| **PNR** | Passenger Name Record - unique booking reference |
| **IATA** | International Air Transport Association |
| **GDS** | Global Distribution System |

### eSIM Terms
| Term | Definition |
|------|------------|
| **eSIM** | Embedded SIM - digital SIM without physical card |
| **SM-DP+** | Subscription Manager for eSIM profiles |
| **QR Code** | Quick Response code for eSIM activation |
| **EID** | eSIM Identifier |

### Hotel Terms
| Term | Definition |
|------|------------|
| **ADR** | Average Daily Rate |
| **BAR** | Best Available Rate |
| **OTA** | Online Travel Agency |

---

## ISO 4217 Currency Codes

| Code | Currency |
|------|----------|
| USD | US Dollar |
| EUR | Euro |
| GBP | British Pound |
| JPY | Japanese Yen |
| AED | UAE Dirham |
| SGD | Singapore Dollar |
| AUD | Australian Dollar |
| CAD | Canadian Dollar |

---

## Changelog

### Version 1.0.0 (January 2026)
- Initial release of Gate2 SDK Suite
- Travel SDK: Flight booking functionality
- eSIM SDK: Data plan purchasing and activation
- Hotel SDK: Hotel booking functionality
- Common theming system
- Flexible payment integration
- Certificate pinning
- Debug logging support

---

<div align="center">

### Ready to Get Started?

| Travel SDK | eSIM SDK | Hotel SDK |
|:---:|:---:|:---:|
| [Quick Start](#12-travel-quick-start) | [Quick Start](#22-esim-quick-start) | [Quick Start](#32-hotel-quick-start) |

---

**Document Version**: 1.0.0 | **SDK Version**: 1.0.0 | **Last Updated**: January 22, 2026

---

**Gate2 SDK Suite** - Complete Travel Services Solution for Android

[GitHub](https://github.com/gate2-travel/demo-android) | [Documentation](https://docs.gate2.travel) | [Support](mailto:support@gate2.travel)

© 2026 Gate2 Travel. All rights reserved.

</div>
