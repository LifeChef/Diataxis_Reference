# Handle Offline Mode in Mobile App

> **Document Type**: How-To Guide (SAMPLE)  
> **Audience**: Mobile Developers  
> **Difficulty**: Advanced  
> **Last Updated**: November 5, 2024

---

## Problem

You need to implement offline support in the Blog Engine mobile app, allowing users to browse cached content and queue actions when offline.

---

## Prerequisites

- Mobile app running
- Redux store configured
- Understanding of AsyncStorage

---

## Solution

### Step 1: Install Dependencies

```bash
npm install @react-native-async-storage/async-storage \
  @react-native-community/netinfo \
  redux-persist
```

### Step 2: Configure Redux Persist

Create `src/store/persist.config.js`:

```javascript
import AsyncStorage from '@react-native-async-storage/async-storage';
import { persistReducer, persistStore } from 'redux-persist';
import rootReducer from './reducers';

const persistConfig = {
  key: 'root',
  storage: AsyncStorage,
  whitelist: ['posts', 'user', 'settings'], // Only persist these
  blacklist: ['network', 'ui'] // Don't persist these
};

export const persistedReducer = persistReducer(persistConfig, rootReducer);
```

Update `src/store/index.js`:

```javascript
import { configureStore } from '@reduxjs/toolkit';
import { persistStore } from 'redux-persist';
import { persistedReducer } from './persist.config';

export const store = configureStore({
  reducer: persistedReducer,
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        ignoredActions: ['persist/PERSIST']
      }
    })
});

export const persistor = persistStore(store);
```

### Step 3: Monitor Network Status

Create `src/services/network.service.js`:

```javascript
import NetInfo from '@react-native-community/netinfo';
import { store } from '../store';
import { setOnlineStatus } from '../store/slices/networkSlice';

class NetworkService {
  init() {
    // Subscribe to network state updates
    this.unsubscribe = NetInfo.addEventListener(state => {
      console.log('Network status:', state.isConnected);
      store.dispatch(setOnlineStatus(state.isConnected));
      
      if (state.isConnected) {
        this.syncQueuedActions();
      }
    });
  }

  async checkConnection() {
    const state = await NetInfo.fetch();
    return state.isConnected;
  }

  async syncQueuedActions() {
    // Sync queued actions when back online
    const queuedActions = await this.getQueuedActions();
    
    for (const action of queuedActions) {
      try {
        await this.executeAction(action);
        await this.removeFromQueue(action.id);
      } catch (error) {
        console.error('Failed to sync action:', error);
      }
    }
  }

  async getQueuedActions() {
    const queue = await AsyncStorage.getItem('action_queue');
    return queue ? JSON.parse(queue) : [];
  }

  async addToQueue(action) {
    const queue = await this.getQueuedActions();
    queue.push({ ...action, id: Date.now(), timestamp: new Date() });
    await AsyncStorage.setItem('action_queue', JSON.stringify(queue));
  }

  async removeFromQueue(actionId) {
    const queue = await this.getQueuedActions();
    const filtered = queue.filter(a => a.id !== actionId);
    await AsyncStorage.setItem('action_queue', JSON.stringify(filtered));
  }

  async executeAction(action) {
    switch (action.type) {
      case 'CREATE_POST':
        return await postService.create(action.payload);
      case 'UPDATE_POST':
        return await postService.update(action.payload.id, action.payload);
      case 'CREATE_COMMENT':
        return await commentService.create(action.payload);
      default:
        console.warn('Unknown action type:', action.type);
    }
  }
}

export default new NetworkService();
```

### Step 4: Create Offline-Aware API Service

Create `src/services/api-offline.service.js`:

```javascript
import networkService from './network.service';
import apiService from './api.service';

class OfflineAPIService {
  async request(config) {
    const isOnline = await networkService.checkConnection();
    
    if (!isOnline) {
      // Handle offline request
      return this.handleOfflineRequest(config);
    }
    
    return apiService.request(config);
  }

  handleOfflineRequest(config) {
    if (config.method === 'GET') {
      // Return cached data for GET requests
      return this.getCachedData(config.url);
    } else {
      // Queue write operations
      return this.queueAction(config);
    }
  }

  async getCachedData(url) {
    const cacheKey = `cache_${url}`;
    const cached = await AsyncStorage.getItem(cacheKey);
    
    if (cached) {
      return JSON.parse(cached);
    }
    
    throw new Error('No cached data available for offline access');
  }

  async cacheData(url, data) {
    const cacheKey = `cache_${url}`;
    await AsyncStorage.setItem(cacheKey, JSON.stringify(data));
  }

  async queueAction(config) {
    await networkService.addToQueue({
      type: config.queueType,
      payload: config.data,
      url: config.url,
      method: config.method
    });
    
    // Return optimistic response
    return {
      ...config.data,
      _pending: true,
      _queuedAt: new Date()
    };
  }
}

export default new OfflineAPIService();
```

### Step 5: Update Post Service

```javascript
import offlineAPIService from './api-offline.service';

class PostService {
  async getPosts() {
    try {
      const response = await offlineAPIService.request({
        method: 'GET',
        url: '/posts'
      });
      
      // Cache successful responses
      if (response.data) {
        await offlineAPIService.cacheData('/posts', response.data);
      }
      
      return response.data;
    } catch (error) {
      // If online request fails, try cached data
      return await offlineAPIService.getCachedData('/posts');
    }
  }

  async createPost(postData) {
    return await offlineAPIService.request({
      method: 'POST',
      url: '/posts',
      data: postData,
      queueType: 'CREATE_POST'
    });
  }
}
```

### Step 6: Show Offline Indicator

Create `src/components/OfflineIndicator.js`:

```javascript
import React from 'react';
import { View, Text, StyleSheet } from 'react-native';
import { useSelector } from 'react-redux';

const OfflineIndicator = () => {
  const isOnline = useSelector(state => state.network.isOnline);
  const queuedActionsCount = useSelector(state => state.network.queuedActionsCount);

  if (isOnline) return null;

  return (
    <View style={styles.container}>
      <Text style={styles.text}>
        📡 Offline Mode {queuedActionsCount > 0 && `(${queuedActionsCount} pending)`}
      </Text>
    </View>
  );
};

const styles = StyleSheet.create({
  container: {
    backgroundColor: '#ff9800',
    padding: 10,
    alignItems: 'center'
  },
  text: {
    color: '#fff',
    fontWeight: 'bold'
  }
});

export default OfflineIndicator;
```

### Step 7: Initialize in App

```javascript
import React, { useEffect } from 'react';
import { PersistGate } from 'redux-persist/integration/react';
import { store, persistor } from './src/store';
import networkService from './src/services/network.service';
import OfflineIndicator from './src/components/OfflineIndicator';

const App = () => {
  useEffect(() => {
    networkService.init();
  }, []);

  return (
    <Provider store={store}>
      <PersistGate loading={<LoadingScreen />} persistor={persistor}>
        <OfflineIndicator />
        <NavigationContainer>
          {/* Your app navigation */}
        </NavigationContainer>
      </PersistGate>
    </Provider>
  );
};
```

---

## Verification

1. Launch app while online
2. Browse posts (data cached)
3. Turn off network connection
4. Browse cached posts (should still work)
5. Create new post (should be queued)
6. Turn on network connection
7. Queued post should sync automatically

---

## Best Practices

✅ Cache critical data (posts, user profile)  
✅ Show offline indicator  
✅ Queue write operations  
✅ Sync automatically when back online  
✅ Handle sync conflicts gracefully  
✅ Provide feedback on pending actions

---

## Troubleshooting

**Data not persisting:**
- Check AsyncStorage permissions
- Verify Redux Persist configuration
- Check whitelist/blacklist settings

**Sync not happening:**
- Verify network listener is active
- Check queued actions in AsyncStorage
- Review sync logic in network service

---

## Related

- [Redux Persist Documentation](https://github.com/rt2zz/redux-persist)
- [NetInfo Documentation](https://github.com/react-native-netinfo/react-native-netinfo)
- [Mobile App Setup](../../tutorials/mobile/mobile-app-setup.md)
