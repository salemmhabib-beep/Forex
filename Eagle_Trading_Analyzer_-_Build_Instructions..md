# Eagle Trading Analyzer - Build Instructions

This document provides detailed instructions for building the APK file for Android 8 and above.

## Prerequisites

### Option A: Using Android Studio (Recommended for Beginners)

1. **Download Android Studio**
   - Visit: https://developer.android.com/studio
   - Download the latest version for your OS (Windows, Mac, or Linux)

2. **Install Android SDK**
   - During Android Studio setup, install SDK for Android 8.0 (API 26) and Android 14 (API 34)
   - Install Android Build Tools version 34.0.0 or higher

3. **Install Java Development Kit (JDK)**
   - Download JDK 11 or higher from: https://www.oracle.com/java/technologies/downloads/
   - Or use OpenJDK: `sudo apt install openjdk-11-jdk` (Linux)

### Option B: Using Command Line (For Advanced Users)

1. **Install Java**
   ```bash
   # Ubuntu/Debian
   sudo apt update && sudo apt install openjdk-11-jdk
   
   # macOS
   brew install openjdk@11
   
   # Windows: Download from Oracle or use Chocolatey
   choco install openjdk11
   ```

2. **Install Android SDK Command-line Tools**
   - Download from: https://developer.android.com/studio#command-tools
   - Extract and set up ANDROID_HOME environment variable

3. **Install Required SDK Packages**
   ```bash
   sdkmanager "platforms;android-34" "build-tools;34.0.0" "platform-tools"
   ```

## Building the APK

### Method 1: Android Studio GUI (Easiest)

1. **Open the Project**
   - Launch Android Studio
   - File → Open → Select `eagle-trading-analyzer` folder
   - Wait for Gradle sync to complete

2. **Build APK**
   - Click: Build → Build Bundle(s) / APK(s) → Build APK(s)
   - Wait for build to complete (2-5 minutes)
   - You'll see a notification: "Build successful"

3. **Locate the APK**
   - Click: "Locate" in the build notification
   - Or navigate to: `app/build/outputs/apk/debug/app-debug.apk`

4. **Install on Device**
   - Connect Android device via USB
   - Enable USB Debugging on device (Settings → Developer Options → USB Debugging)
   - Click: Run → Run 'app'
   - Select your device and click OK

### Method 2: Command Line (Linux/Mac/Windows with Git Bash)

1. **Navigate to Project**
   ```bash
   cd eagle-trading-analyzer
   ```

2. **Build Debug APK**
   ```bash
   ./gradlew assembleDebug
   ```
   
   Or on Windows:
   ```bash
   gradlew.bat assembleDebug
   ```

3. **Wait for Build Completion**
   - First build may take 5-10 minutes (downloading dependencies)
   - Subsequent builds are faster (1-2 minutes)

4. **Locate APK**
   ```bash
   ls app/build/outputs/apk/debug/
   # Output: app-debug.apk
   ```

5. **Install on Device**
   ```bash
   adb install app/build/outputs/apk/debug/app-debug.apk
   ```

### Method 3: Build Release APK (Optimized)

For production use, build a release APK:

```bash
./gradlew assembleRelease
```

Output: `app/build/outputs/apk/release/app-release-unsigned.apk`

**Note**: Release APK requires signing. See "Signing the APK" section below.

## Signing the APK (For Release)

### Create a Keystore

```bash
keytool -genkey -v -keystore eagle-trading.keystore -keyalg RSA -keysize 2048 -validity 10000 -alias eagle-trading
```

Follow prompts to set password and key details.

### Sign the APK

```bash
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 \
  -keystore eagle-trading.keystore \
  app/build/outputs/apk/release/app-release-unsigned.apk \
  eagle-trading
```

### Align the APK

```bash
zipalign -v 4 app/build/outputs/apk/release/app-release-unsigned.apk \
  app/build/outputs/apk/release/app-release-signed.apk
```

Result: `app-release-signed.apk` is ready for distribution.

## Troubleshooting Build Issues

### Issue: "ANDROID_HOME not set"

**Solution**:
```bash
# Linux/Mac
export ANDROID_HOME=$HOME/Android/Sdk
export PATH=$PATH:$ANDROID_HOME/tools:$ANDROID_HOME/platform-tools

# Windows (set environment variable)
ANDROID_HOME = C:\Users\YourUsername\AppData\Local\Android\Sdk
```

### Issue: "Gradle sync failed"

**Solution**:
1. File → Invalidate Caches → Invalidate and Restart
2. Delete `.gradle` folder and `build` folder
3. Retry sync

### Issue: "SDK not found"

**Solution**:
1. Open Android Studio
2. Tools → SDK Manager
3. Install missing SDK versions (API 26, 34)

### Issue: "Java version mismatch"

**Solution**:
```bash
# Check Java version
java -version

# Should output Java 11 or higher
# If not, install JDK 11 or set JAVA_HOME
export JAVA_HOME=/path/to/jdk-11
```

### Issue: "Build takes too long"

**Solution**:
- First build is slow (downloading dependencies)
- Subsequent builds are faster
- Increase Gradle heap: Add to `gradle.properties`:
  ```
  org.gradle.jvmargs=-Xmx4096m
  ```

## Installation on Device

### Via USB Cable (Recommended)

1. **Enable USB Debugging**
   - Settings → About Phone → Tap Build Number 7 times
   - Settings → Developer Options → Enable USB Debugging

2. **Connect Device**
   - Connect via USB cable
   - Allow USB debugging prompt on device

3. **Install APK**
   ```bash
   adb install app/build/outputs/apk/debug/app-debug.apk
   ```

### Via File Transfer

1. **Copy APK to Device**
   - Connect device via USB
   - Copy `app-debug.apk` to device storage
   - Disconnect

2. **Install on Device**
   - Open Files app on device
   - Navigate to downloaded APK
   - Tap to install
   - Grant permissions

### Via Android Studio

1. Run → Run 'app'
2. Select connected device
3. Click OK

## Verifying Installation

```bash
# Check if app is installed
adb shell pm list packages | grep eagle

# Output should show: com.eagle.trading.analyzer

# Launch the app
adb shell am start -n com.eagle.trading.analyzer/.ui.MainActivity
```

## Build Output Files

After successful build, you'll find:

```
app/build/outputs/
├── apk/
│   ├── debug/
│   │   └── app-debug.apk          ← Use this for testing
│   └── release/
│       └── app-release-unsigned.apk
├── bundle/
│   └── release/
│       └── app-release.aab        ← For Google Play Store
```

## Performance Tips

- **First Build**: 5-10 minutes (downloading dependencies)
- **Incremental Build**: 1-2 minutes (only changed files)
- **Clean Build**: 3-5 minutes
  ```bash
  ./gradlew clean assembleDebug
  ```

## Next Steps

1. **Test on Device**
   - Open Eagle Trading Analyzer
   - Grant screen recording permission
   - Start analyzing trading charts

2. **Customize**
   - Modify colors in `app/src/main/res/values/colors.xml`
   - Change app name in `app/src/main/res/values/strings.xml`
   - Update icon in `app/src/main/res/mipmap-hdpi/`

3. **Distribute**
   - Upload signed APK to Google Play Store
   - Or distribute via direct download link

## Support

For build issues:
1. Check Android Studio console for error messages
2. Review the Troubleshooting section above
3. Ensure all prerequisites are installed
4. Try `./gradlew clean build` to rebuild from scratch

---

**Last Updated**: May 2026  
**Gradle Version**: 8.1.0  
**Target Android**: 8.0 - 14 (API 26-34)
