# CashFlow — Mobile Setup Guide

## Prerequisites

| Tool | Version | Download |
|------|---------|---------|
| Node.js | 20+ | nodejs.org |
| Xcode | 15+ | Mac App Store (iOS only) |
| Android Studio | latest | developer.android.com |
| CocoaPods | latest | `sudo gem install cocoapods` (iOS only) |

---

## 1. Initial Setup

```bash
# Clone your repo
git clone https://github.com/YOUR_USERNAME/cashflow-app.git
cd cashflow-app

# Install dependencies
npm install

# Add native platforms
npx cap add ios
npx cap add android

# Sync web app into native shells
npx cap sync
```

---

## 2. Run on Android

```bash
# Open in Android Studio
npx cap open android
```
In Android Studio: select a device/emulator → click **Run ▶**

Or build debug APK directly:
```bash
cd android && ./gradlew assembleDebug
# APK at: android/app/build/outputs/apk/debug/app-debug.apk
```

---

## 3. Run on iOS (Mac required)

```bash
# Install CocoaPods dependencies first
cd ios/App && pod install && cd ../..

# Open in Xcode
npx cap open ios
```
In Xcode: select your device/simulator → click **Run ▶**

---

## 4. Update the app after code changes

```bash
# After editing src/index.html
npx cap sync

# Then re-run from Android Studio / Xcode, or use live reload:
npx cap run android --livereload
npx cap run ios     --livereload
```

---

## 5. GitHub Actions — secrets you need to add

Go to: **GitHub repo → Settings → Secrets and variables → Actions**

### Android signing
| Secret | How to get it |
|--------|--------------|
| `ANDROID_SIGNING_KEY_BASE64` | `base64 -i your-keystore.jks` |
| `ANDROID_KEY_ALIAS` | alias you chose when creating the keystore |
| `ANDROID_KEY_PASSWORD` | key password |
| `ANDROID_STORE_PASSWORD` | store password |

Generate a keystore (one-time, keep it safe):
```bash
keytool -genkey -v \
  -keystore cashflow-release.jks \
  -alias cashflow \
  -keyalg RSA -keysize 2048 \
  -validity 10000
```

### iOS signing
| Secret | How to get it |
|--------|--------------|
| `IOS_CERTIFICATE_BASE64` | Export `.p12` from Keychain, then `base64 -i cert.p12` |
| `IOS_CERTIFICATE_PASSWORD` | Password set when exporting |
| `IOS_PROVISIONING_PROFILE_BASE64` | Download from developer.apple.com, then `base64 -i profile.mobileprovision` |
| `KEYCHAIN_PASSWORD` | Any random string (used only on the CI runner) |

---

## 6. Publish to stores

### Google Play
1. Build the signed AAB (CI produces it on every `main` push)
2. Go to **play.google.com/console** → Create app → Internal testing → Upload AAB

### Apple App Store
1. Build the IPA (CI produces it on every `main` push)
2. Go to **appstoreconnect.apple.com** → New App → TestFlight → Upload IPA
3. Or uncomment the `altool` step in `build-ios.yml` to automate this

---

## 7. Customise your app ID

Edit `capacitor.config.ts`:
```ts
appId: 'com.yourname.cashflow',   // ← must match your Play/App Store listing
appName: 'CashFlow',
```

This must be set **before** you first publish to the stores — it cannot be changed afterwards.

---

## 8. Adding native features (later)

Capacitor plugins are drop-in — install and sync:
```bash
# Push notifications
npm install @capacitor/push-notifications && npx cap sync

# Biometric auth
npm install capacitor-native-biometric && npx cap sync

# Touch / Face ID unlock
npm install capacitor-face-id && npx cap sync
```
Then call them from `src/index.html` using `window.Capacitor.Plugins.*`.
