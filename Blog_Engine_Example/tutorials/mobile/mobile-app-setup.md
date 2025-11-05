# Mobile App Setup - Getting Started

> **Document Type**: Tutorial (SAMPLE)  
> **Audience**: Mobile Developers  
> **Time**: 45-60 minutes  
> **Last Updated**: November 5, 2024

---

## What You'll Learn

In this tutorial, you'll set up the Blog Engine mobile app development environment for both iOS and Android using React Native.

By the end, you'll have:
- ✅ React Native development environment configured
- ✅ Blog Engine mobile app running on iOS and Android simulators
- ✅ Basic understanding of the app structure

---

## Prerequisites

- **macOS** (for iOS development) or Linux/Windows (Android only)
- **Node.js** 18+ installed
- **Git** installed
- **Xcode** (for iOS) or **Android Studio** (for Android)
- Basic React knowledge

---

## Step 1: Install React Native CLI

Install the React Native command-line interface:

```bash
npm install -g react-native-cli
```

Verify installation:

```bash
react-native --version
# Should show: react-native-cli: 2.0.1
```

---

## Step 2: Set Up iOS Development (macOS only)

### Install Xcode

1. Download Xcode from the Mac App Store
2. Open Xcode and accept the license agreement
3. Install Command Line Tools:

```bash
xcode-select --install
```

### Install CocoaPods

```bash
sudo gem install cocoapods
```

Verify:

```bash
pod --version
```

---

## Step 3: Set Up Android Development

### Install Android Studio

1. Download from [https://developer.android.com/studio](https://developer.android.com/studio)
2. Install Android Studio
3. During installation, ensure these are selected:
   - Android SDK
   - Android SDK Platform
   - Android Virtual Device

### Configure Environment Variables

Add to `~/.zshrc` or `~/.bash_profile`:

```bash
export ANDROID_HOME=$HOME/Library/Android/sdk
export PATH=$PATH:$ANDROID_HOME/emulator
export PATH=$PATH:$ANDROID_HOME/tools
export PATH=$PATH:$ANDROID_HOME/tools/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools
```

Apply changes:

```bash
source ~/.zshrc  # or ~/.bash_profile
```

---

## Step 4: Clone the Repository

```bash
git clone https://github.com/your-org/blog-engine-mobile.git
cd blog-engine-mobile
```

---

## Step 5: Install Dependencies

```bash
# Install Node dependencies
npm install

# Install iOS dependencies (macOS only)
cd ios && pod install && cd ..
```

---

## Step 6: Configure Environment

Create `.env` file in the project root:

```bash
API_URL=http://localhost:3000/api
API_TIMEOUT=10000
```

---

## Step 7: Run on iOS Simulator (macOS only)

Start the Metro bundler:

```bash
npx react-native start
```

In a new terminal, run the iOS app:

```bash
npx react-native run-ios
```

**Expected Result**: iOS Simulator opens with the Blog Engine app.

---

## Step 8: Run on Android Emulator

### Create Android Virtual Device (AVD)

1. Open Android Studio
2. Go to Tools → AVD Manager
3. Click "Create Virtual Device"
4. Select a device (e.g., Pixel 5)
5. Select a system image (e.g., Android 13)
6. Click "Finish"

### Run the App

Start emulator first:

```bash
# In Android Studio AVD Manager, click the play button
# Or from terminal:
emulator -avd Pixel_5_API_33
```

In a new terminal, run the Android app:

```bash
npx react-native run-android
```

**Expected Result**: Android Emulator shows the Blog Engine app.

---

## Step 9: Explore the App Structure

```
blog-engine-mobile/
├── src/
│   ├── screens/          # App screens
│   │   ├── HomeScreen.js
│   │   ├── PostScreen.js
│   │   └── ProfileScreen.js
│   ├── components/       # Reusable components
│   │   ├── PostCard.js
│   │   └── CommentList.js
│   ├── navigation/       # Navigation setup
│   │   └── AppNavigator.js
│   ├── services/         # API services
│   │   ├── api.js
│   │   └── auth.service.js
│   ├── store/            # Redux state management
│   │   └── slices/
│   └── utils/            # Utility functions
├── ios/                  # iOS native code
├── android/              # Android native code
└── App.js                # App entry point
```

---

## Step 10: Make Your First Change

Edit `src/screens/HomeScreen.js`:

```javascript
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';

const HomeScreen = () => {
  return (
    <View style={styles.container}>
      <Text style={styles.title}>Welcome to Blog Engine! 🚀</Text>
      <Text style={styles.subtitle}>Your changes work!</Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    flex: 1,
    justifyContent: 'center',
    alignItems: 'center',
    backgroundColor: '#fff',
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 10,
  },
  subtitle: {
    fontSize: 16,
    color: '#666',
  },
});

export default HomeScreen;
```

**Save the file** and the app should automatically reload showing your changes!

---

## Verification

You've successfully set up the mobile app if:
- ✅ iOS simulator shows the app (macOS)
- ✅ Android emulator shows the app
- ✅ Hot reload works when you make changes
- ✅ No red error screens

---

## Troubleshooting

### Metro Bundler Won't Start
```bash
# Clear cache and restart
npx react-native start --reset-cache
```

### iOS Build Fails
```bash
# Clean build
cd ios
pod deintegrate
pod install
cd ..
npx react-native run-ios
```

### Android Build Fails
```bash
# Clean gradle
cd android
./gradlew clean
cd ..
npx react-native run-android
```

### App Won't Connect to API
- Check API is running: `http://localhost:3000`
- For Android, use `10.0.2.2` instead of `localhost`
- For iOS simulator, `localhost` should work

---

## Next Steps

Now that your environment is set up:

1. 📱 [Build Your First Mobile Feature](first-mobile-feature.md)
2. 🔐 [Implement Mobile Authentication](../authentication/mobile-auth.md)
3. 📡 [Configure Push Notifications](../../how-to/mobile/push-notifications.md)

---

## Related

- [React Native Documentation](https://reactnative.dev/)
- [Mobile Architecture](../../reference/architecture/mobile-architecture.md)
- [Troubleshooting Guide](../../how-to/troubleshooting/mobile-issues.md)

---

**Congratulations! Your mobile development environment is ready! 🎉**
