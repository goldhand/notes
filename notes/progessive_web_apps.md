Progressive Web Apps
====================
Notes and comments from [googles guide for building progressive web apps][1].



Service Workers
---------------
A service worker is a script that runs in your browser the background separate from a web page.

See my [short service worker introduction guide][4].



Application Shell Architecture
------------------------------
Display content to the user as quickly as possible on the initial load and future loads.


### Overview

#### Initial load
* Inline __critical__ css: This includes all css to render the _application shell_.
* Async load js and css resources for the current view. Asynchronously loading these resources will allow the DOM and CSSOM to render without being blocked thus reducing your initial load time.
  * Because css cannot be async loaded with just markup, use js to load css that shouldn't be a part of the __critical rendering path__.
  * To avoid accidentally loading css too fast, use `requestAnimationFrame()`.
* All content for first view should be included in the html file so multiple requests for resources don't need to be made to display content.

#### Future loads
* Let __service worker__ cache and update new resources when needed. Google has a tool for this: [sw-precache][2].
* Lazy load or background load all additional resources.


### Implementation
A practical [implementation of a progressive web app][3] was made by google and is available OSS.

__This implementation:__
* Initially serves a static page with content that registers a service worker that caches the application and it's dependencies.
* Requests made by client to urls (`/*`) will use js to load content specific to that url. _nothing special here_
* Have fallbacks for browsers without _service worker_ support.
* File versioning: Network first and use the cached version otherwise - You can use _sw-precache_ for version (cache busting)

__Expanding:__
* Use _background_ or _lazy_ loading strategies in conjunction with [sw-toolbox][4] to load and cache resources a client _is about to_ or _may request in the future_.




<!--References-->
[1]: https://developers.google.com/web/progressive-web-apps/ "Google's progressive-web-apps guide"
[2]: https://github.com/GoogleChrome/sw-precache/ "sw-precache github"
[3]: https://github.com/GoogleChrome/application-shell "Application shell demo github"
[4]: /goldhand/notes/notes/service_workers.md "Web Service Workers Introduction"
