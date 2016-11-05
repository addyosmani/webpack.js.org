---
title: How to use the PRPL pattern
contributors:
  - addyo
---

TODOS:

Code-splitting
async getComponent with React
Commons vendor chunking
Optional: preload / H2 Push with aggressive splitting plugin

Prose:

PRPL is a pattern for structuring and serving Progressive Web Apps (PWAs), with 
an emphasis on the performance of app delivery and launch. It stands for:

* **Push** critical resources for the initial URL route.
* **Render** initial route.
* **Pre-cache** remaining routes.
* **Lazy-load** and create remaining routes on demand.

Loading the code for routes more granularly and allowing browsers to schedule work better 
has the potential to greatly aid reaching interactivity in our applications sooner. We need better 
architectures that enable interactivity quickly and the PRPL pattern is an interesting example of 
how to accomplish this goal on real mobile devices.

## Structure

PRPL can work well if you have a single-page app (SPA) with the following structure:

* The main entrypoint of the application which is served from every valid route. This file should be very small, 
since it will be served from different URLs and therefore be cached multiple times. All resource URLs in the entrypoint 
need to be absolute, since it may be served from non-top-level URLs.
* The shell or app-shell, which includes the top-level app logic, router, and so on.
* Lazily loaded fragments of the app. A fragment can represent the code for a particular view, or other code that can be 
loaded lazily (for example, parts of the main app not required for first paint, like menus that aren't displayed until a 
user interacts with the app). The shell is responsible for dynamically importing the fragments as needed.

The server and service worker together work to precache the resources for the inactive routes.

When the user switches routes, the app lazy-loads any required resources that haven't been cached yet, and creates 
the required views. Repeat visits to routes should be immediately interactive. Service Worker helps a lot here.

## App entrypoint

The entrypoint must import and instantiate the shell, as well as conditionally load any required polyfills.

The main considerations for the entrypoint are:

* Has minimal static dependencies, in other words, not much beyond the app-shell itself.
* Conditionally loads required polyfills.
* Uses absolute paths for all dependencies.


## App shell

The shell is responsible for routing and usually includes the main navigation UI for the app.

The app should lazy-load fragments as they're required. For example, when the user changes to a new route, it imports 
the fragment(s) associated with that route. This may initiate a new request to the server, or simply load the resource 
from the cache.

The shell (including its static dependencies) should contain everything needed for first paint.