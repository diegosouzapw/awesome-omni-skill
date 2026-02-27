---
name: mobile-security
description: Android security patterns for secure storage, network security, input validation, and authentication.
---

# Mobile Security Patterns

Security best practices for Android.

## Secure Storage

### EncryptedSharedPreferences

```kotlin
// Create encrypted preferences
private fun createSecurePrefs(context: Context): SharedPreferences {
    val masterKey = MasterKey.Builder(context)
        .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
        .build()
    
    return EncryptedSharedPreferences.create(
        context,
        "secure_prefs",
        masterKey,
        EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
        EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
    )
}

// Usage
class TokenStorage(context: Context) {
    private val prefs = createSecurePrefs(context)
    
    var accessToken: String?
        get() = prefs.getString("access_token", null)
        set(value) = prefs.edit().putString("access_token", value).apply()
    
    fun clear() = prefs.edit().clear().apply()
}
```

### Android Keystore

```kotlin
// Generate key in Keystore
fun generateSecretKey(alias: String) {
    val keyGenerator = KeyGenerator.getInstance(
        KeyProperties.KEY_ALGORITHM_AES,
        "AndroidKeyStore"
    )
    
    keyGenerator.init(
        KeyGenParameterSpec.Builder(alias,
            KeyProperties.PURPOSE_ENCRYPT or KeyProperties.PURPOSE_DECRYPT)
            .setBlockModes(KeyProperties.BLOCK_MODE_GCM)
            .setEncryptionPaddings(KeyProperties.ENCRYPTION_PADDING_NONE)
            .setUserAuthenticationRequired(true)
            .setUserAuthenticationParameters(300, KeyProperties.AUTH_BIOMETRIC_STRONG)
            .build()
    )
    
    keyGenerator.generateKey()
}
```

## Network Security

### Network Security Config

```xml
<!-- res/xml/network_security_config.xml -->
<?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <base-config cleartextTrafficPermitted="false">
        <trust-anchors>
            <certificates src="system"/>
        </trust-anchors>
    </base-config>
    
    <!-- Debug only -->
    <debug-overrides>
        <trust-anchors>
            <certificates src="user"/>
        </trust-anchors>
    </debug-overrides>
    
    <!-- Certificate pinning -->
    <domain-config>
        <domain includeSubdomains="true">api.example.com</domain>
        <pin-set expiration="2025-12-31">
            <pin digest="SHA-256">AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=</pin>
            <pin digest="SHA-256">BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB=</pin>
        </pin-set>
    </domain-config>
</network-security-config>
```

### Certificate Pinning (Ktor)

```kotlin
val client = HttpClient(OkHttp) {
    engine {
        config {
            certificatePinner(
                CertificatePinner.Builder()
                    .add("api.example.com", "sha256/AAAA...")
                    .add("api.example.com", "sha256/BBBB...")  // Backup
                    .build()
            )
        }
    }
}
```

## Safe Logging

```kotlin
// ❌ NEVER log sensitive data
Log.d("Auth", "Token: $token")

// ✅ Release-safe logging with Timber
class ReleaseTree : Timber.Tree() {
    override fun log(priority: Int, tag: String?, message: String, t: Throwable?) {
        if (priority >= Log.WARN) {
            // Send to crash reporting
            Crashlytics.log(priority, tag, message)
        }
    }
}

// In Application
if (BuildConfig.DEBUG) {
    Timber.plant(Timber.DebugTree())
} else {
    Timber.plant(ReleaseTree())
}
```

## Input Validation

```kotlin
// Validate before use
fun validateEmail(email: String): Result<String> {
    return when {
        email.isBlank() -> Result.failure(ValidationError.Empty)
        !Patterns.EMAIL_ADDRESS.matcher(email).matches() -> 
            Result.failure(ValidationError.InvalidFormat)
        email.length > 254 -> Result.failure(ValidationError.TooLong)
        else -> Result.success(email)
    }
}

// SQL injection prevention - use parameterized queries
@Query("SELECT * FROM users WHERE id = :userId")
suspend fun getUser(userId: String): User?
```

## Biometric Authentication

```kotlin
val biometricPrompt = BiometricPrompt(
    activity,
    executor,
    object : BiometricPrompt.AuthenticationCallback() {
        override fun onAuthenticationSucceeded(result: BiometricPrompt.AuthenticationResult) {
            val cipher = result.cryptoObject?.cipher
            // Use cipher to decrypt sensitive data
        }
    }
)

val promptInfo = BiometricPrompt.PromptInfo.Builder()
    .setTitle("Authenticate")
    .setNegativeButtonText("Cancel")
    .setAllowedAuthenticators(BiometricManager.Authenticators.BIOMETRIC_STRONG)
    .build()

biometricPrompt.authenticate(promptInfo, BiometricPrompt.CryptoObject(cipher))
```

## ProGuard/R8 Security

```proguard
# R8 rules for security
-keepattributes SourceFile,LineNumberTable  # For crash reports only

# Obfuscate sensitive classes
-repackageclasses 'a'
-allowaccessmodification

# Remove logging
-assumenosideeffects class android.util.Log {
    public static *** d(...);
    public static *** v(...);
    public static *** i(...);
}
```

---

**Remember**: Security is not optional. Build it in from the start.
