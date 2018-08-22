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


### Updating the new SW with skipWaiting()

`self.skipWaiting();`

Add this typically to the install event of the sw

```javascript
(function() {
  'use strict';
    self.addEventListener('install', function(event) {
        self.skipWaiting();
    })
})
```

Note that sw should have a file change. (add/removing comment)



### Intercepting Fetch events

```javascript
self.addEventListener('fetch', function(event) {
  console.log('Fetching:', event.request.url);
});
```

---

### Caching app shell

Example of caching an app shell
```javascript
  var filesToCache = [
    '.',
    'style/main.css',
    'https://fonts.googleapis.com/css?family=Roboto:300,400,500,700',
    'images/still_life-1600_large_2x.jpg',
    'images/still_life-800_large_1x.jpg',
    'images/still_life_small.jpg',
    'images/still_life_medium.jpg',
    'index.html',
    'pages/offline.html',
    'pages/404.html'

  ];

  var staticCacheName = 'pages-cache-v1';

  self.addEventListener('install', function (event) {
    self.skipWaiting();
    console.log('hello');
    console.log('Attempting to install service worker and cache static assets');

    // event.waitUntil extends the lifetime of the install event until the
    // passed promise resolves successfully.

    event.waitUntil(
      caches.open(staticCacheName)
        .then(function (cache) {
          return cache.addAll(filesToCache);
        })
    );
  });
```




### Example of using fetch events

```javascript
  
self.addEventListener('fetch', function(event) {
  console.log('Fetch event for ', event.request.url);
  event.respondWith(
    caches.match(event.request).then(function(response) {
      if (response) {
        console.log('Found ', event.request.url, ' in cache');
        return response;
      }
      console.log('Network request for ', event.request.url);
      return fetch(event.request)
    }).catch(function(error) {
    })
  );
});

```



### Full Example


```javascript
/*
Copyright 2016 Google Inc.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
*/
(function() {
	'use strict';
	var filesToCache = [
    '.',
    'style/main.css',
    'https://fonts.googleapis.com/css?family=Roboto:300,400,500,700',
    'images/still_life-1600_large_2x.jpg',
    'images/still_life-800_large_1x.jpg',
    'images/still_life_small.jpg',
    'images/still_life_medium.jpg',
    'index.html',
    'pages/offline.html',
    'pages/404.html'
  ];
	var staticCacheName = 'pages-cache-v2';
	// On the install event use the waitUntil event to extend the life cycle to
	// cache assets listed above
	self.addEventListener('install', function(event) {
		console.log('Attempting to install service worker and cache static assets');
        // caches.open creates the staticCacheName or open the connection to it.
		event.waitUntil(caches.open(staticCacheName)
			.then(function(cache) {
				//cache.addAll will reject if any of the resources fail to cache. This
				//        means the service worker will only install if all of the resources in
				//      cache.addAll have been cached.
				return cache.addAll(filesToCache);
			}));
	});
	// This is where we hijack the fetch event to fetch from the cache
	self.addEventListener('fetch', function(event) {
		console.log('Fetch event for ', event.request.url);
		// Respond with is a fetchEvent part of the sw api FetchEvent
		event.respondWith(caches.match(event.request)
			.then(function(response) {
				if (response) {
					console.log('Found ', event.request.url, ' in cache');
					return response;
				}
				console.log('Network request for ', event.request.url);
				return fetch(event.request)
					.then(function(response) {
						if (response.status === 404) {
							return caches.match('pages/404.html');
						}
						return caches.open(staticCacheName)
							.then(function(cache) {
								if (event.request.url.indexOf('test') < 0) {

                                    // cache.put(request, response) - This
                                    // method takes both the request and response
                                    // object and adds them to the cache. This
                                    // lets you manually insert the response
                                    // object. Often, you will just want to
                                    // fetch() one or more requests and then add
                                    // the result straight to your cache. In such
                                    // cases you are better off just using
                                    // cache.add or cache.addAll, as they are
                                    // shorthand functions for one or more of
                                    // these operations:


									cache.put(event.request.url, response.clone());
								}
								return response;
							});
					});
			})
			.catch(function(error) {
				console.log('Error, ', error);
				return caches.match('pages/offline.html');
			}));
	});
	self.addEventListener('activate', function(event) {
		console.log('Activating new service worker...');
		var cacheWhitelist = [staticCacheName];
		event.waitUntil(caches.keys()
			.then(function(cacheNames) {
				return Promise.all(cacheNames.map(function(cacheName) {
					if (cacheWhitelist.indexOf(cacheName) === -1) {
						return caches.delete(cacheName);
					}
				}));
			}));
	});
})();
```


#### On Click save article

```javascript
document.querySelector('.cache-article').addEventListener('click', function(event) {
  event.preventDefault();
  var id = this.dataset.articleId;
  caches.open('mysite-article-' + id).then(function(cache) {
    fetch('/get-article-urls?id=' + id).then(function(response) {
      // /get-article-urls returns a JSON-encoded array of
      // resource URLs that a given article depends on
      return response.json();
    }).then(function(urls) {
      cache.addAll(urls);
    });
  });
});
```

#### If a request doesn't match anything in the cache, get it from the network, send it to the page and add it to the cache at the same time.


```javascript
self.addEventListener('fetch', function(event) {
  event.respondWith(
    caches.open('mysite-dynamic').then(function(cache) {
      return cache.match(event.request).then(function (response) {
        return response || fetch(event.request).then(function(response) {
          cache.put(event.request, response.clone());
          return response;
        });
      });
    })
  );
});
```

note to delete the old items in the cache or your app will take a large amount of space.



`caches.match(request, options)` - This method returns a Promise that resolves to
the response object associated with the first matching request in the cache or
caches.It returns undefined if no match is found.The first parameter is the
request, and the second is an optional list of options to refine the search
.Here are the options as defined by MDN: 

`ignoreSearch`: A Boolean that specifies
whether to ignore the query string in the URL.For example, if set to true the ?
value = bar part of http : //foo.com/?value=bar would be ignored when
performing a match. It defaults to false. 

`ignoreMethod`: A Boolean that, when
set to true, prevents matching operations from validating the Request HTTP
method( normally only GET and HEAD are allowed.) It defaults to false.

`ignoreVary`: A Boolean that when set to true tells the matching operation not to
perform VARY header matching— that is, if the URL matches you will get a match
regardless of whether the Response object has a VARY header.It defaults to
false . 

`cacheName`: A DOMString that represents a specific cache to search
within.Note that this option is ignored by Cache.match() .
caches.matchAll(request, options) - This method is the same as.match except
that it returns all of the matching responses from the cache instead of just
the first.For example, if your app has cached some images contained in an image
folder, we could return all images and perform some operation on them like
this:


`cache.delete(request, options)` This method finds the item in the cache
matching the request, deletes it, and returns a Promise that resolves to true.
If it doesn't find the item, it resolves to false. It also has the same
optional options parameter available to it as the match method.


We can get a list of cache keys using `cache.keys(request, options)`. This
returns a Promise that resolves to an array of cache keys. These will be
returned in the same order they were inserted into the cache. Both parameters
are optional. If nothing is passed, cache.keys returns all of the requests in
the cache. If a request is passed, it returns all of the matching requests from
the cache. The options are the same as those in the previous methods.

