# App Module

<ul class="nav">
    <li><a href="#introduction">Introduction</a></li>
    <li><a href="#available-methods">Available Methods</a></li>
</ul>

## Introduction <a name="introduction"></a>

The `App` module is the app's main module. This is the module you have to import and initialize in any Koti Cloud App.

```javascript
import { App } from '../../../sdk/v0.4.x/sdk.js';

window.sdk = new App();

sdk.init({ ... });
```

The `App` class instance, called `sdk` here and everywhere else in this documentation, also gives you access to some other modules of the SDK.

## Available Methods <a name="available-methods"></a>

### `void init(options)`

Initialize a Koti Cloud app with the provided options.

```javascript
sdk.init({
    // [REQUIRED]: Path to your service worker, relative to the root directory
    // of your project
    serviceWorker: 'sw.js',
    // [REQUIRED]: list of files to be cached for the app to work offline.
    // Should be the same as the list of files/path in your service worker.
    cacheables: {
        untilUpdate: [
            './',
            './manifest.json',
            './bin/app.js',
            './bin/app.css',
            // App icons
            './assets/icons/favicon.ico',
            './assets/icons/icon-128.png',
            './assets/icons/icon-144.png',
            './assets/icons/icon-152.png',
            './assets/icons/icon-192.png',
            './assets/icons/icon-256.png',
            './assets/icons/icon-512.png',
        ],
    },
    // [OPTIONAL]: specify an app's local DB name if your app will persist any
    // user/app data. The name could be anything ant it won't interfere with
    // other apps' databases.
    db: 'koti-cloud-app-db',
});
```

### `bool isInitialized()`

Returns `true` if the app has been initialized.

### `bool isOnline()`

Returns `true` if the app is online (if there's Internet connection).

### `Promise hasUpdates()`

Returns `true` if there's a newer version of the app than the installed (cached) one. Only works when online.

### `void async checkForUpdates()`

Checks for updates by calling `hasUpdates()`, waits for the result, shows an update prompt to the user if there's a newer version of the app than the installed (cached) one. Only works when online.

### `void update()`

Manually initialize an app update process. Normally you should call `checkForUpdates()` instead.

### `bool isAuthenticated()`

Returns `true` if the current app user is authenticated at Koti Cloud. **This is pretty unreliable.** The authenticated state is set to `true` by default even if the user is not really authenticated. It is set to `false` only when the app uses Koti Cloud DB and after the initial DB sync has failed due to the user being unauthenticated at Koti Cloud.

### `string appName()`

Returns the current app name. Might return `null` if called too early. Might return `null` or the old value when offline.

Can be called both on the `App` class instance and statically.

### `void setTitle(title = null)`

Set current app title, which is displayed in the Koti Cloud panel at the top of the app after the app name.

### `void setTitle(title = null)`

Set current app title, which is displayed in the Koti Cloud panel at the top of the app after the app name.