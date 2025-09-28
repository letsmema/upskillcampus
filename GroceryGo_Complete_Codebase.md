# GroceryGo - Complete Android Application Codebase

This document contains the complete source code for the GroceryGo Android application, including all Kotlin files, XML layouts, resources, and configuration files.

## Table of Contents
1. [Project Configuration](#project-configuration)
2. [Kotlin Source Files](#kotlin-source-files)
3. [XML Layout Files](#xml-layout-files)
4. [Resource Files](#resource-files)
5. [Database Schema](#database-schema)

---

## Project Configuration

### AndroidManifest.xml
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="GroceryGo"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.GroceryGo"
        tools:targetApi="31">

        <activity android:name=".activities.BuyerDashboardActivity" />
        <activity android:name=".activities.SellerDashboardActivity" />
        <activity android:name=".activities.LoginActivity" />
        <activity android:name=".activities.RegisterActivity" />
        <activity android:name=".activities.SettingsActivity" />
        <activity android:name=".activities.ProductCatalogActivity" />
        <activity android:name=".activities.ProductDetailsActivity" />
        <activity android:name=".activities.CartActivity" />
        <activity android:name=".activities.CheckoutActivity" />
        <activity android:name=".activities.OrderHistoryActivity" />
        <activity android:name=".activities.AddProductActivity" />
        <activity android:name=".activities.EditProductActivity" />
        <activity android:name=".activities.SellerOrdersActivity" />
        <activity android:name=".activities.AnalyticsActivity" />
        <activity android:name=".activities.UserTypeSelectionActivity" />
        <activity
            android:name=".activities.SplashActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>

</manifest>
```

### app/build.gradle.kts
```kotlin
plugins {
    id("com.android.application")
    kotlin("android")
}

android {
    namespace = "com.example.grocerygo"
    compileSdk = 34

    defaultConfig {
        applicationId = "com.example.grocerygo"
        minSdk = 24 // Android 7.0+
        targetSdk = 34
        versionCode = 1
        versionName = "1.0"

        testInstrumentationRunner = "androidx.test.runner.AndroidJUnitRunner"
    }

    buildFeatures {
        viewBinding = true
        buildConfig = true
    }

    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17
        targetCompatibility = JavaVersion.VERSION_17
    }

    kotlinOptions {
        jvmTarget = "17"
    }
    buildTypes {
        debug {
            buildConfigField("boolean", "ENABLE_FIREBASE", (project.findProperty("enableFirebase") == "true").toString())
        }
        release {
            buildConfigField("boolean", "ENABLE_FIREBASE", (project.findProperty("enableFirebase") == "true").toString())
            isMinifyEnabled = false
        }
    }
}

dependencies {
    // Core AndroidX
    implementation("androidx.core:core-ktx:1.13.1")
    implementation("androidx.appcompat:appcompat:1.6.1")
    implementation("com.google.android.material:material:1.12.0")
    implementation("androidx.constraintlayout:constraintlayout:2.1.4")
    implementation("androidx.cardview:cardview:1.0.0")
    implementation("androidx.recyclerview:recyclerview:1.3.2")
    implementation("io.coil-kt:coil:2.6.0")
    implementation("com.google.zxing:core:3.5.1")
    implementation("com.github.PhilJay:MPAndroidChart:v3.1.0")

    // Lifecycle
    implementation("androidx.lifecycle:lifecycle-runtime-ktx:2.8.1")

    // Firebase removed per request; app runs fully local without it

    // Unit & Instrumented Tests
    testImplementation("junit:junit:4.13.2")
    androidTestImplementation("androidx.test.ext:junit:1.1.5")
    androidTestImplementation("androidx.test.espresso:espresso-core:3.5.1")
}
```

---

## Kotlin Source Files

### Activities

#### SplashActivity.kt
```kotlin
package com.example.grocerygo.activities

import android.content.Intent
import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity
import com.example.grocerygo.session.SessionManager

class SplashActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val session = SessionManager(this).getSession()
        if (session != null) {
            val (username, userType) = session
            val target = if (userType == "seller") SellerDashboardActivity::class.java else BuyerDashboardActivity::class.java
            val i = Intent(this, target)
            i.putExtra("username", username)
            startActivity(i)
            finish()
        } else {
            startActivity(Intent(this, UserTypeSelectionActivity::class.java))
            finish()
        }
    }
}
```

#### UserTypeSelectionActivity.kt
```kotlin
package com.example.grocerygo.activities

import android.content.Intent
import android.os.Bundle
import android.util.Log
import android.widget.Button
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import androidx.appcompat.app.AlertDialog
import com.example.grocerygo.R
import com.example.grocerygo.session.SessionManager

class UserTypeSelectionActivity : AppCompatActivity() {
    
    companion object {
        private const val TAG = "UserTypeSelection"
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        try {
            Log.d(TAG, "onCreate started")
            // Auto-login if session exists
            val session = SessionManager(this).getSession()
            if (session != null) {
                val (username, userType) = session
                val target = if (userType == "seller") SellerDashboardActivity::class.java else BuyerDashboardActivity::class.java
                val i = Intent(this, target)
                i.putExtra("username", username)
                startActivity(i)
                finish()
                return
            }
            setContentView(R.layout.activity_user_type_selection)
            Log.d(TAG, "Layout set successfully")

            val buyerBtn = findViewById<Button>(R.id.btnBuyer)
            val sellerBtn = findViewById<Button>(R.id.btnSeller)
            
            if (buyerBtn == null || sellerBtn == null) {
                Log.e(TAG, "Buttons not found in layout")
                Toast.makeText(this, "Layout Error: Buttons not found", Toast.LENGTH_LONG).show()
                return
            }

            buyerBtn.setOnClickListener {
                try {
                    Log.d(TAG, "Buyer button clicked")
                    showAuthChoiceDialog("buyer")
                } catch (e: Exception) {
                    Log.e(TAG, "Error starting auth choice for buyer", e)
                    Toast.makeText(this, "Error: ${e.message}", Toast.LENGTH_SHORT).show()
                }
            }

            sellerBtn.setOnClickListener {
                try {
                    Log.d(TAG, "Seller button clicked")
                    showAuthChoiceDialog("seller")
                } catch (e: Exception) {
                    Log.e(TAG, "Error starting auth choice for seller", e)
                    Toast.makeText(this, "Error: ${e.message}", Toast.LENGTH_SHORT).show()
                }
            }
            
            Log.d(TAG, "onCreate completed successfully")
        } catch (e: Exception) {
            Log.e(TAG, "Fatal error in onCreate", e)
            Toast.makeText(this, "App Error: ${e.message}", Toast.LENGTH_LONG).show()
        }
    }

    private fun showAuthChoiceDialog(userType: String) {
        val options = arrayOf("Register", "Login")
        AlertDialog.Builder(this)
            .setTitle("Continue as ${userType.capitalize()}")
            .setItems(options) { _, which ->
                when (which) {
                    0 -> {
                        val i = Intent(this, RegisterActivity::class.java)
                        i.putExtra("usertype", userType)
                        startActivity(i)
                    }
                    1 -> {
                        val i = Intent(this, LoginActivity::class.java)
                        i.putExtra("usertype", userType)
                        startActivity(i)
                    }
                }
            }
            .setNegativeButton("Cancel", null)
            .show()
    }
}
```

#### LoginActivity.kt
```kotlin
package com.example.grocerygo.activities

import android.content.Intent
import android.os.Bundle
import android.util.Log
import android.widget.Button
import android.widget.EditText
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import com.example.grocerygo.R
import com.example.grocerygo.database.DatabaseHelper
import com.example.grocerygo.session.SessionManager

class LoginActivity : AppCompatActivity() {

    companion object {
        private const val TAG = "LoginActivity"
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        try {
            Log.d(TAG, "onCreate started")
            setContentView(R.layout.activity_login)
            Log.d(TAG, "Layout set successfully")

            val usernameEditText = findViewById<EditText>(R.id.editUsername)
            val passwordEditText = findViewById<EditText>(R.id.editPassword)
            val showCb = findViewById<android.widget.CheckBox>(R.id.cbShowPassword)
            // Toggle password visibility by tapping the field end (simple: switch inputType on click)
            showCb?.setOnCheckedChangeListener { _, isChecked ->
                val pos = passwordEditText.selectionEnd
                passwordEditText.inputType = if (isChecked) {
                    android.text.InputType.TYPE_CLASS_TEXT or android.text.InputType.TYPE_TEXT_VARIATION_VISIBLE_PASSWORD
                } else {
                    android.text.InputType.TYPE_CLASS_TEXT or android.text.InputType.TYPE_TEXT_VARIATION_PASSWORD
                }
                passwordEditText.setSelection(pos)
            }
            val loginButton = findViewById<Button>(R.id.btnLogin)

            if (usernameEditText == null || passwordEditText == null || loginButton == null) {
                Log.e(TAG, "Views not found in layout")
                Toast.makeText(this, "Layout Error: Views not found", Toast.LENGTH_LONG).show()
                return
            }

            loginButton.setOnClickListener {
                try {
                    Log.d(TAG, "Login button clicked")
                    val username = usernameEditText.text.toString().trim()
                    val password = passwordEditText.text.toString().trim()

                    val userType = intent.getStringExtra("usertype") ?: "buyer"
                    val db = DatabaseHelper(this)
                    val ok = db.loginUser(username, password, userType)
                    if (ok) {
                        Log.d(TAG, "$userType login successful")
                        // persist session
                        SessionManager(this).saveSession(username, userType)
                        val target = if (userType == "seller") SellerDashboardActivity::class.java else BuyerDashboardActivity::class.java
                        val intent = Intent(this, target)
                        intent.putExtra("username", username)
                        startActivity(intent)
                        finish()
                    } else {
                        Log.d(TAG, "Invalid credentials")
                        Toast.makeText(this, "Invalid credentials", Toast.LENGTH_SHORT).show()
                    }
                } catch (e: Exception) {
                    Log.e(TAG, "Error during login", e)
                    Toast.makeText(this, "Login Error: ${e.message}", Toast.LENGTH_SHORT).show()
                }
            }
            
            Log.d(TAG, "onCreate completed successfully")
        } catch (e: Exception) {
            Log.e(TAG, "Fatal error in onCreate", e)
            Toast.makeText(this, "App Error: ${e.message}", Toast.LENGTH_LONG).show()
        }
    }
}
```

#### RegisterActivity.kt
```kotlin
package com.example.grocerygo.activities

import android.content.Intent
import android.os.Bundle
import android.util.Log
import android.widget.Button
import android.widget.EditText
import android.widget.TextView
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import com.example.grocerygo.R
import com.example.grocerygo.database.DatabaseHelper
import com.example.grocerygo.session.SessionManager

class RegisterActivity : AppCompatActivity() {

    companion object {
        private const val TAG = "RegisterActivity"
    }

    private lateinit var userType: String

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        try {
            Log.d(TAG, "onCreate started")
            setContentView(R.layout.activity_register)
            Log.d(TAG, "Layout set successfully")

            userType = intent.getStringExtra("usertype") ?: "buyer"
            Log.d(TAG, "User type: $userType")

            // Update the user type display
            val tvUserType = findViewById<TextView>(R.id.tvUserType)
            if (tvUserType != null) {
                tvUserType.text = userType.capitalize()
            }

            val username = findViewById<EditText>(R.id.editUsername)
            val password = findViewById<EditText>(R.id.editPassword)
            val showCb = findViewById<android.widget.CheckBox>(R.id.cbShowPassword)
            val registerBtn = findViewById<Button>(R.id.btnRegister)
            showCb?.setOnCheckedChangeListener { _, isChecked ->
                val pos = password.selectionEnd
                password.inputType = if (isChecked) {
                    android.text.InputType.TYPE_CLASS_TEXT or android.text.InputType.TYPE_TEXT_VARIATION_VISIBLE_PASSWORD
                } else {
                    android.text.InputType.TYPE_CLASS_TEXT or android.text.InputType.TYPE_TEXT_VARIATION_PASSWORD
                }
                password.setSelection(pos)
            }

            if (username == null || password == null || registerBtn == null) {
                Log.e(TAG, "Views not found in layout")
                Toast.makeText(this, "Layout Error: Views not found", Toast.LENGTH_LONG).show()
                return
            }

            registerBtn.setOnClickListener {
                try {
                    Log.d(TAG, "Register button clicked")
                    val user = username.text.toString()
                    val pass = password.text.toString()

                    if (user.isNotEmpty() && pass.isNotEmpty()) {
                        val db = DatabaseHelper(this)
                        val success = db.registerUser(user, pass, userType)
                        if (success) {
                            Log.d(TAG, "Registration successful for $userType")
                            Toast.makeText(this, "Registered Successfully", Toast.LENGTH_SHORT).show()
                            // Auto-login after register
                            SessionManager(this).saveSession(user, userType)
                            val target = if (userType == "seller") SellerDashboardActivity::class.java else BuyerDashboardActivity::class.java
                            val intent = Intent(this, target)
                            intent.putExtra("username", user)
                            startActivity(intent)
                            finish()
                        } else {
                            Log.d(TAG, "Registration failed: user exists?")
                            Toast.makeText(this, "Username already exists", Toast.LENGTH_SHORT).show()
                        }
                    } else {
                        Log.d(TAG, "Empty fields")
                        Toast.makeText(this, "Fill all fields", Toast.LENGTH_SHORT).show()
                    }
                } catch (e: Exception) {
                    Log.e(TAG, "Error during registration", e)
                    Toast.makeText(this, "Registration Error: ${e.message}", Toast.LENGTH_SHORT).show()
                }
            }
            
            Log.d(TAG, "onCreate completed successfully")
        } catch (e: Exception) {
            Log.e(TAG, "Fatal error in onCreate", e)
            Toast.makeText(this, "App Error: ${e.message}", Toast.LENGTH_LONG).show()
        }
    }
}
```

#### BuyerDashboardActivity.kt
```kotlin
package com.example.grocerygo.activities

import android.content.Intent
import android.os.Bundle
import android.widget.Button
import android.widget.TextView
import android.widget.Toast
import androidx.appcompat.app.AlertDialog
import androidx.appcompat.app.AppCompatActivity
import com.example.grocerygo.R
import com.example.grocerygo.session.SessionManager

class BuyerDashboardActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        try {
            setContentView(R.layout.activity_buyer_dashboard)

            val textView = findViewById<TextView>(R.id.dashboardText)
            val username = intent.getStringExtra("username")
            textView.text = "Welcome, ${username ?: "Buyer"}!"

            val btnMenu = findViewById<Button>(R.id.btnMenu)
            val optBrowse = findViewById<TextView>(R.id.optBrowse)
            val optCart = findViewById<TextView>(R.id.optCart)
            val optCheckout = findViewById<TextView>(R.id.optCheckout)
            val optOrders = findViewById<TextView>(R.id.optOrders)
            btnMenu.setOnClickListener {
                try {
                    showMenuDialog()
                } catch (e: Exception) {
                    Toast.makeText(this, "Menu Error: ${e.message}", Toast.LENGTH_SHORT).show()
                }
            }

            // username already read above
            optBrowse?.setOnClickListener {
                val i = Intent(this, ProductCatalogActivity::class.java)
                i.putExtra("username", username)
                startActivity(i)
            }
            optCart?.setOnClickListener {
                val i = Intent(this, CartActivity::class.java)
                i.putExtra("username", username)
                startActivity(i)
            }
            optCheckout?.setOnClickListener {
                val i = Intent(this, CheckoutActivity::class.java)
                i.putExtra("username", username)
                startActivity(i)
            }
            optOrders?.setOnClickListener {
                val i = Intent(this, OrderHistoryActivity::class.java)
                i.putExtra("username", username)
                startActivity(i)
            }
        } catch (e: Exception) {
            Toast.makeText(this, "App Error: ${e.message}", Toast.LENGTH_LONG).show()
        }
    }

    private fun showMenuDialog() {
        try {
            val options = arrayOf("Product Catalog", "Cart", "Order History")
            AlertDialog.Builder(this)
                .setTitle("Menu")
                .setItems(options) { _, which ->
                    try {
                        val username = intent.getStringExtra("username")
                        when (which) {
                            0 -> {
                                val i = Intent(this, ProductCatalogActivity::class.java)
                                i.putExtra("username", username)
                                startActivity(i)
                            }
                            1 -> {
                                val i = Intent(this, CartActivity::class.java)
                                i.putExtra("username", username)
                                startActivity(i)
                            }
                            2 -> {
                                val i = Intent(this, OrderHistoryActivity::class.java)
                                i.putExtra("username", username)
                                startActivity(i)
                            }
                            
                        }
                    } catch (e: Exception) {
                        Toast.makeText(this, "Navigation Error: ${e.message}", Toast.LENGTH_SHORT).show()
                    }
                }
                .setNegativeButton("Cancel", null)
                .show()
        } catch (e: Exception) {
            Toast.makeText(this, "Dialog Error: ${e.message}", Toast.LENGTH_SHORT).show()
        }
    }
    
    override fun onResume() {
        super.onResume()
        try {
            val username = intent.getStringExtra("username")
            findViewById<Button>(R.id.tabCatalog)?.setOnClickListener {
                val i = Intent(this, ProductCatalogActivity::class.java)
                i.putExtra("username", username)
                startActivity(i)
            }
            findViewById<Button>(R.id.tabCart)?.setOnClickListener {
                val i = Intent(this, CartActivity::class.java)
                i.putExtra("username", username)
                startActivity(i)
            }
            findViewById<Button>(R.id.tabSettings)?.setOnClickListener {
                val i = Intent(this, SettingsActivity::class.java)
                i.putExtra("username", username)
                i.putExtra("usertype", "buyer")
                startActivity(i)
            }
        } catch (e: Exception) {
            Toast.makeText(this, "Navigation Error: ${e.message}", Toast.LENGTH_SHORT).show()
        }
    }
}
```

---

*[This file continues with all remaining activities, layouts, and resources. The complete file would be very large, so I'm showing the structure and first few files. Would you like me to continue with the remaining files or would you prefer a different approach?]*
