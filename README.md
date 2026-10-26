<div align="center">

# Gate2 Travel SDK for Android

### eSIM data plans, flight booking and hotel booking inside your own app

[![SDK Version](https://img.shields.io/badge/SDK-1.2.1-blue.svg)](https://central.sonatype.com/search?q=travel.gate2)
[![Platform](https://img.shields.io/badge/Platform-Android-green.svg)](https://developer.android.com)
[![Min SDK](https://img.shields.io/badge/minSdk-24-orange.svg)](https://developer.android.com)
[![Kotlin](https://img.shields.io/badge/Kotlin-2.2.21-purple.svg)](https://kotlinlang.org)
[![Maven Central](https://img.shields.io/badge/Maven%20Central-travel.gate2-teal.svg)](https://central.sonatype.com/search?q=travel.gate2)

**One dependency. One `initialize()`. One call per product.**

| [eSIM](#5-esim) | [Flights](#6-flights) | [Hotels](#7-hotels) | [Showcase](#8-showcase-screen-travel-all) |
|:---:|:---:|:---:|:---:|
| Data plans + activation | Search, seats, booking | Search, rooms, booking | Drop-in product picker |

</div>

---

> **Document version:** 2.0 · **SDK version:** 1.2.1 · **Last updated:** 2026-10-26
> **Breaking changes since the 1.0.x docs:** new Maven coordinates, a shared `core`, Activity-based
> entry points (`Gate2Esim` / `Gate2Flight` / `Gate2Hotel`) and `resumeAfterPayment()` instead of
> `notifyPaymentResult()`. See [§15 Migrating from 1.0.x](#15-migrating-from-10x).

---

## Table of Contents

1. [Overview](#1-overview)
2. [Requirements](#2-requirements)
3. [Installation](#3-installation)
4. [Initialize the SDK](#4-initialize-the-sdk)
5. [eSIM](#5-esim)
6. [Flights](#6-flights)
7. [Hotels](#7-hotels)
8. [Showcase screen (travel-all)](#8-showcase-screen-travel-all)
9. [Payments](#9-payments)
10. [Back navigation](#10-back-navigation)
11. [Theming and branding](#11-theming-and-branding)
12. [Logging](#12-logging)
13. [Network and security](#13-network-and-security)
14. [Troubleshooting](#14-troubleshooting)
15. [Migrating from 1.0.x](#15-migrating-from-10x)
16. [API summary](#16-api-summary)
17. [Changelog](#17-changelog)
18. [Support](#18-support)

---

## 1. Overview

The Gate2 Travel SDK is a set of Android libraries that put complete, ready-made travel flows inside
a host app. Each product renders its own Jetpack Compose UI in its own `Activity`, talks to the Gate2
backend over a pinned mTLS connection, and hands control back to the host through callbacks. The
host never handles card data: payment is delegated to the host's own provider and the SDK resumes
afterwards.

| Product | Artifact | Entry point | What the user gets |
|:--|:--|:--|:--|
| **eSIM** | `esim-<company>` | `Gate2Esim` | Destination search, plan list, order, eSIM installation, "My eSIMs", top-ups |
| **Flights** | `flights-<company>` | `Gate2Flight` | Search, results, fare details, travelers, seat map, review, confirmation, "My flights", saved travelers, FAQ |
| **Hotels** | `hotels-<company>` | `Gate2Hotel` | Search, results, hotel detail, room pre-booking, guest details, confirmation |
| **All three** | `travel-all-<company>` | the three above + `Gate2TravelShowcase` | Everything, plus a customizable product picker with a "My cabinet" area |

All products share one `core` artifact (`core-<company>`) that owns initialization, networking,
theming and logging. It arrives transitively: **never declare `core` yourself.**

### Integration at a glance

```
┌──────────────────────────────────────────────────────────────────────────┐
│                              YOUR APPLICATION                            │
│                                                                          │
│  Application.onCreate()            Anywhere in your UI                   │
│  ┌──────────────────────┐          ┌────────────────────────────────┐    │
│  │ Gate2TravelSdk       │          │ Gate2Esim.start(...)           │    │
│  │   .initialize(config)│  once    │ Gate2Flight.start(...)         │    │
│  └──────────────────────┘          │ Gate2Hotel.start(...)          │    │
│                                    │ Gate2TravelShowcase { ... }    │    │
│                                    └──────────────┬─────────────────┘    │
│                                                   │ launches             │
│                                    ┌──────────────▼─────────────────┐    │
│  ┌──────────────────────┐          │   SDK Activity (own UI)        │    │
│  │ Your callbacks       │◀─────────│   • full product flow          │    │
│  │  onComplete          │          │   • creates the order          │    │
│  │  onCancel            │          └──────────────┬─────────────────┘    │
│  │  onFail              │                         │ onProcessPayment     │
│  │  onProcessPayment ───┼──▶ your payment UI ──▶ resumeAfterPayment()   │
│  └──────────────────────┘                                                │
└──────────────────────────────────────────────────────────────────────────┘
```

### Who does what

| SDK | Host app |
|:--|:--|
| All product screens, state and validation | Call `initialize()` once with the Gate2 access token |
| Backend calls, mTLS, certificate pinning, retries | Provide a `sessionId` per flow and (ideally) a `userId` |
| Order creation and confirmation | Charge the customer with its own payment provider |
| eSIM provisioning on the device | Persist `orderId` + `sessionId` and call `resumeAfterPayment()` |
| Brand look (compiled into the brand artifact) | Decide where the product entry points live |

---

## 2. Requirements

| | Requirement | Notes |
|:--|:--|:--|
| **minSdk** | 24 | The library's declared floor. Gate2 supports Android 8.1 (API 27) and newer. |
| **eSIM provisioning** | API 28+ | `EuiccManager` does not exist below API 28. Gate the entry point with `Gate2Esim.isDeviceEsimCapable()`. |
| **compileSdk** | 35 or newer | The SDK is compiled against API 35. |
| **Kotlin** | 2.2.21 | |
| **JVM target** | 11 | |
| **Jetpack Compose** | Not required | Each product renders inside its own `Activity`. You need Compose in your app only if you embed `Gate2TravelShowcase`. |
| **Google Play Services** | Not required | |
| **Access token** | Issued by Gate2 | A JWT sent as `X-API-Key`. Product entitlement is enforced server-side. |

### Permissions merged from the SDK manifests

| Permission | Declared by | Why |
|:--|:--|:--|
| `INTERNET` | all modules | Backend calls |
| `READ_PHONE_STATE` / `READ_BASIC_PHONE_STATE` | eSIM | Detect eUICC capability and existing subscriptions. The eSIM Activity requests this at runtime. |
| `WRITE_SETTINGS` | eSIM | Required by the system eSIM activation flow |
| `USE_CREDENTIALS` | eSIM | eUICC subscription provisioning |

Each product declares one non-exported `Activity` (`EsimActivity`, `FlightActivity`,
`HotelActivity`). No launcher entries, services, receivers or content providers.

---

## 3. Installation

Artifacts are published to **Maven Central** under the group `travel.gate2`. Brand and backend
environment are part of the **artifact ID**, so the version is always plain semver:

```
travel.gate2:<module>-<company>[-staging]:<version>
```

| Part | Values | Meaning |
|:--|:--|:--|
| `<module>` | `travel-all`, `esim`, `flights`, `hotels` | What you integrate (`core` is transitive) |
| `<company>` | assigned by Gate2 (one short lowercase word) | Your brand build: colours, welcome screens, fonts and the backend credentials are compiled in |
| `-staging` | optional suffix | Points at the staging backend (`api.gate2.travel`). Omit it for production (`gw.gate2.travel`). |
| `<version>` | e.g. `1.2.1` | Plain semver. **Never** pin `1.2.1-<company>` or `1.0.20-test`; that scheme is retired. |

```kotlin
// app/build.gradle.kts
dependencies {
    // Everything (eSIM + Flights + Hotels + showcase) over one shared core:
    implementation("travel.gate2:travel-all-<company>:1.2.1")

    // ...or a single product. Each pulls its matching core automatically:
    // implementation("travel.gate2:esim-<company>:1.2.1")
    // implementation("travel.gate2:flights-<company>:1.2.1")
    // implementation("travel.gate2:hotels-<company>:1.2.1")
}
```

```kotlin
// settings.gradle.kts
dependencyResolutionManagement {
    repositories {
        google()
        mavenCentral()
    }
}
```

> **Exactly one `<company>` per app.** Two brand artifacts on the classpath produce
> `Duplicate class com.gate2.sdk...` at build time.
>
> **Staging builds** carry the same version number as production. Check the version exists on
> Maven Central before pinning it, and ask Gate2 if it is missing.

### ProGuard / R8

Nothing to add. The AARs ship unminified with their own `consumer-rules.pro`; your app's R8 pass
obfuscates the SDK together with your own code. Do not add manual `-keep class com.gate2.sdk.**`
rules, they only reduce your own obfuscation coverage.

---

## 4. Initialize the SDK

Call this once, before the first product call. `Application.onCreate()` is the natural place.

```kotlin
import com.gate2.sdk.core.Gate2TravelSdk
import com.gate2.sdk.core.config.Gate2SdkConfig
import com.gate2.sdk.core.config.SdkLogLevel

class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        if (Gate2TravelSdk.isInitialized()) return   // initialize() throws if called twice

        Gate2TravelSdk.initialize(
            Gate2SdkConfig.builder()
                .context(this)
                .apiKey(BuildConfig.GATE2_ACCESS_TOKEN)   // inject from a secret store, never commit it
                .logLevel(if (BuildConfig.DEBUG) SdkLogLevel.DEBUG else SdkLogLevel.NONE)
                .build()
        )
    }
}
```

### `Gate2SdkConfig.Builder`

| Method | Required | Notes |
|:--|:--:|:--|
| `.context(Context)` | yes | Stored as `applicationContext` |
| `.apiKey(String)` | yes | Your Gate2 token. `build()` throws `IllegalArgumentException` if blank. |
| `.logLevel(SdkLogLevel)` | no | `ALL`, `DEBUG`, `INFO`, `WARN`, `ERROR`, `FATAL`, `NONE` (default `NONE`) |
| `.enableLogging(Boolean)` | no | Shorthand: `true` is `DEBUG`, `false` is `NONE` |
| `.onLog { entry -> }` | no | Receive every `SdkLogEntry`; see [§12](#12-logging) |
| `.theme(Gate2Theme)` | no | App-wide theme override; see [§11](#11-theming-and-branding) |
| `.clientCertificate(...)` | deprecated | mTLS is embedded in the SDK. Remove any call. |

### What can and cannot fail here

- `initialize()` throws `IllegalStateException` **only** when the SDK is already initialized. Guard with `isInitialized()`.
- The token is **not** validated on the client. A wrong or expired token initializes cleanly and
  surfaces later as an `UNAUTHORIZED` (HTTP 401) failure in the product callbacks. Handle it there,
  not with a `try/catch` around `initialize()`.
- `Gate2TravelSdk.getInstance().getVersion()` returns the running SDK version (staging builds report
  `1.2.1-test`). Include it in support requests.

---

## 5. eSIM

> For a complete, production-ready walkthrough (persisting payment state, error mapping,
> pre-release checklist) see **[ESIM_INTEGRATION_GUIDE.md](./ESIM_INTEGRATION_GUIDE.md)**.

### Launch

```kotlin
import com.gate2.sdk.esim.Gate2Esim
import com.gate2.sdk.esim.EsimCallbacks
import com.gate2.sdk.esim.api.EsimError

Gate2Esim.start(
    activity  = this,
    sessionId = sessionId,          // required, non-blank; one per purchase attempt (a UUID is fine)
    callbacks = esimCallbacks,
    userId    = currentUser.id,     // optional, strongly recommended
    language  = "en",               // "en", "az", "ru"
    title     = "eSIM"              // toolbar title
)
```

| Parameter | Default | Notes |
|:--|:--|:--|
| `activity: Activity` | — | Launch context |
| `sessionId: String` | — | Required. Sent as `X-Session-Id`; keep the **same** value for `resumeAfterPayment()` |
| `callbacks: EsimCallbacks` | — | See below |
| `userId: String?` | `null` | Sent as `X-User-Id`; scopes orders and "My eSIMs" |
| `language: String` | `"en"` | UI language. The eSIM module ships `en`, `az`, `ru` strings. |
| `accentColor: Int` | brand colour | ARGB override for this launch |
| `fontResId: Int` | `0` | `@FontRes` override (0 = brand/Inter) |
| `title: String` | `"eSIM"` | Toolbar title |
| `esimLabel: String?` | `null` | Best-effort display name for the installed subscription |

`start()` never throws. If the SDK is uninitialized, the token has no eSIM entitlement, or
`sessionId` is blank, it calls `onFail` and returns.

### Callbacks

```kotlin
val esimCallbacks = EsimCallbacks(
    onComplete = { /* order completed, eSIM installed */ },
    onCancel   = { /* user backed out. Do NOT finish() your Activity here, see §10 */ },
    onFail     = { error: EsimError -> handleError(error) },
    onProcessPayment = { request -> /* charge the customer, then resumeAfterPayment(); see §9 */ },
    onApiResponse = { statusCode, errorMessage -> /* optional: every HTTP response */ }
)
```

> **Annotate the `onFail` parameter** (`{ error: EsimError -> ... }`). `EsimCallbacks` is both a data
> class (whose `onFail` takes `EsimError`) and a same-named function in the same package (whose
> `onFail` takes a `String`). An un-annotated lambda is ambiguous between the two.

### `EsimError`

```kotlin
data class EsimError(
    val code: EsimErrorCode,
    val message: String,          // display-ready
    val cause: Throwable? = null,
    val isRetryable: Boolean = false,
    val httpStatusCode: Int? = null
)
```

| Group | `EsimErrorCode` values |
|:--|:--|
| SDK state | `SDK_NOT_INITIALIZED`, `MODULE_ACCESS_DENIED`, `DEVICE_NOT_SUPPORTED` |
| Validation | `INVALID_SESSION_ID`, `INVALID_ORDER_ID`, `INVALID_USER_ID`, `VALIDATION_ERROR` |
| Network | `NETWORK_ERROR` |
| HTTP | `UNAUTHORIZED`, `FORBIDDEN`, `NOT_FOUND`, `RATE_LIMITED`, `SERVER_ERROR` |
| Order | `NO_PRODUCT_SELECTED`, `ORDER_CREATION_FAILED`, `PAYMENT_FAILED`, `ORDER_CONFIRMATION_FAILED` |
| Installation | `ESIM_INSTALLATION_FAILED`, `ESIM_EUICC_DISABLED` |
| Other | `PARSE_ERROR`, `UNKNOWN` |

### Payment request

```kotlin
data class EsimPaymentRequest(
    val orderId: String,        // persist this
    val sessionId: String,      // persist this
    val userId: String,
    val productId: String,
    val productName: String,
    val amount: Double,
    val currency: String,       // ISO 4217, e.g. "AZN"
    val dataLimit: String,      // "5GB", "Unlimited"
    val validityDays: Int,
    val coverage: List<String>,
    val isTopUp: Boolean = false,
    val iccid: String? = null   // set for top-up orders
)
```

After a successful charge:

```kotlin
Gate2Esim.resumeAfterPayment(
    activity  = this,
    orderId   = storedOrderId,
    sessionId = storedSessionId,       // the same value you passed to start()
    callbacks = EsimCallbacks(
        onComplete = { clearPaymentState() },
        onCancel   = { clearPaymentState() },
        onFail     = { error: EsimError -> clearPaymentState(); handleError(error) }
    ),
    language = "en"
)
```

### "My eSIMs"

```kotlin
Gate2Esim.showMyEsims(
    activity  = this,
    sessionId = sessionId,
    callbacks = esimCallbacks,
    userId    = currentUser.id,   // REQUIRED here: the list is scoped to a user
    language  = "en"
)
```

### Gate the entry point

```kotlin
val deviceOk = Gate2Esim.isDeviceEsimCapable(context)   // false below API 28 or with eUICC disabled
val moduleOk = Gate2Esim.isAvailable()                   // SDK initialized + eSIM module registered

if (deviceOk && moduleOk) showEsimEntryPoint()
```

`isAvailable()` reports SDK state, not entitlement. An unentitled token fails later with an HTTP error.

### Flow

```
Welcome (T&C, first launch only) → Search destination → Plans → Order → [host payment] → Confirmation → Install
                                                                                         ↘ My eSIMs → Detail → Top-up
```

The search screen can show a merchant banner configured by Gate2 for your brand; it is hidden
silently when none is configured.

---

## 6. Flights

### Launch

```kotlin
import com.gate2.sdk.flights.Gate2Flight
import com.gate2.sdk.flights.FlightCallbacks

Gate2Flight.start(
    activity  = this,
    sessionId = sessionId,
    callbacks = flightCallbacks,
    userId    = currentUser.id,   // optional; required for saved travelers
    language  = "en",
    currency  = "AZN"             // ISO code used for pricing
)
```

| Parameter | Default | Notes |
|:--|:--|:--|
| `activity`, `sessionId`, `callbacks` | — | As for eSIM |
| `userId: String?` | `null` | Sent as `X-User-Id` |
| `language: String` | `"en"` | Applied to the Activity locale and sent as `Accept-Language`. Built-in strings are English in this version. |
| `accentColor: Int` | brand colour | ARGB override for this launch |
| `currency: String` | `"AZN"` | Pricing currency |

### Callbacks

```kotlin
val flightCallbacks = FlightCallbacks(
    onComplete = { orderId -> /* booked and ticketed; orderId may be null */ },
    onCancel   = { /* user backed out */ },
    onFail     = { message -> showError(message) },        // String
    onProcessPayment = { request -> /* optional, see below */ }
)
```

`onProcessPayment` is optional. **When it is null the SDK confirms the booking immediately after
order creation.** When it is set, the SDK creates the order, hands you a `FlightPaymentRequest`
and finishes its Activity; charge the customer, then resume:

```kotlin
data class FlightPaymentRequest(
    val orderId: String,
    val amount: Double,
    val currency: String,
    val sessionId: String
)

Gate2Flight.resumeAfterPayment(
    activity  = this,
    orderId   = request.orderId,
    sessionId = request.sessionId,
    callbacks = FlightCallbacks(
        onComplete = { orderId -> /* ticketed */ },
        onFail     = { message -> /* confirmation failed */ }
    ),
    currency  = "AZN"
)
```

### Cabinet entry points

These open a single screen directly, with the same `FlightCallbacks`:

| Call | Opens | `userId` |
|:--|:--|:--|
| `Gate2Flight.startMyFlights(activity, sessionId, callbacks, userId, ...)` | Orders booked on this account (active and past); empty state can start a new booking | optional |
| `Gate2Flight.startTravelers(activity, sessionId, callbacks, userId, ...)` | Saved travelers and documents: list, view, add, edit | **required** |
| `Gate2Flight.startFaq(activity, sessionId, callbacks, userId, ...)` | Merchant FAQ topics and questions | optional |

### Flow

```
Welcome (T&C, first launch only) → Search → Results → Fare detail → Travelers
    → Pricing → Seat map (when the carrier provides one) → Review → [host payment] → Confirmation
```

Seat selection is shown on the review screen; in this version the seat choice is display-only and is
not yet carried on the order.

---

## 7. Hotels

### Launch

```kotlin
import com.gate2.sdk.hotels.Gate2Hotel
import com.gate2.sdk.hotels.HotelCallbacks

Gate2Hotel.start(
    activity         = this,
    sessionId        = sessionId,
    callbacks        = HotelCallbacks(
        onComplete = { confirmationNumber -> /* booked; may be null */ },
        onCancel   = { /* user backed out */ },
        onFail     = { message -> showError(message) }
    ),
    userId           = currentUser.id,
    language         = "en",
    guestNationality = "AZ"      // ISO country code, default "AZ"
)
```

| Parameter | Default | Notes |
|:--|:--|:--|
| `activity`, `sessionId`, `callbacks` | — | As for eSIM |
| `userId: String?` | `null` | Sent as `X-User-Id` |
| `language: String` | `"en"` | Activity locale + `Accept-Language`; built-in strings are English in this version |
| `accentColor: Int` | brand colour | ARGB override for this launch |
| `guestNationality: String` | `"AZ"` | Nationality sent with searches and bookings |

### Flow

```
Welcome (T&C, first launch only) → Search → Results (sort, refundable filter) → Hotel detail
    → Room pre-book (price re-confirmed) → Guest details → Confirmation
```

There is **no external payment hook for hotels in this version**: `HotelCallbacks` has no
`onProcessPayment`, and the SDK confirms the booking directly from the guest-details screen.

---

## 8. Showcase screen (travel-all)

`travel-all-<company>` adds `Gate2TravelShowcase`, a Compose screen that presents the products and
lets the host launch each one with its own callbacks. It wraps itself in the SDK theme; your app
only needs Compose to place it.

```kotlin
import com.gate2.sdk.travel.Gate2TravelShowcase
import com.gate2.sdk.travel.Gate2ShowcaseConfig
import com.gate2.sdk.travel.Gate2CabinetConfig
import com.gate2.sdk.travel.Gate2Product

setContent {
    Gate2TravelShowcase(
        onProductSelected = { product ->
            when (product) {
                Gate2Product.ESIM    -> launchEsim()
                Gate2Product.FLIGHTS -> launchFlights()
                Gate2Product.HOTELS  -> launchHotels()
            }
        },
        onBack = { finish() },   // null = embeddable (no back arrow, no back interception)
        config = Gate2ShowcaseConfig(
            headerTitle    = "Travel with us",
            headerSubtitle = "Everything for your trip, in one place",
            promoImageUrl  = null,                           // optional banner above the cards
            products       = Gate2ShowcaseConfig.defaultProducts(),
            cabinet        = Gate2CabinetConfig(
                onOpenOrders = { product ->
                    when (product) {
                        Gate2Product.FLIGHTS -> Gate2Flight.startMyFlights(this, sessionId, flightCallbacks, userId)
                        Gate2Product.ESIM    -> Gate2Esim.showMyEsims(this, sessionId, esimCallbacks, userId)
                        Gate2Product.HOTELS  -> launchHotels()
                    }
                },
                onOpenTravelers = { Gate2Flight.startTravelers(this, sessionId, flightCallbacks, userId) },
                onOpenFaq       = { Gate2Flight.startFaq(this, sessionId, flightCallbacks, userId) },
                supportWhatsApp = "+12025550123",
                supportEmail    = "support@gate2.travel",
            ),
        ),
    )
}
```

| Type | Fields |
|:--|:--|
| `Gate2ShowcaseConfig` | `headerTitle`, `headerSubtitle`, `promoImageUrl`, `products: List<Gate2ShowcaseProduct>`, `cabinet: Gate2CabinetConfig?` |
| `Gate2ShowcaseProduct` | `product: Gate2Product`, `title`, `subtitle`, `imageUrl`, `badge` (e.g. `"NEW"`), `enabled` |
| `Gate2CabinetConfig` | `onOpenOrders`, `onOpenTravelers`, `onOpenFaq`, `supportWhatsApp`, `supportEmail`. Rows appear only for the callbacks you provide. |
| `Gate2Product` | `ESIM`, `FLIGHTS`, `HOTELS` |

The rendered design is brand-specific: some brand builds ship a redesigned home with a "My cabinet"
section, others use the default card picker. The API is identical.

---

## 9. Payments

Gate2 never takes payment. For eSIM and Flights the SDK creates the order, hands it to you and
waits:

```
Gate2Esim.start(...)  /  Gate2Flight.start(...)
    │
    │   user completes the flow, SDK creates the order
    ▼
onProcessPayment(request)        ← your payment UI runs here; the SDK Activity has FINISHED
    │
    │   you charge the customer with your provider
    ▼
Gate2Esim.resumeAfterPayment(orderId, sessionId, freshCallbacks)
Gate2Flight.resumeAfterPayment(orderId, sessionId, freshCallbacks)
    │
    ▼
onComplete()                     ← SDK confirms the order (and installs the eSIM / issues tickets)
```

### The rule that prevents most support tickets

**Persist both `orderId` and `sessionId` durably inside `onProcessPayment`**, and pass them to a
fresh `resumeAfterPayment()` call with fresh callbacks. The SDK's Activity finishes while your
payment screens are in front, so the callback objects you passed to `start()` are gone by the
time payment returns, and your process may have been killed in between.

```kotlin
onProcessPayment = { request ->
    paymentPrefs.edit()
        .putString("gate2_order_id", request.orderId)
        .putString("gate2_session_id", request.sessionId)
        .apply()
    startYourPaymentFlow(amount = request.amount, currency = request.currency, reference = request.orderId)
}

// From your deep link / activity result / provider callback:
fun onPaymentSucceeded() {
    val orderId   = paymentPrefs.getString("gate2_order_id", null) ?: return
    val sessionId = paymentPrefs.getString("gate2_session_id", null) ?: return
    Gate2Esim.resumeAfterPayment(this, orderId, sessionId, EsimCallbacks(/* fresh */))
}
```

If payment **fails or is abandoned**, do not call `resumeAfterPayment()`; just clear your stored
state. The order expires on the Gate2 side.

> `Gate2TravelSdk.notifyPaymentResult()` is **deprecated** precisely because of the stale-callback
> problem above. Use `resumeAfterPayment()`.

---

## 10. Back navigation

Every product launches as its own `Activity` on top of yours, in the same task, with no special
launch flags. Inside the flow, back steps through the SDK's screens; at the root it calls
`onCancel` (or `onComplete` on a confirmation screen) and finishes the SDK Activity, which
returns the user to your screen automatically.

**Do not call `finish()` on your own Activity inside `onComplete` / `onCancel`.** That is what
makes a host app appear to exit after a product flow. If you present the showcase as its own
destination, give it `onBack` so the system back button lands where you launched it from.

---

## 11. Theming and branding

Your brand artifact already carries your look: accent colour, welcome screens, assets and (where
supplied) your typeface are compiled into `<module>-<company>`. In the normal case there is nothing
to configure.

### Per-launch overrides

```kotlin
Gate2Esim.start(..., accentColor = ContextCompat.getColor(this, R.color.brand), fontResId = R.font.brand_sans)
Gate2Flight.start(..., accentColor = ContextCompat.getColor(this, R.color.brand))
Gate2Hotel.start(...,  accentColor = ContextCompat.getColor(this, R.color.brand))
```

`fontResId` is accepted by the eSIM entry points. A variable font maps weights 400/500/600/700
automatically; a static font renders at one weight.

### App-wide `Gate2Theme`

Pass a theme at initialization or swap it at runtime. Reach for this only if the compiled-in brand
look must change; contact Gate2 first, since brand changes are normally made in the brand build.

```kotlin
import com.gate2.sdk.core.config.*

val theme = Gate2Theme.builder()
    .primaryColor(0xFF1A73E8.toInt())             // shortcut for PrimaryColors(main = ...)
    .secondaryColor(0xFF34A853.toInt())
    .backgroundColors(BackgroundColors.builder().background(0xFFFFFFFF.toInt()).card(0xFFFFFFFF.toInt()).build())
    .textColors(TextColors.builder().primary(0xFF202124.toInt()).link(0xFF1A73E8.toInt()).build())
    .statusColors(StatusColors.builder().success(0xFF34A853.toInt()).error(0xFFEA4335.toInt()).build())
    .shapes(ShapeConfig.builder().cardRadius(16).buttonRadius(24).inputRadius(8).build())   // dp
    .spacing(SpacingConfig.builder().md(16).lg(24).build())                                 // dp
    .typography(TypographyConfig.builder().titleLarge(22).bodyMedium(14).build())           // sp
    .build()

Gate2SdkConfig.builder().context(this).apiKey(token).theme(theme).build()   // at init
Gate2TravelSdk.updateTheme(theme)                                           // or later
```

| Builder | Properties |
|:--|:--|
| `PrimaryColors` / `SecondaryColors` | `main`, `dark`, `light`, `onPrimary` / `onSecondary` |
| `BackgroundColors` | `background`, `surface`, `card`, `divider`, `border` |
| `TextColors` | `primary`, `secondary`, `hint`, `disabled`, `link` |
| `StatusColors` | `success`, `warning`, `error`, `info` |
| `ShapeConfig` | `cardRadius`, `buttonRadius`, `inputRadius`, `chipRadius`, `bottomSheetRadius`, `dialogRadius`, `imageRadius`, `uniform(radius)` |
| `SpacingConfig` | `xxs` … `xxxl` |
| `TypographyConfig` | Material 3 type scale sizes (`displayLarge` … `labelSmall`) |
| `ComponentConfig` | Card elevation, button/input heights, icon and avatar sizes, divider thickness |

Presets: `Gate2Theme.cleanWhite`, `Gate2Theme.darkMode`, `Gate2Theme.rounded`, `Gate2Theme.minimal`,
and `Gate2Theme.simple(primary, secondary)`.

---

## 12. Logging

```kotlin
Gate2SdkConfig.builder()
    .context(this)
    .apiKey(token)
    .logLevel(SdkLogLevel.WARN)
    .onLog { entry: SdkLogEntry ->
        when (entry.level) {
            SdkLogLevel.ERROR, SdkLogLevel.FATAL -> crashReporter.captureMessage("${entry.tag}: ${entry.message}")
            SdkLogLevel.WARN                     -> crashReporter.addBreadcrumb(entry.message)
            else                                 -> Unit
        }
    }
    .build()
```

`SdkLogEntry` carries `level`, `tag` (e.g. `Gate2-esim`), `message` and `timestampMs`. Exceptions
thrown inside `onLog` are caught and never crash the SDK. Use `SdkLogLevel.ALL` while integrating
and `NONE` in release builds.

---

## 13. Network and security

| | |
|:--|:--|
| **Production host** | `https://gw.gate2.travel/` (unsuffixed artifacts) |
| **Staging host** | `https://api.gate2.travel/` (`-staging` artifacts) |
| **Transport** | HTTPS with certificate pinning and an embedded mTLS client certificate. Nothing to configure, no certificate to ship. |
| **Timeouts** | 30 s connect / read / write |
| **Card data** | Never touches the SDK |

Headers on every request:

```
X-API-Key: <your token>
X-Session-Id: <sessionId you passed>
X-User-Id: <userId you passed, if any>
X-SDK-Version: 1.2.1
X-Platform: Android
Accept-Language: <language you passed>
```

---

## 14. Troubleshooting

| Symptom | Cause | Fix |
|:--|:--|:--|
| `IllegalStateException: SDK already initialized` | `initialize()` called twice | Guard with `isInitialized()` |
| `onFail` with `SDK_NOT_INITIALIZED` / "SDK is not initialized." | Product called before `initialize()` | Initialize in `Application.onCreate()` |
| `onFail` with `MODULE_ACCESS_DENIED` / "not available with your current subscription" | The product module was not found at runtime (modules register by reflection) or the SDK is uninitialized. **Not** a token problem. | Confirm the product artifact resolved. If it fails only in release, check that the AAR's consumer R8 rules are applied and not overridden. |
| `onFail` with `UNAUTHORIZED` (401) | Wrong or expired token, or production token against staging (or vice versa) | Check the token and the artifact's `-staging` suffix |
| `onFail` with `INVALID_SESSION_ID` | Blank `sessionId` | Generate one per session; never pass `""` |
| Purchase never completes | `resumeAfterPayment()` not called, or a different `sessionId` | Persist **both** `orderId` and `sessionId` ([§9](#9-payments)) |
| App exits after a product flow | Host called `finish()` in `onComplete` / `onCancel` | Remove it ([§10](#10-back-navigation)) |
| eSIM entry shown but always errors | Device is not eSIM-capable | Gate on `isDeviceEsimCapable()` ([§5](#5-esim)) |
| `Duplicate class com.gate2.sdk...` at build time | Two brand artifacts on the classpath | Depend on exactly one `<module>-<company>` |
| `EsimCallbacks` overload ambiguity at compile time | Un-annotated `onFail` lambda | Write `{ error: EsimError -> ... }` |

To capture SDK internals during integration set `.logLevel(SdkLogLevel.ALL)` and supply `.onLog`.

---

## 15. Migrating from 1.0.x

The 1.0.x line (`travel.gate2:esim:1.0.x`, `travel.gate2:core:1.0.0`, and the pre-release
`com.gate2.sdk:*` coordinates in the January 2026 docs) is retired and receives no updates.

| Then | Now |
|:--|:--|
| `travel.gate2:esim:1.0.x` | `travel.gate2:esim-<company>:1.2.1` (or `travel-all-<company>`) |
| A separate `core` dependency | Transitive. Remove it. |
| Version suffixes `1.0.20-test`, `1.1.4-<company>` | `-staging` suffix on the **artifact ID**; plain semver version |
| `Gate2EsimSdk` / `Gate2HotelSdk` | One `Gate2TravelSdk.initialize()` for every product |
| `Gate2TravelScreen` / `Gate2EsimScreen` / `Gate2HotelScreen` composables wrapped in `Gate2TravelTheme` | `Gate2Flight.start()` / `Gate2Esim.start()` / `Gate2Hotel.start()`. The screens are internal; each product runs in its own Activity. |
| `PaymentRequest` + `PaymentResult` + `notifyPaymentResult()` | `EsimPaymentRequest` / `FlightPaymentRequest` + `resumeAfterPayment(orderId, sessionId)` |
| `onFail: (String) -> Unit` for eSIM | `onFail: (EsimError) -> Unit` (the `String` form still compiles via the `EsimCallbacks(...)` function) |
| `.clientCertificate(pkcs12, password)` | Remove; mTLS is embedded |

---

## 16. API summary

| Symbol | Package |
|:--|:--|
| `Gate2TravelSdk.initialize` / `.isInitialized` / `.updateTheme` / `.getInstance().getVersion()` | `com.gate2.sdk.core` |
| `Gate2SdkConfig`, `Gate2Theme`, `SdkLogLevel`, `SdkLogEntry`, colour/shape/spacing/typography configs | `com.gate2.sdk.core.config` |
| `Gate2Esim.start` / `.resumeAfterPayment` / `.showMyEsims` / `.isAvailable` / `.isDeviceEsimCapable`, `EsimCallbacks` | `com.gate2.sdk.esim` |
| `EsimPaymentRequest`, `EsimError`, `EsimErrorCode` | `com.gate2.sdk.esim.api` |
| `Gate2Flight.start` / `.startMyFlights` / `.startTravelers` / `.startFaq` / `.resumeAfterPayment`, `FlightCallbacks` | `com.gate2.sdk.flights` |
| `FlightPaymentRequest` | `com.gate2.sdk.flights.api` |
| `Gate2Hotel.start`, `HotelCallbacks` | `com.gate2.sdk.hotels` |
| `Gate2TravelShowcase`, `Gate2ShowcaseConfig`, `Gate2ShowcaseProduct`, `Gate2CabinetConfig`, `Gate2Product` | `com.gate2.sdk.travel` (travel-all only) |

The Maven group is `travel.gate2`; the Kotlin packages are `com.gate2.sdk.*`. That is expected.

---

## 17. Changelog

| Version | Date | Highlights |
|:--|:--|:--|
| **1.2.1** | Oct 2026 | Flights: seats overview on the review screen, "Select seats" entry from the passenger form, scroll edge fades on flight screens. |
| **1.2.0** | Oct 2026 | Flights: seat map after pricing (display-only selection), saved-traveler edit, recent-search removal (flights v2 endpoints). |
| **1.1.6** | Sep 2026 | eSIM: merchant banner on the search screen. |
| **1.1.5** | Sep 2026 | eSIM: plan discounts. |
| **1.1.4** | Jun – Jul 2026 | Brand and environment moved from the version into the artifact ID. Brandable typography. Flight order validation. |
| **1.1.0 – 1.1.1** | Jun 2026 | Shared `core` across products; `travel-all` umbrella with `Gate2TravelShowcase`; per-brand builds; Flights and Hotels products. |
| **1.0.x** | Jan – Jun 2026 | eSIM-only line: automatic installation, T&C flow, structured `EsimError`, My eSIMs, top-ups, device-compatibility banner. Retired coordinates. |

---

## 18. Support

| Resource | |
|:--|:--|
| **Email** | [support@gate2.travel](mailto:support@gate2.travel) |
| **Backend API reference** | [Postman collection](https://documenter.getpostman.com/view/7713462/2sB3dWq6UM) |
| **This documentation** | [github.com/gate2-travel/demo-android](https://github.com/gate2-travel/demo-android) |
| **eSIM integration guide** | [ESIM_INTEGRATION_GUIDE.md](./ESIM_INTEGRATION_GUIDE.md) |

When reporting a problem include the SDK version (`Gate2TravelSdk.getInstance().getVersion()`),
the error code or message, and the `orderId` / `sessionId` of any failed purchase.

---

<div align="center">

**Gate2 Travel SDK** · Android · © 2026 Gate2 Travel. All rights reserved.

</div>
