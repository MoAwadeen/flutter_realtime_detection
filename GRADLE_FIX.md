# Flutter Gradle Build Fix - Complete Solution

## Problem Identified
The project was experiencing a Gradle build error:
```
java.lang.NoClassDefFoundError: Could not initialize class org.codehaus.groovy.vmplugin.v7.Java7
```

**Root Cause**: 
- `gradle-wrapper.properties` was pointing to **Gradle 5.4.1** (from 2020)
- Even though `build.gradle` specified AGP 7.3.1, the wrapper was forcing the old Gradle daemon
- Old Gradle 5.4.1 cannot initialize Groovy on Java 11+

## Solution Applied ✅

### Files Updated:

#### 1. `android/gradle/wrapper/gradle-wrapper.properties` 🔧 **CRITICAL**
```properties
distributionUrl=https\://services.gradle.org/distributions/gradle-7.6-all.zip
```
**Changed from**: `gradle-5.4.1-all.zip` → `gradle-7.6-all.zip`

This is the **key fix**. The wrapper controls which Gradle version is actually used.

#### 2. `android/build.gradle`
Added wrapper task:
```gradle
wrapper {
    gradleVersion = '7.6'
}
```

#### 3. `android/gradle.properties`
Optimized for modern Gradle:
```properties
org.gradle.jvmargs=-Xmx2048M
android.useAndroidX=true
android.enableJetifier=true
android.enableR8=true
android.gradle.jetifier.ignorelist=*
```

#### 4. `android/app/build.gradle`
Updated to Java 11:
```gradle
compileSdkVersion 33
compileOptions {
    sourceCompatibility = JavaVersion.VERSION_11
    targetCompatibility = JavaVersion.VERSION_11
}
```

## Complete Fix - Step by Step

### Step 1: Clean Everything (PowerShell)
```powershell
# Kill Gradle daemon
cd android
.\gradlew --stop
cd ..

# Clean caches
flutter clean
rmdir -r .gradle (if exists)
cd android
.\gradlew clean
cd ..
```

### Step 2: Verify Gradle Version
```powershell
cd android
.\gradlew --version
# Should show: Gradle 7.6
cd ..
```

### Step 3: Fresh Build
```powershell
flutter pub get
flutter run
```

---

## Why This Works

| Issue | Root Cause | Fix |
|-------|-----------|-----|
| **Groovy error** | Gradle 5.4.1 doesn't support Java 11+ | Upgraded to Gradle 7.6 |
| **Daemon cached old version** | Wrapper properties pointed to old Gradle | Updated wrapper to 7.6 |
| **AGP mismatch** | AGP 3.5.3 incompatible with Java 11 | Upgraded to AGP 7.3.1 |
| **SDK version too low** | compileSdkVersion 28 is obsolete | Updated to 33 |

---

## Troubleshooting

### If still seeing Groovy error:

**Option 1: Force Gradle Update**
```powershell
cd android
# Delete cached Gradle
rmdir -r $env:USERPROFILE\.gradle\wrapper\dists
.\gradlew --version
cd ..
```

**Option 2: Use Gradle wrapper command** (instead of `./gradlew`)
```powershell
cd android
# On Windows, PowerShell might have issues with &&
# Run separately:
.\gradlew clean
.\gradlew build
cd ..
```

**Option 3: Check Java version**
```powershell
java -version
# Should show Java 11 or higher
```

If Java 8:
- Download Java 11+ from [oracle.com](https://www.oracle.com/java/technologies/downloads/)
- Update JAVA_HOME environment variable

### If build still fails:

**Clear all Gradle cache**:
```powershell
$gradleUserHome = "$env:USERPROFILE\.gradle"
rmdir -r $gradleUserHome\caches
rmdir -r $gradleUserHome\wrapper\dists

# Then rebuild
cd android
.\gradlew clean
cd ..
flutter run
```

**Increase heap for large builds**:
```properties
# In android/gradle.properties:
org.gradle.jvmargs=-Xmx3072M
```

---

## Version Summary

| Component | Before | After | Why |
|-----------|--------|-------|-----|
| Gradle | 5.4.1 | 7.6 | Java 11+ support, Groovy fix |
| AGP | 3.5.3 | 7.3.1 | Modern Android, Java 11+ |
| compileSdkVersion | 28 | 33 | Google Play requirement |
| Java | 1.8 | 11 | Modern language features |
| minSdkVersion | 21 | 21 | Still compatible |
| targetSdkVersion | 29 | 33 | Current standard |

---

## What Changed in Code

✅ **android/gradle/wrapper/gradle-wrapper.properties** (CRITICAL)
```diff
- distributionUrl=https\://services.gradle.org/distributions/gradle-5.4.1-all.zip
+ distributionUrl=https\://services.gradle.org/distributions/gradle-7.6-all.zip
```

✅ **android/build.gradle**
```diff
  dependencies {
-   classpath 'com.android.tools.build:gradle:3.5.3'
+   classpath 'com.android.tools.build:gradle:7.3.1'
  }
```

✅ **android/app/build.gradle**
```diff
- compileSdkVersion 29
+ compileSdkVersion 33
- sourceCompatibility = 1.8
- targetCompatibility = 1.8
+ sourceCompatibility = JavaVersion.VERSION_11
+ targetCompatibility = JavaVersion.VERSION_11
```

✅ **android/gradle.properties**
```diff
  org.gradle.jvmargs=-Xmx1536M
+ org.gradle.jvmargs=-Xmx2048M
+ android.enableR8=true
  android.useAndroidX=true
```

---

## References

- [Gradle 7.6 Release](https://gradle.org/releases/)
- [AGP 7.3.1 Documentation](https://developer.android.com/studio/releases/gradle-plugin#7-3-0)
- [Java 11 Compatibility](https://docs.oracle.com/en/java/javase/11/)
- [Android SDK API 33](https://developer.android.com/about/versions/13)

---

**Last Updated**: December 17, 2025
**Status**: ✅ Ready to Build

### Need Help?
1. Verify Gradle 7.6 with `.\gradlew --version`
2. Check Java 11+ with `java -version`
3. Clear all caches before rebuilding
4. Run on separate lines if PowerShell `&&` fails
