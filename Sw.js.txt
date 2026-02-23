// Service Worker for Athlete Dashboard PWA
const CACHE_NAME = ‘athlete-dashboard-v1’;
const urlsToCache = [
‘/’,
‘/index.html’,
‘/manifest.json’,
‘https://cdn.tailwindcss.com’,
‘https://unpkg.com/react@18/umd/react.production.min.js’,
‘https://unpkg.com/react-dom@18/umd/react-dom.production.min.js’,
‘https://unpkg.com/@babel/standalone/babel.min.js’
];

// Install - cache resources
self.addEventListener(‘install’, (event) => {
event.waitUntil(
caches.open(CACHE_NAME)
.then((cache) => cache.addAll(urlsToCache))
.then(() => self.skipWaiting())
);
});

// Activate - clean up old caches
self.addEventListener(‘activate’, (event) => {
event.waitUntil(
caches.keys().then((cacheNames) => {
return Promise.all(
cacheNames.map((cacheName) => {
if (cacheName !== CACHE_NAME) {
return caches.delete(cacheName);
}
})
);
}).then(() => self.clients.claim())
);
});

// Fetch - serve from cache, fallback to network
self.addEventListener(‘fetch’, (event) => {
event.respondWith(
caches.match(event.request)
.then((response) => {
if (response) {
return response;
}

```
    const fetchRequest = event.request.clone();

    return fetch(fetchRequest).then((response) => {
      if (!response || response.status !== 200 || response.type !== 'basic') {
        return response;
      }

      const responseToCache = response.clone();

      caches.open(CACHE_NAME)
        .then((cache) => {
          cache.put(event.request, responseToCache);
        });

      return response;
    }).catch(() => {
      return caches.match('/index.html');
    });
  })
```

);
});