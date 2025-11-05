# Configure Push Notifications

> **Document Type**: How-To Guide (SAMPLE)  
> **Audience**: Mobile Developers, Backend Developers  
> **Difficulty**: Advanced  
> **Last Updated**: November 5, 2024

---

## Problem

You need to implement push notifications for the Blog Engine mobile app using Firebase Cloud Messaging (FCM) for both iOS and Android.

---

## Prerequisites

- Firebase project created
- Mobile app running (see [Mobile App Setup](../../tutorials/mobile/mobile-app-setup.md))
- APNs certificate (for iOS)

---

## Solution

### Step 1: Install Dependencies

```bash
npm install @react-native-firebase/app @react-native-firebase/messaging
```

### Step 2: Configure Firebase (iOS)

1. Download `GoogleService-Info.plist` from Firebase Console
2. Add to `ios/` directory
3. Update `ios/Podfile`:

```ruby
pod 'Firebase/Messaging'
```

4. Install pods:

```bash
cd ios && pod install && cd ..
```

### Step 3: Configure Firebase (Android)

1. Download `google-services.json` from Firebase Console
2. Add to `android/app/` directory
3. Update `android/build.gradle`:

```gradle
buildscript {
  dependencies {
    classpath 'com.google.gms:google-services:4.3.15'
  }
}
```

4. Update `android/app/build.gradle`:

```gradle
apply plugin: 'com.google.gms.google-services'
```

### Step 4: Request Permission

Create `src/services/notifications.service.js`:

```javascript
import messaging from '@react-native-firebase/messaging';

class NotificationService {
  async requestPermission() {
    const authStatus = await messaging().requestPermission();
    const enabled =
      authStatus === messaging.AuthorizationStatus.AUTHORIZED ||
      authStatus === messaging.AuthorizationStatus.PROVISIONAL;

    if (enabled) {
      console.log('Authorization status:', authStatus);
    }
    
    return enabled;
  }

  async getToken() {
    const token = await messaging().getToken();
    console.log('FCM Token:', token);
    
    // Send token to your backend
    await this.sendTokenToServer(token);
    
    return token;
  }

  async sendTokenToServer(token) {
    await fetch('https://api.blogengine.com/users/device-token', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${userToken}`
      },
      body: JSON.stringify({ deviceToken: token, platform: Platform.OS })
    });
  }

  setupNotificationListeners() {
    // Foreground messages
    messaging().onMessage(async remoteMessage => {
      console.log('Foreground notification:', remoteMessage);
      // Show local notification
      this.displayNotification(remoteMessage);
    });

    // Background messages
    messaging().setBackgroundMessageHandler(async remoteMessage => {
      console.log('Background notification:', remoteMessage);
    });

    // Notification opened app
    messaging().onNotificationOpenedApp(remoteMessage => {
      console.log('Notification opened app:', remoteMessage);
      // Navigate to relevant screen
      this.handleNotificationNavigation(remoteMessage);
    });

    // App opened from quit state
    messaging()
      .getInitialNotification()
      .then(remoteMessage => {
        if (remoteMessage) {
          console.log('App opened from notification:', remoteMessage);
          this.handleNotificationNavigation(remoteMessage);
        }
      });
  }

  displayNotification(remoteMessage) {
    // Implementation depends on your UI library
    Alert.alert(
      remoteMessage.notification.title,
      remoteMessage.notification.body
    );
  }

  handleNotificationNavigation(remoteMessage) {
    const { type, postId } = remoteMessage.data;
    
    switch (type) {
      case 'new_post':
        navigate('Post', { id: postId });
        break;
      case 'new_comment':
        navigate('Post', { id: postId, focusComments: true });
        break;
    }
  }
}

export default new NotificationService();
```

### Step 5: Initialize in App

Update `App.js`:

```javascript
import React, { useEffect } from 'react';
import notificationService from './src/services/notifications.service';

const App = () => {
  useEffect(() => {
    // Request permission
    notificationService.requestPermission()
      .then(enabled => {
        if (enabled) {
          // Get token and send to server
          notificationService.getToken();
          
          // Setup listeners
          notificationService.setupNotificationListeners();
        }
      });
  }, []);

  return (
    // Your app component
  );
};

export default App;
```

### Step 6: Backend Integration

Backend endpoint to send notifications:

```javascript
const admin = require('firebase-admin');

admin.initializeApp({
  credential: admin.credential.cert(serviceAccount)
});

async function sendPushNotification(deviceToken, notification) {
  const message = {
    notification: {
      title: notification.title,
      body: notification.body
    },
    data: {
      type: notification.type,
      postId: notification.postId
    },
    token: deviceToken
  };

  try {
    const response = await admin.messaging().send(message);
    console.log('Notification sent:', response);
  } catch (error) {
    console.error('Error sending notification:', error);
  }
}
```

### Step 7: Test Notifications

Send test notification from Firebase Console or using the backend:

```javascript
await sendPushNotification(userDeviceToken, {
  title: 'New Post!',
  body: 'Check out the latest blog post',
  type: 'new_post',
  postId: '123'
});
```

---

## Verification

1. Request notification permission
2. App receives token and sends to backend
3. Send test notification from Firebase Console
4. Notification appears on device
5. Tapping notification opens relevant screen

---

## Troubleshooting

**iOS not receiving notifications:**
- Verify APNs certificate uploaded to Firebase
- Check provisioning profile includes Push Notifications
- Test on physical device (simulator doesn't support push)

**Android not receiving notifications:**
- Verify `google-services.json` is in correct location
- Check Firebase project matches package name
- Ensure Google Play Services installed on device/emulator

**Token not being sent to server:**
- Check network connectivity
- Verify API endpoint is correct
- Review authentication token

---

## Related

- [Firebase Cloud Messaging Docs](https://firebase.google.com/docs/cloud-messaging)
- [Notification Service Design](../../docs/architecture/notification-service.md)
- [Mobile App Setup](../../tutorials/mobile/mobile-app-setup.md)
