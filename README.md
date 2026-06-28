# AF-TEST

Testing the latency of the round-trip.
One outbound request to example.com:
- `0.019594s` localhost
- `0.634333s` AF

Find percentage using (((new - old) / old) * 100) formula:
`(((0.019594 - 0.634333)/0.634333)*100)`
Localhost is 96% faster at a round-trip to example.com

# TypeScript

```
interface FetchEvent extends Event {
  request: Request;
  respondWith(response: Promise<Response> | Response): void;
}

import { AutoRouter } from 'itty-router';

interface TimingEntry {
  operation: string;
  status: number;
  durationMs: number;
  timestamp: string;
}

function recordTiming(operation: string, status: number, startMs: number) {
  const durationMs = Date.now() - startMs;
  console.log(`[TIMING] ${operation} → ${status} in ${durationMs}ms`);
}

const router = AutoRouter();

router
  .get('/', () => new Response('Hello, Spin.'))
  .get('/hello/:name', ({ name }) => new Response(`Hello, ${name}!`));

router.get('/test', async () => {
  const start = Date.now();

  const resp = await fetch('https://example.com', {
    method: 'GET',
    headers: { accept: 'text/html' },
  });

  recordTiming('example.com', resp.status, start);

  return new Response('ok');
});

addEventListener('fetch', (event) => {
  (event as FetchEvent).respondWith(router.fetch((event as FetchEvent).request));
});

```

# Localhost

```
$ curl -i -s -w "RTT: %{time_total}s\n" 127.0.0.1:3001
HTTP/1.1 200 OK
content-length: 12
content-type: text/plain;charset=UTF-8
date: Sun, 28 Jun 2026 00:47:57 GMT
Hello, Spin.
RTT: 0.005706s
```

```
$ curl -i -s -w "RTT: %{time_total}s\n" 127.0.0.1:3001/hello/tim
HTTP/1.1 200 OK
content-length: 11
content-type: text/plain;charset=UTF-8
date: Sun, 28 Jun 2026 00:48:24 GMT
Hello, tim!
RTT: 0.005512s
```

```

$ curl -i -s -w "RTT: %{time_total}s\n" 127.0.0.1:3001/test
HTTP/1.1 200 OK
content-length: 2
content-type: text/plain;charset=UTF-8
date: Sun, 28 Jun 2026 00:49:00 GMT
ok
RTT: 0.019594s
```

# AF

```
$ curl -i -s -w "RTT: %{time_total}s\n" https://749f31f4-a554-495a-8407-e8435e3d06ff.fwf.app
HTTP/1.1 200 OK
Content-Length: 12
Content-Type: text/plain;charset=UTF-8
x-envoy-upstream-service-time: 11
Server: envoy
Date: Sun, 28 Jun 2026 00:50:13 GMT
Connection: keep-alive
Akamai-Request-BC: [a=23.45.168.118,b=99653042,c=g,n=AU_QLD_BRISBANE,o=20940],[a=197,c=o]
Set-Cookie: akaalb_fwf-prod-apps=~op=fwf_prod:fwf-dev-au-mel|~rv=71~m=fwf-dev-au-mel:0|~os=1231e1ede8704e97468b2ddc2c84cd5b~id=09439a21d74412da99ad957a2bf9fdf4; path=/; HttpOnly; Secure; SameSite=None
Akamai-GRN: 0.76a82d17.1782607813.5f095b2
Hello, Spin.
RTT: 0.156205s
```

```
$ curl -i -s -w "RTT: %{time_total}s\n" https://749f31f4-a554-495a-8407-e8435e3d06ff.fwf.app/hello/tim
HTTP/1.1 200 OK
Content-Length: 11
Content-Type: text/plain;charset=UTF-8
x-envoy-upstream-service-time: 2
Server: envoy
Date: Sun, 28 Jun 2026 00:51:01 GMT
Connection: keep-alive
Akamai-Request-BC: [a=23.45.168.116,b=141319396,c=g,n=AU_QLD_BRISBANE,o=20940],[a=76,c=o]
Set-Cookie: akaalb_fwf-prod-apps=~op=fwf_prod:fwf-dev-au-mel|~rv=93~m=fwf-dev-au-mel:0|~os=1231e1ede8704e97468b2ddc2c84cd5b~id=c856f5436650368c48a4053afae8d10f; path=/; HttpOnly; Secure; SameSite=None
Akamai-GRN: 0.74a82d17.1782607860.86c5ce4
Hello, tim!
RTT: 0.127381s
```

```
$ curl -i -s -w "RTT: %{time_total}s\n" https://749f31f4-a554-495a-8407-e8435e3d06ff.fwf.app/test
HTTP/1.1 200 OK
Content-Length: 2
Content-Type: text/plain;charset=UTF-8
x-envoy-upstream-service-time: 215
Server: envoy
Date: Sun, 28 Jun 2026 00:51:40 GMT
Connection: keep-alive
Akamai-Request-BC: [a=23.45.168.118,b=99727189,c=g,n=AU_QLD_BRISBANE,o=20940],[a=197,c=o]
Set-Cookie: akaalb_fwf-prod-apps=~op=fwf_prod:fwf-dev-au-mel|~rv=65~m=fwf-dev-au-mel:0|~os=1231e1ede8704e97468b2ddc2c84cd5b~id=3e5e0e2e0ff1f5f1c58b8c107c219b69; path=/; HttpOnly; Secure; SameSite=None
Akamai-GRN: 0.76a82d17.1782607900.5f1b755
ok
RTT: 0.634333s
```



