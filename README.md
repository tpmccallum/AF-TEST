# AF-TEST

Testing the latency of the round-trip.

# Install Spin (and plugins)

Install Spin:

```
curl -fsSL https://developer.fermyon.com/downloads/fwf_install.sh | bash
sudo mv ./spin /usr/local/bin/spin
```

Install `aka` plugin:

```
spin plugin install aka
spin plugins update  
spin plugins upgrade aka
```

# Check Node Version

```
node --version
```

Returns:

```
v22.22.3
```

# Create New Spin Application

```
spin new -E akamai-functions -t http-js --accept-defaults hello-spin
cd hello-spin
```

# Create TypeScript Application Source Code

```
// For AutoRouter documentation refer to https://itty.dev/itty-router/routers/autorouter
import { AutoRouter } from 'itty-router';

let router = AutoRouter();

// Route ordering matters, the first route that matches will be used
// Any route that does not return will be treated as a middleware
// Any unmatched route will return a 404
router
    .get('/', () => new Response('Hello, Spin!'))
    .get('/hello/:name', ({ name }) => `Hello, ${name}!`)

addEventListener('fetch', (event) => {
    event.respondWith(router.fetch(event.request));
});
```

# Build & Run Application

```
spin up --listen 127.0.0.1:3001
```

Returns:
```
Logging component stdio to ".spin/logs/"
Preparing Wasm modules is taking a few seconds...


Serving http://127.0.0.1:3001
Available Routes:
  hello-spin: http://127.0.0.1:3001 (wildcard)
```

# Time Localhost

```
curl -i -s -w "RTT: %{time_total}s\n" 127.0.0.1:3001
```

Returns:

```
HTTP/1.1 200 OK
content-length: 12
content-type: text/plain;charset=UTF-8
date: Sat, 27 Jun 2026 23:34:12 GMT

Hello, Spin!RTT: 0.005086s
```

# Push App To Registry

```
spin aka deploy
```

Returns:

```
Name of new app: hello-spin
Creating new app hello-spin in account 
Note: If you would instead like to deploy to an existing app, cancel this deploy and link this workspace to the app with `spin aka app link`
OK to continue? yes
Workspace linked to app hello-spin
```

# Time Registry

```
curl -i -s -w "RTT: %{time_total}s\n" https://749f31f4-a554-495a-8407-e8435e3d06ff.fwf.app
```

Returns:

```
HTTP/1.1 200 OK
Content-Length: 12
Content-Type: text/plain;charset=UTF-8
x-envoy-upstream-service-time: 61
Server: envoy
Date: Sat, 27 Jun 2026 23:38:00 GMT
Connection: keep-alive
Akamai-Request-BC: [a=23.45.168.116,b=138064109,c=g,n=AU_QLD_BRISBANE,o=20940],[a=95,c=o]
Set-Cookie: akaalb_fwf-prod-apps=~op=fwf_prod:fwf-dev-au-mel|~rv=20~m=fwf-dev-au-mel:0|~os=1231e1ede8704e97468b2ddc2c84cd5b~id=ce6b4da58d7dfa04e500c58dc316c040; path=/; HttpOnly; Secure; SameSite=None
Akamai-GRN: 0.74a82d17.1782603479.83ab0ed

Hello, Spin!RTT: 0.677859s
```

