# Getting Started

Koti Cloud apps are built with the help of Koti Cloud SDK. The SDK currently supports Vanilla JS and Svelte.js. While there are some helpful reusable UI components for Svelte in the SDK, the core is Vanilla JS and you can easily use any other framework of choice.

You can either build your app from scratch or integrate an existing app into Koti Cloud. Either way, you'll have to go through the same initial steps to get you started.

The first thing you need to do is to download [Koti Cloud SDK](https://github.com/koticloud/sdk). Create a folder called `sdk` next to your app's folder. Then create a folder with the latest SDK version as the name inside `sdk`, e.g. `v0.4.x` (replace the the specific patch version with `x`). Your folder structure should look like this:

```
.
..
YourApp
sdk
└─── v0.4.x
```

After creating the folders, navigate to the SDK folder and clone the SDK repo from GitHub:

```bash
cd sdk/v0.4.x
git clone https://github.com/koticloud/sdk.git .
```

## Using Koti Cloud App Templates

If you're building a Svelte app, use the [Svelte App Template](https://github.com/koticloud/app-template-svelte). If you're using another framework or no framework at all, use the [Vanilla JS App Template](https://github.com/koticloud/app-template). Follow the instructions from the README to get started.

After pulling the code, search it for the `../sdk/` string. You should find multiple SDK imports (JS and CSS/SASS) with paths looking like `'../../../sdk/v0.4.x/sdk.js'`. Replace `v0.4.x` with your SDK version in the same format (without specifying the specific patch version).

Do read the following section too as it will give you a better understanding of what Koti Cloud app requirements are.

## Building an App From Scratch or Adapting Your Existing Web App or PWA

You are almost not limited with the way you organize your files or build your app, but there are a few things your app has to have or implement.

### 1. You app has to be a Progressive Web App (PWA)

If you don't know what a PWA is - please do your research. Like any other PWA, your PWA has to have a Service Worker, a manifest file, a set of icons, certain meta tags in the entry HTML file.

The main or entry file of your app should be located at the root folder of the app and be called `index.html`.

We strongly encourage you to follow our service worker format. It's a very simple format that follows the "Cache, falling back to network" caching strategy. You won't need to worry about updating your app/cache in the future - Koti Cloud handles that for you!

```javascript
const cacheName = 'koti-cloud-app';

const filesToCache = [
    // NOTE: Edit this list accordingly
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
];

// When an app is installed - start the service worker and cache all of the
// app's content
self.addEventListener('install', function (e) {
    e.waitUntil(
        caches.open(cacheName).then(function (cache) {
            return cache.addAll(filesToCache);
        })
    );
});

// Serving content
self.addEventListener('fetch', function (e) {
    e.respondWith(
        caches.match(e.request).then(function (response) {
            // "Cache, falling back to network" strategy - return cached files
            // if available, otherwise fetch from network
            return response || fetch(e.request);
        })
            .catch(() => {
                // Ignore exceptions
            })
    );
});
```

### 2. You are required to import and initialize core SDK components

First, the JavaScript. Please call the following as early as possible in your code (please read the comments in the code):

```javascript
// The number of '../' in the path will depend on where you put your files.
// The SDK should always be at the `../sdk/v0.4.x/` folder relative to your
// project's root though.
import { App } from '../../../sdk/v0.4.x/sdk.js';

// Don't call this global variable `app` as it might interfere with some
// frameworks' global vars. For example, this would be true for Svelte
window.sdk = new App();

sdk.init({
    // Path to your service worker, relative to the root directory of your
    // project
    serviceWorker: 'sw.js',
    cacheables: {
        untilUpdate: [
            // NOTE: Edit as needed. This list should be identical to the list
            // of cacheable files/paths you specify in your service worker. This
            // is crucial as this list is used to handle app updates.
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
    // Remove this if your app will not persists any user/app data. This is the
    // name of your database and it won't interfere with other apps' databases
    // no matter what you call it
    db: 'koti-cloud-app-db',
});

// Check for updates (async)
sdk.checkForUpdates();
```

Next you need to import the SDK's SASS file somewhere in your code:

```scss
// The number of '../' in the path will depend on where you put your files.
// The SDK should always be at the `../sdk/v0.4.x/` folder relative to your
// project's root though.
@import '../../../sdk/v0.4.x/sass/ui';
```

The SDK applies certain styling to the `<body>` element and injects a panel at the top of the page. The `<body>` will become a `display: flex` and a `flex-direction: column` element. Give `flex: 1` to your main app element (wrapper) so that it takes all the available space below the top panel.

### 3. What's next?

The steps above are enough for a very basic setup. Keep reading this documentation to learn what else Koti Cloud SDK has to offer.

You should also take a look at the app templates mentioned above. We recommend them as a starting point.

### 4. The most important limitation

The philosophy of Koti Cloud is: one account - all your apps and data. In other words, a single Koti Cloud account should be enough for a user to access all their app data.

Therefore, you are not allowed to require your app users to register or log in anywhere besides their Koti Cloud account, especially in order to access their app data. All user data should be stored at Koti Cloud using the SDK's DB module.