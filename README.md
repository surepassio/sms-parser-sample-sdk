# SMS Parser SDK - Android Integration Guide

A simple and clean guide to integrate the SMS Parser SDK into your Android application.

## 📋 Table of Contents
- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Integration Steps](#integration-steps)
- [Configuration](#configuration)
- [Usage](#usage)
- [Permissions](#permissions)
- [Features](#features)

---

## 🎯 Overview

This sample app demonstrates how to integrate and use the SMS Parser SDK to parse SMS messages and retrieve Filtered data. The SDK provides automatic SMS parsing capabilities with a simple callback-based API.

**Key Features:**
- Automatic SMS parsing
- Real-time SDK initialization status
- Permission handling
- Jetpack Compose UI

---

## ⚙️ Prerequisites

- **Android Studio**: Arctic Fox or later
- **Minimum SDK**: 29 (Android 10)
- **Target SDK**: 36
- **Kotlin**: 2.0.21+
- **Gradle**: 8.13.2+
- **Java Version**: 11

---

## 🚀 Integration Steps

### Step 1: Generating Your SDK Token

Before integrating the SDK, you need to obtain an authentication token from the Digiboost API.

#### 1.1 Get API Details from Your Sales Manager

Contact your sales manager to receive:
- Digilocker initialize endpoint URL
- Authorization Bearer Token (required for API access)
- Access permissions

#### 1.2 Environment Configuration

We provide two environments for different stages of development:

| Environment | Base URL | Usage |
|-------------|----------|-------|
| **Staging** | `https://sandbox.surepass.app` | For development and testing |
| **Production** | `https://kyc-api.surepass.app` | For live applications |

#### 1.3 API Parameters

| Parameter | Type | Required | Description | Default Value |
|-----------|------|----------|-------------|---------------|
| `webhook_url` | String | ❌ Optional | You can get data in your own webhook URL | None |

---

### Step 2: Configure Repository Access

Add the GitHub Packages repository to your `settings.gradle.kts`:

```kotlin
dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        
        // GitHub Packages for SMS Parser SDK
        maven {
            name = "GitHubPackages"
            url = uri("https://maven.pkg.github.com/surepassio/sms-parser-sdk")
            credentials {
                username = "YOUR_GITHUB_USERNAME"
                password = "YOUR_GITHUB_TOKEN"
            }
        }
    }
}
```

> **Note:** Replace `YOUR_GITHUB_USERNAME` and `YOUR_GITHUB_TOKEN` with your actual GitHub credentials. You can generate a personal access token from GitHub Settings → Developer Settings → Personal Access Tokens.

---

### Step 3: Add Dependencies

Add the following dependencies to your `app/build.gradle.kts`:

```kotlin
dependencies {
    // SMS Parser SDK
    implementation("io.surepass.smsparsersdk:sms-parser-sdk:1.0.0")
    
    // Coroutines for async operations
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.7.3")
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:1.7.3")
}
```

---

### Step 4: Configure AndroidManifest.xml

Add required permissions to your `AndroidManifest.xml`:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
    
    <!-- SMS Permissions -->
    <uses-permission android:name="android.permission.READ_SMS" />
    <uses-permission android:name="android.permission.RECEIVE_SMS" />
    
    <!-- Notification Permission (Android 13+) -->
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
    
    <!-- Internet Permissions -->
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    
    <application
        android:name=".SampleApp"
        ...>
        <!-- Your activities -->
    </application>
</manifest>
```

> **Important:** Make sure to set `android:name` to your Application class where SDK initialization happens.

---

### Step 5: Initialize SDK in Application Class

Create an `Application` class and initialize the SDK:

```kotlin
class SampleApp : Application() {
    override fun onCreate() {
        super.onCreate()
        
        // Initialize SDK once at app startup
        SmsParserSdk.initialize(
            context = this,
            token = "YOUR_API_TOKEN",
            environment = Environment.PRODUCTION  // or Environment.STAGING
        )
    }
}
```

**Environment Options:**
- `Environment.PRODUCTION` - For production use
- `Environment.STAGING` - For testing/development

> **Note:** Replace `YOUR_API_TOKEN` with your actual API token provided by Surepass.

---

## 📱 Usage

### 1. Request Runtime Permissions

Request SMS and notification permissions at runtime (required for Android 6.0+):

```kotlin
private val requestPermissionLauncher = registerForActivityResult(
    ActivityResultContracts.RequestMultiplePermissions()
) { permissions ->
    val allGranted = permissions.entries.all { it.value }
    if (allGranted) {
        // Permissions granted - setup SDK
        setupSDKCallback()
    } else {
        // Handle permission denial
    }
}

private fun requestSMSPermissions() {
    val permissions = mutableListOf(
        Manifest.permission.READ_SMS,
        Manifest.permission.RECEIVE_SMS
    )
    
    // Add notification permission for Android 13+
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
        permissions.add(Manifest.permission.POST_NOTIFICATIONS)
    }
    
    requestPermissionLauncher.launch(permissions.toTypedArray())
}
```

---

### 2. Setup SDK Initialization Callback

Monitor SDK initialization status:

```kotlin
SmsParserSdk.getInstance().setInitializationCallback(object : InitializationCallback {
    override fun onInitializationStarted() {
        // SDK initialization started
    }
    
    override fun onInitializationCompleted(data: SmsParserResponseData) {
        // SDK ready to use
        // data contains client_id, request_id, and url
    }
    
    override fun onInitializationFailed(error: String) {
        // Handle initialization error
    }
})
```

> **Note:** SDK initialization takes approximately 20 seconds. The app should handle this loading state appropriately.

---

### 3. Parse SMS Messages

Use `SmsParserManager` to parse SMS content:

```kotlin
val smsParserManager = SmsParserManager.getInstance()

smsParserManager.parseSms(smsContent, object : SmsParserCallback<SmsData> {
    override fun onSuccess(data: SmsData) {
        // SMS parsed successfully
        // Access: data.sender, data.message, data.category
    }
    
    override fun onError(exception: Exception, message: String) {
        // Handle parsing error
    }
    
    override fun onLoading() {
        // Show loading state
    }
})
```

**SmsData Properties:**
- `sender` - SMS sender information
- `message` - Parsed message content
- `category` - Message category (if available)

---

### 4. Retrieve Sms Parsing Data

Get SMS Parsing data after SDK initialization:

```kotlin
val data = smsParserManager.getSMSParsingData()
if (data != null) {
    // Access: data
} else {
    // Data not available yet (SDK still initializing)
}
```

---

## 🔐 Permissions

The SDK requires the following permissions:

| Permission | Purpose | Required | Runtime |
|------------|---------|----------|---------|
| `READ_SMS` | Read SMS messages for parsing | ✅ Yes | ✅ Yes |
| `RECEIVE_SMS` | Receive incoming SMS | ✅ Yes | ✅ Yes |
| `POST_NOTIFICATIONS` | Show notifications (Android 13+) | ✅ Yes | ✅ Yes (API 33+) |
| `INTERNET` | API communication | ✅ Yes | ❌ No |
| `ACCESS_NETWORK_STATE` | Check network connectivity | ✅ Yes | ❌ No |

**Runtime Permission Handling:**
- Request permissions before using SDK features
- `POST_NOTIFICATIONS` is only required on Android 13 (API 33) and above
- Handle permission denial gracefully
---

## ✨ Features

### SDK Features
- ✅ Automatic SMS parsing
- ✅ Real-time initialization callbacks
- ✅ Error handling and logging
- ✅ Production and staging environments
- ✅ Coroutine-based async operations

---

## 🔧 Build Configuration

**Gradle Version:** 8.13.2  
**Android Gradle Plugin:** 8.13.2  
**Compile SDK:** 36  
**Min SDK:** 29  
**Target SDK:** 36  
**Java Compatibility:** 11  

---

## 📝 Important Notes


1. **Token Security:** Never commit API tokens to version control. Use:
   - Environment variables
   - `local.properties` file (add to `.gitignore`)
   - Build config fields
   - Secure key management systems

2. **GitHub Token:** Store GitHub credentials securely:
   ```properties
   # local.properties (add to .gitignore)
   github.username=YOUR_USERNAME
   github.token=YOUR_TOKEN
   ```
   
   Then reference in `settings.gradle.kts`:
   ```kotlin
   val githubUsername = providers.gradleProperty("github.username").get()
   val githubToken = providers.gradleProperty("github.token").get()
   ```

3. **Permissions:** Always request permissions before using SDK features. Handle denial cases gracefully.

4. **Error Handling:** Implement proper error handling for network failures, initialization errors, and parsing failures.

---

## 🐛 Troubleshooting

### SDK Initialization Fails
- Check internet connectivity
- Verify API token is valid
- Ensure correct environment is set
- Check logs for detailed error messages

### Permission Issues
- Verify permissions are declared in AndroidManifest.xml
- Ensure runtime permissions are requested and granted
- Check if permissions are revoked in device settings

### Build Issues
- Verify GitHub credentials are correct
- Check if you have access to the repository
- Clear Gradle cache: `./gradlew clean`
- Invalidate caches in Android Studio

---

## 📞 Support

For issues or questions:
- Check SDK documentation
- Review sample code in this repository
- Contact Surepass support: **tech@surepass.app**

---

## 📄 License

This sample application is provided as-is for integration reference purposes.

---

**Happy Coding! 🚀**
