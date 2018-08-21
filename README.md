# sw_10

---

## Description

This repo is to my notes on service workers.


--- 




### First register a service worker in the root app directory

Index.html

```html
    <script>
      (function() {
        'use strict';
        if (!('serviceWorker' in navigator)) {
          console.log('Service worker not supported');
          return;
        }
        navigator.serviceWorker.register('service-worker.js')
          .then(function () {
            console.log('Registered');
          })
          .catch(function (error) {
            console.log('Registration failed:', error);
          });
      })();
    </script>
```

Note that `.register()` takes in file as the first argument than optional
scope, telling where the sw can target what files.


```javascript
navigator.serviceWorker.register('/service-worker.js', {
  scope: '/app/'
});

```


### Second add event listeners on the install event of the sw lifecycle 

The Install Phase is where precache can occur.

After the Install Event there is a waiting event. 

Then activiation event.

sw.js

```javascript
self.addEventListener('install', function(event) {
  console.log('Service worker installing...');
});

self.addEventListener('activate', function(event) {
  console.log('Service worker activating...');
});
```

After the Install the activate event occurs, this is where you can clean up
outdated caches.



