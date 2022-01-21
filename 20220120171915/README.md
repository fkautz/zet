# Progressive Web Apps

Progressive Web Applications are installable web applications written in HTML/CSS/js that are designed to progressively increase in capabilities depending on where they are running and what they have access to.

One common structure for WPA is the js13kPWA architecture.
A service worker provides functionality of the application.
The data is typically held within a javascript object.

Service workers can only be executed in a secure context (e.g. they require HTTPS)
This pattern allows for offline service workers to work with cached data.
The service worker cannot communicate with the DOM directly. Instead, the service worker handles events posted by the `postMessage` interface.
Responses from the service workers are handled via `Promises`.

# Resources

https://developers.google.com/web/fundamentals/primers/service-workers
