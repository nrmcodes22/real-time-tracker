# Real-Time Location Tracker

A browser-based location sharing demo built with Node.js, Express, Socket.IO, EJS, and Leaflet. Each browser that opens the app shares its current geolocation with the server. The server broadcasts locations to all connected browsers, which display each connected client as a map marker.

> This project shares precise location with everyone connected to the same server. Use it only with people who have agreed to share their location.

## Features

- Displays an OpenStreetMap map using Leaflet.
- Requests location access from the browser.
- Watches for location changes and sends latitude and longitude over Socket.IO.
- Broadcasts received locations to all connected clients.
- Updates a client's existing marker as its location changes.
- Removes a marker when its client disconnects.

## Requirements

- Node.js and npm installed.
- A modern browser with geolocation support.
- Internet access to load the Leaflet and Socket.IO browser scripts from the CDN and map tiles from OpenStreetMap.
- Location permission enabled for the browser. Geolocation is generally available on `localhost` for development; deployed sites should use HTTPS.

## Project structure

```text
realtime_tracker/
├── app.js                 # Express server and Socket.IO location relay
├── package.json            # Project metadata and dependencies
├── package-lock.json       # Locked npm dependency versions
├── public/
│   ├── css/
│   │   └── style.css       # Full-page map styling
│   └── js/
│       └── script.js       # Browser geolocation, socket, and map logic
└── views/
    └── index.ejs           # Map page template
```

## Install and run

From the project directory, install dependencies:

```bash
npm install
```

Start the server:

```bash
node app.js
```

Open [http://localhost:3000](http://localhost:3000) in your browser and allow location access when prompted. The server listens on port `3000`.

To stop it, focus the terminal running the server and press `Ctrl+C`.

## See multiple markers

The app creates one marker per connected browser client. To see two markers, open the app from two separate devices or browsers, both connected to the same server, and allow location access in each. Opening another tab in the same browser can also create another client, depending on browser permissions and behavior.

On a second device on the same Wi-Fi network, use the host computer's local network IP address with port 3000, for example `http://192.168.1.20:3000`. The host firewall must allow inbound connections on port 3000. For access over the internet, deploy the app behind HTTPS; browsers generally block geolocation on non-secure public origins.

A marker may not appear until that client successfully reports a location. If geolocation times out, move to a location with a better GPS/network fix, check browser and operating system location permissions, and retry. The current browser code reports geolocation errors in the developer console.

## How it works

1. Express serves `views/index.ejs` at `/` and serves files under `public/` as static assets.
2. The page loads Leaflet, the Socket.IO browser client, and `/js/script.js`.
3. The browser calls `navigator.geolocation.watchPosition(...)` and emits `send-location` with latitude and longitude when a position is reported.
4. `app.js` receives that event and broadcasts `receive-location`, including the sender's Socket.IO ID, to all connected clients.
5. Each browser centers the map on the latest received position and creates or updates the marker keyed by client ID.
6. On disconnect, the server broadcasts `user-disconnected`; each browser removes that client's marker.

## Troubleshooting

### CSS or JavaScript returns HTML or shows a MIME type error

Check that `app.js` registers static middleware with:

```js
app.use(express.static(path.join(__dirname, "public")));
```

The asset URLs in the page are `/css/style.css` and `/js/script.js`, corresponding to files under `public/css/` and `public/js/`. Restart the server after changing server code, then reload the page.

### No marker appears or the browser says the user is online

A Socket.IO connection only means that a browser is connected to the server; it does not mean a location was obtained. Check the browser console for a geolocation error and verify that location permission is allowed. The current app has no separate online status display: markers are added only when a location event is received.

### Geolocation times out

The current `watchPosition` options request high accuracy with a 5-second timeout. Location can take longer, especially on desktops, indoors, or in virtual machines. Check device location services and browser permissions. If needed, adjust `timeout` or `enableHighAccuracy` in `public/js/script.js`.

### Another device cannot open the page

Use the server computer's LAN IP rather than `localhost`, ensure both devices can reach one another on the network, and allow Node.js or port 3000 through the host firewall. Public deployments need HTTPS for browser geolocation.

## Current limitations

- There is no authentication or room separation; every connected client receives every reported location.
- Locations are not saved. A newly connected browser only sees positions when clients emit another location update.
- The app does not display a connection or location status in the page.
- The server uses a fixed port (`3000`) and has no production deployment configuration.

## Privacy and deployment

This is a learning/demo app, not a secure location-sharing service. Do not expose it publicly with real users without adding authentication, access controls, consent and privacy controls, secure deployment, and an appropriate data-retention policy. Use HTTPS in deployment so browser geolocation is available and traffic is encrypted.
