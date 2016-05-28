Service Workers
===============
A service worker is a script that runs in your browser the background separate from a web page.


Pre-caching with a service worker
---------------------------------
A short guide to setting up a service worker inside an existing application to pre-cache resources to optimize your web application's initial load speed.


1. Make the service worker available in the same directory it will be needed. This is because the scope of the service worker is defined by the directory the worker exists.
  * Install app shell resources during the `install` phase.
  * Clear old cache files during the `activate` phase.
  * Intercept and modify main application requests by listening to events like `fetch`.
2. Check for support then register the worker with the browser (if its supported).


_Warning: The following is not a production ready solution. In a production environment use a tool like [sw-precache][1] for finely tuned control._

When a browser registers a service worker for the first time (on the users first visit to the page) an `install` event is triggered. At this moment you can take advantage and cache all assets needed for the application shell to render.
```javascript
self.addEventListener('install', function(event) {  
  // Store application shell files
  event.waitUntil(
    caches.open(cacheName).then(function(cache) {
      console.log('[ServiceWorker] Caching app shell');
      return cache.addAll([...filePaths]);
    })
  );
});
```

You can also connect to the `activate` event that is triggered after the _install_ process is completed on all pages that are loaded. It is generally a good time to clean up old caches and other things associated with the previous version of your service worker when this event fires.
```javascript
self.addEventListener('activate', function(event) {  
  // Do something when the activate event is fired like clear unused cache files.
  console.log('[ServiceWorker] Activate');  
  event.waitUntil(  
    caches.keys().then(function(keyList) {  
      return Promise.all(keyList.map(function(key) {  
        console.log('[ServiceWorker] Removing old cache', key);  
        if (key !== cacheName) {  
          return caches.delete(key);  
        }  
      }));  
    })  
  );
});
```

You can intercept requests made by your main javascript application with your service worker. With this in mind, you can intercept `fetch` events made by your main application and do something interesting to the request. In the example below, intercept `fetch` requests and check if the requested resource already exists in the cache and if not, request the resource over the network.
```javascript
self.addEventListener('fetch', function(event) {
  console.log('[ServiceWorker] Fetch', event.request.url);
  // Do something interesting with the fetch here
  event.respondWith(
    caches.match(event.request).then(function(response) {
      if (response) {
        console.log('Found response in cache:', response);

        return response;
      }
      console.log('No response found in cache. About to fetch from network...');

      return fetch(event.request).then(function(response) {
        console.log('Response from network is:', response);

        return response;
      }).catch(function(error) {
        console.error('Fetching failed:', error);

        throw error;
      })
    })
  );
});
```

Registering the service worker inside your regular application javascript is easy. Just make sure the browser supports service workers and use the `navigator.serviceWorker` api to register your worker.
```javascript
// Main application javascript file
if('serviceWorker' in navigator) {
  navigator.serviceWorker  
           .register('/service-worker.js')
           .then(function() { console.log('Service Worker Registered'); });
}
```


Caching app data with a service worker
--------------------------------------
Implement a __cache first then network__ strategy using a web service worker to quickly and reliably request and present data to a user.

1. Intercept network requests and cache responses
2. Modify your application code to request data resources from the cache and network at the same time


Inside the `fetch` event handler in the _service-worker_ create a switch to handle requests to data urls differently than other resources. If its a data url, _cache_ and return the response.
```javascript
const DATA_DOMAIN = 'localhost:8081';
function getDomain = (url) => url.split('/')[2];  // strips http and paths from url
self.addEventListener('fetch', function(event) {
  switch (getDomain(event.request.url)) {
    case DATA_DOMAIN:
      // implement cache first then network strategy for data urls
      event.respondWith(
        // fetch network data
        fetch(event.request)  
          .then(function(response) {  
            return caches.open(dataCacheName).then(function(cache) {
              // cache network data
              cache.put(event.request.url, response.clone());  
              console.log('[ServiceWorker] Fetched&Cached Data');  
              return response;  
            });  
          })  
      );
    default:
      // handle all other requests
      event.respondWith(  
        caches.match(event.request).then(function(response) {  
          return response || fetch(event.request);  
        })  
      );  
  }
});
```

In your main application code you need to modify your requests to data urls so that they send requests to the cache and the network asynchronously. Two asynchronous requests are made, one to the cache and one to the network. Make sure the `caches` object exists before sending a request to it. If it doesn't, you will defer to the network request anyway :)
```javascript
// fetch from cache
if ('caches' in window) {
  caches.match(url).then(function(response) {  
    if (response) {  
      response.json().then(function(json) {  
        console.log('updated from cache');  
        // display the data or something
      });  
    }
  });  
}
// fetch from network
fetch(url).then(function(response) {
  return response.json().then(function(json) {
    // update display with network data
  });
})
```

In most circumstances the cache will return before the network request. In the edge case that this doesn't happen, add a flag so that you don't overwrite your network data with the cached data. _The network is assumed to be the single source of truth._ It will be providing the most updated information so we don't want to overwrite it with cache data.
```javascript
var requestPendingFlag;  // our flag to check if network request is pending
// fetch from cache
if ('caches' in window) {
  requestPendingFlag = true;  // we're sending the network request at the same time so we set the flag to true
  caches.match(url).then(function(response) {  
    if (response) {  
      response.json().then(function(json) {  
        // Only update if the network request is still pending, otherwise the network request has already returned and provided the latest data.  
        if (requestPendingFlag) {  
          console.log('updated from cache');  
          // display the data or something
        }  
      });  
    }  
  });  
}
// fetch from network
fetch(url).then(function(response) {
  return response.json().then(function(json) {
    requestPendingFlag = false;  // Network request has returned so set flag to false preventing cache data from overwriting the network data
    // update display with network data
  });
})
```


At this point you can turn off your server and reload the page. If your app was cached properly it should still run!


Useful Tools
------------

* [sw-precache][1]: Module for generating a service worker that precaches resources during your applications build step.
* [chrome://serviceworker-internals/][2]: Chrome dev tool for managing and inspecting service workers.



<!--References-->
[1]: https://github.com/GoogleChrome/sw-precache "Service worker precache github"
[2]: chrome://serviceworker-internals/ "Chrome’s Service Worker Internals page"
