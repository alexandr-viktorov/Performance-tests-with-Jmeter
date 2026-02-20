# k6 Load Testing Training Course
## Complete Guide for Performance Testing Engineers

---

## Course Overview

This comprehensive training course covers Grafana k6 from fundamentals to advanced performance testing techniques. Each lesson includes theory, hands-on practice, and assessment questions.

**Prerequisites:** Basic understanding of HTTP, JavaScript, web applications, and testing concepts
**Duration:** 8-10 weeks (self-paced)
**Tools Required:** k6 v0.50+, Node.js (optional, for tooling), a text editor or IDE
**Practice APIs Used:**
- https://httpbin.org — HTTP inspection and utility endpoints
- https://jsonplaceholder.typicode.com — Fake REST API for prototyping
- https://dummyjson.com — Feature-rich dummy JSON API

---

## Lesson 1: Introduction to k6 and Performance Testing Fundamentals

### Theoretical Part

#### What is Performance Testing?

Performance testing evaluates how a system behaves under specific workload conditions. Key categories include:

- **Load Testing**: Validates behavior under expected user load
- **Stress Testing**: Pushes the system beyond normal capacity to find breaking points
- **Spike Testing**: Simulates sudden, dramatic increases in traffic
- **Soak/Endurance Testing**: Long-duration tests to detect memory leaks and degradation
- **Scalability Testing**: Measures how the system scales with added resources
- **Smoke Testing**: Minimal load test to verify the script and environment work

#### Key Performance Metrics

- **Response Time (Latency)**: Time from request sent to response received
- **Throughput (RPS)**: Requests processed per second
- **Error Rate**: Percentage of failed requests
- **Concurrent Users (VUs)**: Simultaneous virtual users active at a point in time
- **Percentiles (p90, p95, p99)**: Distribution-based response time analysis
- **Saturation Point**: Load at which performance starts to degrade

#### What is k6?

k6 is a modern, developer-centric open-source load testing tool built by Grafana Labs. Key characteristics:

- **Language**: Tests are written in JavaScript (ES2015+)
- **Engine**: Written in Go for high performance and low resource usage
- **Execution**: Runs scripts locally, in the cloud (Grafana Cloud), or in CI/CD
- **Output**: Built-in metrics, console output, and integrations with InfluxDB, Prometheus, Datadog, and more
- **No Browser / UI**: k6 operates at the protocol level (HTTP, WebSocket, gRPC) — no GUI overhead

#### k6 vs JMeter — Key Differences

| Feature | k6 | JMeter |
|---|---|---|
| Language | JavaScript | XML / GUI |
| Resource usage | Very low (Go) | Higher (JVM) |
| Scripting | Code-first | GUI-first |
| Protocol support | HTTP, WS, gRPC | HTTP, FTP, JDBC, SMTP, etc. |
| CI/CD integration | Native, easy | Possible but complex |
| Browser testing | k6 Browser (experimental) | None (requires plugins) |

#### k6 Architecture

- **VU (Virtual User)**: A simulated user running the script; each VU runs the `default` function in a loop
- **Executor**: Controls how VUs are scheduled (constant VUs, ramping VUs, arrival rate, etc.)
- **Scenario**: A named workload configuration (executor + options)
- **Metrics**: Built-in and custom measurements collected during the test
- **Thresholds**: Pass/fail criteria applied to metrics
- **Checks**: Inline assertions (like assertions in code)

#### k6 Script Lifecycle

```
┌─────────────────────────────────────────────┐
│  init code  (runs once per VU, on start)    │
│    ↓                                         │
│  setup()   (runs once before test)          │
│    ↓                                         │
│  default() (runs repeatedly per VU)         │
│    ↓                                         │
│  teardown() (runs once after test)          │
└─────────────────────────────────────────────┘
```

### Practical Part

#### Exercise 1.1: Install k6

**Windows (Chocolatey):**
```bash
choco install k6
```

**Windows (Winget):**
```bash
winget install k6 --source winget
```

**macOS (Homebrew):**
```bash
brew install k6
```

**Linux (Debian/Ubuntu):**
```bash
sudo gpg -k
sudo gpg --no-default-keyring --keyring /usr/share/keyrings/k6-archive-keyring.gpg \
  --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C5AD17C747E3415A3642D57D77C6C491D6AC1D69
echo "deb [signed-by=/usr/share/keyrings/k6-archive-keyring.gpg] \
  https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list
sudo apt-get update && sudo apt-get install k6
```

**Verify installation:**
```bash
k6 version
```

#### Exercise 1.2: Your First k6 Script

Create a file named `lesson1_first_test.js`:

```javascript
import http from 'k6/http';
import { sleep } from 'k6';

// Options block: defines the load profile
export const options = {
  vus: 5,          // 5 virtual users
  duration: '10s', // run for 10 seconds
};

// Default function: executed repeatedly by each VU
export default function () {
  const res = http.get('https://httpbin.org/get');

  console.log(`Status: ${res.status} | Duration: ${res.timings.duration.toFixed(0)}ms`);

  sleep(1); // think time: 1 second between iterations
}
```

**Run the test:**
```bash
k6 run lesson1_first_test.js
```

**Observe the output:** k6 prints a live progress bar and a summary table with metrics after the test completes.

#### Exercise 1.3: Explore the k6 Summary Output

After running Exercise 1.2, read the summary table:

```
data_received........: X kB   X kB/s
data_sent............: X kB   X kB/s
http_req_blocked.....: avg=Xms  min=Xms  med=Xms  max=Xms  p(90)=Xms p(95)=Xms
http_req_connecting..: avg=Xms  ...
http_req_duration....: avg=Xms  ...   ← Total request time (most important)
http_req_failed......: X%     X/X
http_reqs............: X      X/s     ← Throughput
iteration_duration...: avg=Xs  ...
iterations...........: X      X/s
vus...................: X      min=X   max=X
vus_max..............: X      min=X   max=X
```

**Tasks:**
1. Identify the average response time (`http_req_duration` avg)
2. Identify the throughput (`http_reqs` per second)
3. Identify the error rate (`http_req_failed`)
4. Note the difference between `http_req_duration` and `iteration_duration`

#### Exercise 1.4: Understanding the init, setup, teardown Lifecycle

Create `lesson1_lifecycle.js`:

```javascript
import http from 'k6/http';
import { sleep } from 'k6';

// ─── INIT (runs once per VU at startup) ───────────────────────────────────────
console.log('INIT: VU initialization');

const BASE_URL = 'https://jsonplaceholder.typicode.com';

export const options = {
  vus: 2,
  duration: '5s',
};

// ─── SETUP (runs once before all VUs start) ───────────────────────────────────
export function setup() {
  console.log('SETUP: Preparing test data...');
  const res = http.get(`${BASE_URL}/users/1`);
  const user = JSON.parse(res.body);
  console.log(`SETUP: Loaded user "${user.name}"`);
  return { userId: user.id, userName: user.name }; // returned data is passed to default()
}

// ─── DEFAULT (runs in a loop per VU for the test duration) ────────────────────
export default function (data) {
  // data contains what setup() returned
  console.log(`VU running for user: ${data.userName}`);

  const res = http.get(`${BASE_URL}/posts?userId=${data.userId}`);
  console.log(`Posts status: ${res.status}`);

  sleep(1);
}

// ─── TEARDOWN (runs once after all VUs finish) ────────────────────────────────
export function teardown(data) {
  console.log(`TEARDOWN: Test complete for user "${data.userName}"`);
}
```

**Run and observe** which phase runs when, and how `data` flows from `setup()` to `default()` and `teardown()`.

### Control Questions

1. **What is the difference between a VU (Virtual User) and an iteration in k6?**
   <details>
   <summary>Answer</summary>
   A VU is a simulated user that runs the script concurrently. An iteration is one complete execution of the `default()` function. One VU can perform many iterations over the test duration.
   </details>

2. **Why is k6 considered lightweight compared to JMeter?**
   <details>
   <summary>Answer</summary>
   k6 is written in Go, which is highly efficient and has low memory overhead. Each VU is a goroutine, not a full OS thread (unlike JMeter's Java threads). This lets k6 run thousands of VUs on modest hardware.
   </details>

3. **What runs in the `init` section of a k6 script and why should HTTP calls NOT be placed there?**
   <details>
   <summary>Answer</summary>
   The init section runs once per VU before the test starts, used for imports and configuration. HTTP calls must NOT be placed there because it runs outside the VU context and k6 will throw an error — network calls are only allowed inside `default()`, `setup()`, or `teardown()`.
   </details>

4. **What is the purpose of `sleep()` in a k6 script?**
   <details>
   <summary>Answer</summary>
   `sleep()` adds think time between requests to simulate realistic user pacing. Without sleep, VUs execute requests as fast as possible, creating unrealistic load patterns that can skew results and not represent real user behavior.
   </details>

5. **In the k6 summary output, what does `p(95)` in `http_req_duration` represent?**
   <details>
   <summary>Answer</summary>
   p(95) means the 95th percentile response time — 95% of all requests completed faster than this value. It's more useful than the average because it's not skewed by a few very fast or very slow responses.
   </details>

---

## Lesson 2: HTTP Requests and Protocol Testing

### Theoretical Part

#### k6 HTTP Module Overview

The `k6/http` module provides functions to make HTTP requests. All requests are synchronous within a VU's execution context.

**Core Functions:**
```javascript
http.get(url, [params])
http.post(url, body, [params])
http.put(url, body, [params])
http.patch(url, body, [params])
http.del(url, body, [params])        // DELETE
http.options(url, body, [params])
http.head(url, [params])
http.request(method, url, body, [params])  // generic
http.batch([requests])               // parallel requests
```

#### The Response Object

Every request returns a `Response` object:

```javascript
const res = http.get('https://httpbin.org/get');

res.status          // HTTP status code (number)
res.body            // Response body (string)
res.json()          // Parsed JSON body (shorthand for JSON.parse(res.body))
res.headers         // Response headers (object)
res.timings         // Detailed timing breakdown
res.url             // Final URL (after redirects)
res.error           // Error string if request failed
res.error_code      // Numeric error code
```

#### Timing Breakdown (res.timings)

```
res.timings.blocked      // Time waiting for TCP connection slot
res.timings.connecting   // TCP handshake time
res.timings.tls_handshaking // TLS/SSL negotiation time
res.timings.sending      // Time to send the request
res.timings.waiting      // Time to first byte (TTFB)
res.timings.receiving    // Time to receive the response body
res.timings.duration     // Total request duration (most used)
```

#### Request Parameters (params)

```javascript
const params = {
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer my-token',
  },
  tags: { name: 'get-post' },   // for grouping in metrics
  timeout: '10s',                // request timeout
  redirects: 5,                  // max redirects (default 10)
};
```

#### Sending Request Bodies

```javascript
// JSON body — must stringify and set Content-Type
const payload = JSON.stringify({ title: 'Test', body: 'Content', userId: 1 });
const params  = { headers: { 'Content-Type': 'application/json' } };
http.post(url, payload, params);

// Form-encoded body — pass an object directly
http.post(url, { username: 'alice', password: 'secret' });

// Raw string body
http.post(url, 'raw body string', params);
```

#### Batch Requests

Send multiple requests in parallel within a single iteration:

```javascript
const responses = http.batch([
  ['GET', 'https://jsonplaceholder.typicode.com/posts/1'],
  ['GET', 'https://jsonplaceholder.typicode.com/users/1'],
  ['GET', 'https://jsonplaceholder.typicode.com/albums/1'],
]);
// responses is an array matching the request order
```

### Practical Part

#### Exercise 2.1: Testing All HTTP Methods

Create `lesson2_http_methods.js`:

```javascript
import http from 'k6/http';
import { sleep, check } from 'k6';

export const options = { vus: 1, iterations: 1 };

const JSON_API  = 'https://jsonplaceholder.typicode.com';
const HTTPBIN   = 'https://httpbin.org';
const JSON_HDR  = { headers: { 'Content-Type': 'application/json' } };

export default function () {
  // GET
  let res = http.get(`${JSON_API}/posts/1`);
  console.log(`GET  /posts/1     → ${res.status}`);

  // POST
  const newPost = JSON.stringify({ title: 'k6 Test', body: 'Created by k6', userId: 1 });
  res = http.post(`${JSON_API}/posts`, newPost, JSON_HDR);
  console.log(`POST /posts       → ${res.status} | id=${res.json('id')}`);

  // PUT
  const update = JSON.stringify({ id: 1, title: 'Updated', body: 'Updated body', userId: 1 });
  res = http.put(`${JSON_API}/posts/1`, update, JSON_HDR);
  console.log(`PUT  /posts/1     → ${res.status}`);

  // PATCH
  const patch = JSON.stringify({ title: 'Patched Title' });
  res = http.patch(`${JSON_API}/posts/1`, patch, JSON_HDR);
  console.log(`PATCH /posts/1    → ${res.status}`);

  // DELETE
  res = http.del(`${JSON_API}/posts/1`);
  console.log(`DELETE /posts/1   → ${res.status}`);

  // Verify request headers sent
  res = http.get(`${HTTPBIN}/headers`, {
    headers: { 'X-Custom-Header': 'k6-test', 'Accept': 'application/json' },
  });
  console.log(`GET  /headers     → ${res.status}`);
  console.log('Received headers:', JSON.stringify(res.json('headers'), null, 2));

  sleep(1);
}
```

#### Exercise 2.2: Working with Query Parameters

Create `lesson2_query_params.js`:

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = { vus: 3, duration: '15s' };

const BASE = 'https://dummyjson.com';

export default function () {
  // Filter products by category
  let res = http.get(`${BASE}/products/category/smartphones`);
  check(res, { 'category returns 200': (r) => r.status === 200 });

  // Search with query string — build URL manually
  res = http.get(`${BASE}/products/search?q=phone&limit=5&skip=0`);
  check(res, {
    'search 200': (r) => r.status === 200,
    'search has products': (r) => r.json('products').length > 0,
  });
  console.log(`Search found: ${res.json('total')} products`);

  // Pagination
  for (let page = 0; page < 3; page++) {
    const limit = 10;
    const skip  = page * limit;
    res = http.get(`${BASE}/products?limit=${limit}&skip=${skip}`);
    check(res, { [`page ${page} 200`]: (r) => r.status === 200 });
    console.log(`Page ${page}: ${res.json('products').length} items`);
    sleep(0.5);
  }

  sleep(1);
}
```

#### Exercise 2.3: Authentication Headers

Create `lesson2_auth.js`:

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = { vus: 2, duration: '10s' };

export default function () {
  // Bearer token auth
  const bearerRes = http.get('https://httpbin.org/bearer', {
    headers: { Authorization: 'Bearer my-test-token-12345' },
  });
  check(bearerRes, {
    'bearer auth 200': (r) => r.status === 200,
    'authenticated flag': (r) => r.json('authenticated') === true,
  });

  // Basic auth — httpbin.org/basic-auth/{user}/{pass} expects Basic header
  const credentials = 'testuser:testpass';
  const encoded     = btoa(credentials); // base64 encode
  const basicRes = http.get('https://httpbin.org/basic-auth/testuser/testpass', {
    headers: { Authorization: `Basic ${encoded}` },
  });
  check(basicRes, {
    'basic auth 200': (r) => r.status === 200,
    'user matches': (r) => r.json('user') === 'testuser',
  });

  // Simulated JWT login with dummyjson
  const loginRes = http.post(
    'https://dummyjson.com/auth/login',
    JSON.stringify({ username: 'emilys', password: 'emilyspass' }),
    { headers: { 'Content-Type': 'application/json' } }
  );
  check(loginRes, { 'login 200': (r) => r.status === 200 });

  if (loginRes.status === 200) {
    const token = loginRes.json('accessToken');
    // Use token for authenticated endpoint
    const meRes = http.get('https://dummyjson.com/auth/me', {
      headers: { Authorization: `Bearer ${token}` },
    });
    check(meRes, { 'auth/me 200': (r) => r.status === 200 });
    console.log(`Logged in as: ${meRes.json('username')}`);
  }

  sleep(1);
}
```

#### Exercise 2.4: Batch Parallel Requests

Create `lesson2_batch.js`:

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = { vus: 5, duration: '20s' };

const BASE = 'https://jsonplaceholder.typicode.com';

export default function () {
  // Send 4 requests simultaneously
  const responses = http.batch([
    ['GET', `${BASE}/posts/1`],
    ['GET', `${BASE}/users/1`],
    ['GET', `${BASE}/albums/1`],
    ['GET', `${BASE}/todos/1`],
  ]);

  check(responses[0], { 'post 200':  (r) => r.status === 200 });
  check(responses[1], { 'user 200':  (r) => r.status === 200 });
  check(responses[2], { 'album 200': (r) => r.status === 200 });
  check(responses[3], { 'todo 200':  (r) => r.status === 200 });

  console.log(`Post: "${responses[0].json('title').substring(0, 30)}..."`);
  console.log(`User: ${responses[1].json('name')}`);

  sleep(1);
}
```

#### Exercise 2.5: Inspecting Response Timings

Create `lesson2_timings.js`:

```javascript
import http from 'k6/http';
import { sleep } from 'k6';

export const options = { vus: 1, iterations: 3 };

export default function () {
  const res = http.get('https://httpbin.org/delay/1');
  const t   = res.timings;

  console.log('─── Timing Breakdown ───────────────────');
  console.log(`  Blocked (queuing):     ${t.blocked.toFixed(0)}ms`);
  console.log(`  Connecting (TCP):      ${t.connecting.toFixed(0)}ms`);
  console.log(`  TLS Handshaking:       ${t.tls_handshaking.toFixed(0)}ms`);
  console.log(`  Sending (request):     ${t.sending.toFixed(0)}ms`);
  console.log(`  Waiting (TTFB):        ${t.waiting.toFixed(0)}ms`);
  console.log(`  Receiving (body):      ${t.receiving.toFixed(0)}ms`);
  console.log(`  TOTAL Duration:        ${t.duration.toFixed(0)}ms`);
  console.log('────────────────────────────────────────');

  sleep(0.5);
}
```

### Control Questions

1. **What is the difference between `http.get()` and `http.request('GET', ...)`?**
   <details>
   <summary>Answer</summary>
   They are functionally equivalent. `http.get()`, `http.post()`, etc. are convenience wrappers around `http.request()`. Use the method-specific helpers for clarity, and `http.request()` when the method is dynamic (e.g., determined at runtime).
   </details>

2. **How do you send a JSON body in a POST request? What two things are required?**
   <details>
   <summary>Answer</summary>
   1) Stringify the payload with `JSON.stringify(obj)` and 2) set the `Content-Type: application/json` header in params. Without the header, the server may not parse the body correctly.
   </details>

3. **What does `res.timings.waiting` measure and why is it important?**
   <details>
   <summary>Answer</summary>
   `waiting` is the Time To First Byte (TTFB) — the time between sending the request and receiving the first byte of the response. It reflects server-side processing time and is a key indicator of backend performance, separate from network latency.
   </details>

4. **When would you use `http.batch()` instead of sequential requests?**
   <details>
   <summary>Answer</summary>
   When simulating a browser loading multiple resources concurrently (images, scripts, APIs called in parallel). `http.batch()` sends all requests simultaneously and returns when all complete, accurately modeling parallel loading behavior.
   </details>

5. **What is the difference between `res.body` and `res.json()`?**
   <details>
   <summary>Answer</summary>
   `res.body` is the raw response body as a string. `res.json()` parses it as JSON and returns a JavaScript object. `res.json('field')` also accepts a JSONPath-like selector to extract a specific field directly.
   </details>

---

## Lesson 3: Checks and Thresholds — Assertions and Pass/Fail Criteria

### Theoretical Part

#### Checks — Inline Assertions

Checks are boolean conditions evaluated against response data. Unlike test framework assertions, **a failed check does NOT stop the test** — it records a failure in the `checks` metric.

```javascript
import { check } from 'k6';

const res = http.get('https://example.com/api/users');

check(res, {
  'status is 200':       (r) => r.status === 200,
  'body contains users': (r) => r.body.includes('users'),
  'response time < 2s':  (r) => r.timings.duration < 2000,
  'user count > 0':      (r) => r.json('total') > 0,
});
```

**Check results appear in the summary:**
```
checks.........................: 97.50% ✓ 390  ✗ 10
```

#### Thresholds — Pass/Fail Criteria

Thresholds are **test-level SLA definitions** that determine whether the test passes or fails. They are defined in `options` and evaluated against built-in or custom metrics.

```javascript
export const options = {
  thresholds: {
    // 95% of requests must finish below 500ms
    'http_req_duration': ['p(95)<500'],

    // Error rate must be below 1%
    'http_req_failed': ['rate<0.01'],

    // All checks must pass
    'checks': ['rate>0.99'],

    // Custom metric threshold
    'my_custom_metric': ['avg<100'],
  },
};
```

**Threshold expressions:**
```
avg   — average
min   — minimum
max   — maximum
med   — median (p50)
p(N)  — Nth percentile (e.g., p(95), p(99))
count — total count
rate  — rate (for counters/rates, 0.0–1.0)
```

**Threshold abortion (fail fast):**
```javascript
thresholds: {
  'http_req_duration': [{ threshold: 'p(99)<1000', abortOnFail: true, delayAbortEval: '10s' }],
}
```

#### Built-in k6 Metrics

| Metric | Type | Description |
|---|---|---|
| `http_reqs` | Counter | Total HTTP requests |
| `http_req_duration` | Trend | Total request duration |
| `http_req_failed` | Rate | Failed requests (non-2xx/3xx) |
| `http_req_blocked` | Trend | Time blocked waiting for TCP slot |
| `http_req_connecting` | Trend | TCP connection time |
| `http_req_tls_handshaking` | Trend | TLS handshake time |
| `http_req_sending` | Trend | Request send time |
| `http_req_waiting` | Trend | Time to first byte |
| `http_req_receiving` | Trend | Response body receive time |
| `vus` | Gauge | Current active VUs |
| `iterations` | Counter | Total completed iterations |
| `iteration_duration` | Trend | Time per iteration |
| `checks` | Rate | Check pass rate |
| `data_sent` | Counter | Data sent (bytes) |
| `data_received` | Counter | Data received (bytes) |

### Practical Part

#### Exercise 3.1: Comprehensive Checks

Create `lesson3_checks.js`:

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = { vus: 10, duration: '20s' };

const BASE = 'https://dummyjson.com';

export default function () {
  // ── Check GET response ──────────────────────────────────────────────────
  const productRes = http.get(`${BASE}/products/1`);
  check(productRes, {
    'product status 200':       (r) => r.status === 200,
    'product has id':           (r) => r.json('id') !== undefined,
    'product has title':        (r) => r.json('title').length > 0,
    'product price positive':   (r) => r.json('price') > 0,
    'response time < 3s':       (r) => r.timings.duration < 3000,
    'content-type is JSON':     (r) => r.headers['Content-Type'].includes('application/json'),
  });

  // ── Check POST response ─────────────────────────────────────────────────
  const newProduct = JSON.stringify({
    title: 'k6 Test Product',
    price: 99.99,
    stock: 50,
    category: 'smartphones',
  });
  const postRes = http.post(`${BASE}/products/add`, newProduct, {
    headers: { 'Content-Type': 'application/json' },
  });
  check(postRes, {
    'created status 201':  (r) => r.status === 201,
    'new id assigned':     (r) => r.json('id') > 0,
    'title matches':       (r) => r.json('title') === 'k6 Test Product',
  });

  // ── Check array response ────────────────────────────────────────────────
  const listRes = http.get(`${BASE}/products?limit=5`);
  check(listRes, {
    'list 200':             (r) => r.status === 200,
    'has 5 products':       (r) => r.json('products').length === 5,
    'total > 0':            (r) => r.json('total') > 0,
    'each product has id':  (r) => r.json('products').every((p) => p.id > 0),
  });

  sleep(1);
}
```

#### Exercise 3.2: Defining Thresholds for SLA Validation

Create `lesson3_thresholds.js`:

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  vus: 20,
  duration: '30s',

  thresholds: {
    // SLA 1: 95% of requests finish in under 2 seconds
    'http_req_duration': ['p(95)<2000', 'p(99)<3000'],

    // SLA 2: Error rate must stay below 2%
    'http_req_failed': ['rate<0.02'],

    // SLA 3: At least 98% of all checks must pass
    'checks': ['rate>0.98'],

    // SLA 4: Per-endpoint thresholds using tags
    'http_req_duration{endpoint:products}': ['p(90)<1500'],
    'http_req_duration{endpoint:search}':   ['p(90)<2000'],
  },
};

export default function () {
  // Tag requests to separate their metrics in thresholds
  const productRes = http.get('https://dummyjson.com/products/1', {
    tags: { endpoint: 'products' },
  });
  check(productRes, { 'products 200': (r) => r.status === 200 });

  const searchRes = http.get('https://dummyjson.com/products/search?q=laptop', {
    tags: { endpoint: 'search' },
  });
  check(searchRes, { 'search 200': (r) => r.status === 200 });

  sleep(1);
}
```

#### Exercise 3.3: Abort on Threshold Violation

Create `lesson3_abort.js`:

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  vus: 10,
  duration: '60s',

  thresholds: {
    // If error rate exceeds 10% for 15 seconds, abort the test immediately
    'http_req_failed': [
      {
        threshold:       'rate<0.10',
        abortOnFail:     true,
        delayAbortEval:  '15s', // wait 15s before checking (warm-up grace period)
      },
    ],
    // p99 must stay below 5 seconds, abort immediately on breach
    'http_req_duration': [
      {
        threshold:    'p(99)<5000',
        abortOnFail:  true,
        delayAbortEval: '10s',
      },
    ],
  },
};

export default function () {
  const res = http.get('https://jsonplaceholder.typicode.com/posts');
  check(res, { 'posts 200': (r) => r.status === 200 });
  sleep(0.5);
}
```

#### Exercise 3.4: Checking Specific Fields and Nested Data

Create `lesson3_deep_checks.js`:

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = { vus: 3, duration: '15s' };

export default function () {
  // Deep JSON navigation with res.json()
  const userRes = http.get('https://jsonplaceholder.typicode.com/users/1');
  const user    = userRes.json();

  check(userRes, {
    'status 200':               (r) => r.status === 200,
    'user has name':            ()  => user.name.length > 0,
    'user has email':           ()  => user.email.includes('@'),
    'user has address':         ()  => user.address !== undefined,
    'city is Gwenborough':      ()  => user.address.city === 'Gwenborough',
    'geo lat is string/number': ()  => user.address.geo.lat !== undefined,
    'company name not empty':   ()  => user.company.name.length > 0,
  });

  // Check array data from dummyjson
  const cartRes  = http.get('https://dummyjson.com/carts/1');
  const cart     = cartRes.json();

  check(cartRes, {
    'cart 200':                  (r) => r.status === 200,
    'cart has products':         ()  => cart.products.length > 0,
    'each product has quantity': ()  => cart.products.every((p) => p.quantity > 0),
    'total > 0':                 ()  => cart.total > 0,
    'discounted total <= total': ()  => cart.discountedTotal <= cart.total,
  });

  sleep(1);
}
```

### Control Questions

1. **What happens to test execution when a `check()` fails in k6?**
   <details>
   <summary>Answer</summary>
   Nothing — the test continues executing. A failed check is recorded in the `checks` metric (incrementing the fail counter) but does not throw an error or abort the VU. You must configure a `threshold` on the `checks` metric to make check failures cause a test failure.
   </details>

2. **What is the difference between a `check` and a `threshold` in k6?**
   <details>
   <summary>Answer</summary>
   A `check` is a per-request inline assertion (did this specific response meet criteria?). A `threshold` is a test-level pass/fail criterion based on aggregated metric values across the entire test (e.g., p95 < 500ms for all requests). Checks provide visibility; thresholds determine test outcome.
   </details>

3. **Write a threshold that requires the 90th percentile response time to be under 1 second.**
   <details>
   <summary>Answer</summary>
   `'http_req_duration': ['p(90)<1000']`
   </details>

4. **How can you apply different thresholds to different API endpoints in one test?**
   <details>
   <summary>Answer</summary>
   Use tags on requests (`tags: { endpoint: 'login' }`) and reference them in thresholds using the filter syntax: `'http_req_duration{endpoint:login}': ['p(95)<500']`.
   </details>

5. **What does `abortOnFail: true` with `delayAbortEval: '10s'` do in a threshold?**
   <details>
   <summary>Answer</summary>
   If the threshold is breached (e.g., error rate > 10%), k6 will stop the entire test after the 10-second grace period. The delay allows the system to warm up before the threshold is evaluated, preventing false positives at test start.
   </details>

---

## Lesson 4: Load Profiles, Scenarios, and Executors

### Theoretical Part

#### Understanding Load Profiles

A load profile defines how virtual users (or arrival rate) change over time. k6 provides multiple **executors** to model different patterns.

#### k6 Executors

**1. `constant-vus` — Steady Load**
```javascript
executor: 'constant-vus',
vus: 50,
duration: '5m',
```
Runs a fixed number of VUs for a set duration. Best for: baseline tests.

**2. `ramping-vus` — Gradual Ramp**
```javascript
executor: 'ramping-vus',
startVUs: 0,
stages: [
  { duration: '2m', target: 100 },  // ramp up to 100 VUs over 2 min
  { duration: '5m', target: 100 },  // stay at 100 VUs for 5 min
  { duration: '1m', target: 0 },    // ramp down to 0
],
```
Best for: load and stress tests with gradual ramp-up.

**3. `constant-arrival-rate` — Fixed RPS**
```javascript
executor: 'constant-arrival-rate',
rate: 100,          // 100 iterations per timeUnit
timeUnit: '1s',     // = 100 req/sec
duration: '1m',
preAllocatedVUs: 50,
maxVUs: 200,        // k6 will add VUs as needed to sustain rate
```
Best for: SLA validation at a specific throughput target.

**4. `ramping-arrival-rate` — Ramp RPS**
```javascript
executor: 'ramping-arrival-rate',
startRate: 10,
timeUnit: '1s',
preAllocatedVUs: 50,
maxVUs: 500,
stages: [
  { duration: '2m', target: 50  },  // ramp from 10 to 50 req/s
  { duration: '5m', target: 50  },  // hold at 50 req/s
  { duration: '2m', target: 200 },  // spike to 200 req/s
  { duration: '1m', target: 0   },  // ramp down
],
```
Best for: spike testing, stress testing by request rate.

**5. `per-vu-iterations` — Exact Iteration Count per VU**
```javascript
executor: 'per-vu-iterations',
vus: 10,
iterations: 100,  // each VU runs exactly 100 iterations
maxDuration: '10m',
```
Best for: smoke tests, one-shot data-driven tests.

**6. `shared-iterations` — Total Iteration Pool**
```javascript
executor: 'shared-iterations',
vus: 10,
iterations: 500,  // 500 iterations shared across all VUs
maxDuration: '5m',
```
Best for: processing a fixed dataset.

**7. `externally-controlled` — Manual/API Control**
Allows controlling VUs via k6's REST API at runtime.

#### Scenarios — Multiple Concurrent Workloads

Scenarios let you run multiple named workloads simultaneously in one test run:

```javascript
export const options = {
  scenarios: {
    browse_products: {
      executor: 'ramping-vus',
      stages: [
        { duration: '1m', target: 50 },
        { duration: '3m', target: 50 },
        { duration: '30s', target: 0 },
      ],
      exec: 'browseProducts',  // function to call
    },
    checkout: {
      executor: 'constant-vus',
      vus: 10,
      duration: '4m',
      exec: 'checkout',
      startTime: '1m',  // start 1 min after test begins
    },
  },
};

export function browseProducts() { /* ... */ }
export function checkout() { /* ... */ }
```

#### gracefulStop and gracefulRampDown

```javascript
export const options = {
  scenarios: {
    my_scenario: {
      executor: 'ramping-vus',
      stages: [/* ... */],
      gracefulRampDown: '30s',  // VUs finishing current iteration get 30s before forced stop
    },
  },
  gracefulStop: '30s',          // global graceful stop period
};
```

### Practical Part

#### Exercise 4.1: Ramping Load Test (Classic Load Pattern)

Create `lesson4_ramp.js`:

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  stages: [
    { duration: '30s', target: 10  },  // ramp up
    { duration: '1m',  target: 10  },  // steady state
    { duration: '30s', target: 50  },  // scale up
    { duration: '1m',  target: 50  },  // higher steady state
    { duration: '30s', target: 0   },  // ramp down
  ],
  thresholds: {
    'http_req_duration': ['p(95)<2000'],
    'http_req_failed':   ['rate<0.01'],
  },
};

export default function () {
  const res = http.get('https://jsonplaceholder.typicode.com/posts');
  check(res, { 'status 200': (r) => r.status === 200 });
  sleep(Math.random() * 2 + 1); // random 1–3s think time
}
```

#### Exercise 4.2: Spike Test (Arrival Rate)

Create `lesson4_spike.js`:

```javascript
import http from 'k6/http';
import { check } from 'k6';

export const options = {
  scenarios: {
    normal_load: {
      executor: 'constant-arrival-rate',
      rate: 10,
      timeUnit: '1s',
      duration: '3m',
      preAllocatedVUs: 20,
      maxVUs: 50,
    },
    spike: {
      executor: 'ramping-arrival-rate',
      startRate: 10,
      timeUnit: '1s',
      preAllocatedVUs: 100,
      maxVUs: 500,
      stages: [
        { duration: '1m30s', target: 10  },  // normal
        { duration: '15s',   target: 200 },  // SPIKE
        { duration: '15s',   target: 10  },  // recover
        { duration: '1m',    target: 10  },  // normal tail
      ],
    },
  },
  thresholds: {
    'http_req_duration': ['p(99)<5000'],
    'http_req_failed':   ['rate<0.05'],
  },
};

export default function () {
  const res = http.get('https://dummyjson.com/products?limit=10');
  check(res, { '200': (r) => r.status === 200 });
}
```

#### Exercise 4.3: Multi-Scenario Workload Mix

Create `lesson4_scenarios.js`:

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

const BASE_DUMMY = 'https://dummyjson.com';
const BASE_JP    = 'https://jsonplaceholder.typicode.com';

export const options = {
  scenarios: {
    // High-frequency read traffic
    read_products: {
      executor:  'constant-arrival-rate',
      rate:      20,
      timeUnit:  '1s',
      duration:  '2m',
      preAllocatedVUs: 30,
      maxVUs:    100,
      exec:      'readProducts',
      tags:      { scenario: 'read' },
    },
    // Low-frequency write traffic
    create_posts: {
      executor:  'constant-vus',
      vus:       5,
      duration:  '2m',
      exec:      'createPost',
      tags:      { scenario: 'write' },
    },
    // Occasional search
    search_users: {
      executor:  'ramping-vus',
      startVUs:  0,
      stages: [
        { duration: '30s', target: 10 },
        { duration: '1m',  target: 10 },
        { duration: '30s', target: 0  },
      ],
      exec: 'searchUsers',
      tags: { scenario: 'search' },
    },
  },
  thresholds: {
    'http_req_duration{scenario:read}':   ['p(95)<1000'],
    'http_req_duration{scenario:write}':  ['p(95)<2000'],
    'http_req_duration{scenario:search}': ['p(95)<1500'],
    'http_req_failed': ['rate<0.01'],
  },
};

export function readProducts() {
  const id  = Math.floor(Math.random() * 100) + 1;
  const res = http.get(`${BASE_DUMMY}/products/${id}`, { tags: { scenario: 'read' } });
  check(res, { 'read 200': (r) => r.status === 200 });
  sleep(0.5);
}

export function createPost() {
  const payload = JSON.stringify({
    title:  `Load Test Post ${Date.now()}`,
    body:   'Created during k6 load test',
    userId: Math.floor(Math.random() * 10) + 1,
  });
  const res = http.post(`${BASE_JP}/posts`, payload, {
    headers: { 'Content-Type': 'application/json' },
    tags:    { scenario: 'write' },
  });
  check(res, { 'create 201': (r) => r.status === 201 });
  sleep(2);
}

export function searchUsers() {
  const res = http.get(`${BASE_DUMMY}/users/search?q=John`, { tags: { scenario: 'search' } });
  check(res, {
    'search 200':      (r) => r.status === 200,
    'results present': (r) => r.json('users').length >= 0,
  });
  sleep(1);
}
```

#### Exercise 4.4: Soak Test Pattern

Create `lesson4_soak.js`:

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

// Soak test: moderate load sustained for a long time
// (Reduce duration to 5m for practice; real soak tests run for hours)
export const options = {
  stages: [
    { duration: '2m',  target: 30 },  // ramp-up
    { duration: '10m', target: 30 },  // sustained load  ← extend to hours for real test
    { duration: '2m',  target: 0  },  // ramp-down
  ],
  thresholds: {
    'http_req_duration': ['p(95)<2000'],
    'http_req_failed':   ['rate<0.01'],
  },
};

export default function () {
  // Rotate through multiple endpoints to simulate real traffic mix
  const endpoints = [
    'https://dummyjson.com/products?limit=10',
    'https://dummyjson.com/users?limit=5',
    'https://dummyjson.com/posts?limit=10',
    'https://jsonplaceholder.typicode.com/comments?postId=1',
  ];

  const url = endpoints[Math.floor(Math.random() * endpoints.length)];
  const res = http.get(url);
  check(res, { '200': (r) => r.status === 200 });

  sleep(Math.random() * 2 + 1);
}
```

### Control Questions

1. **What is the key difference between `ramping-vus` and `ramping-arrival-rate` executors?**
   <details>
   <summary>Answer</summary>
   `ramping-vus` controls the number of concurrent virtual users — throughput varies based on how fast each VU iterates. `ramping-arrival-rate` controls the rate of new iterations per second — k6 dynamically adjusts VU count to sustain the target rate regardless of response time.
   </details>

2. **When would you use `constant-arrival-rate` instead of `constant-vus`?**
   <details>
   <summary>Answer</summary>
   Use `constant-arrival-rate` when you need to test at a specific RPS (e.g., "validate system at 100 req/s"). Use `constant-vus` when you want to model a specific number of concurrent users. Arrival rate is better for SLA validation; VUs are better for concurrency modeling.
   </details>

3. **What does the `startTime` property in a scenario do?**
   <details>
   <summary>Answer</summary>
   It delays the start of that scenario relative to the overall test start. For example, `startTime: '2m'` means that scenario begins 2 minutes after the test starts. This is useful for sequencing — e.g., start a write scenario after a read scenario has warmed up.
   </details>

4. **If a test has `stages: [{ duration: '1m', target: 100 }]`, how many VUs are active at the 30-second mark?**
   <details>
   <summary>Answer</summary>
   Approximately 50 VUs. k6 linearly interpolates between the start VUs (0 by default) and the target (100) over the stage duration. At the halfway point (30s of 60s), approximately 50 VUs are active.
   </details>

5. **What is `gracefulRampDown` and why is it important?**
   <details>
   <summary>Answer</summary>
   `gracefulRampDown` gives VUs that are in the middle of an iteration extra time to complete before being forcibly stopped during ramp-down. Without it, k6 abruptly kills VUs mid-iteration, which can cause incomplete transactions and skew error rates in results.
   </details>

---

## Lesson 5: Parameterization and Test Data Management

### Theoretical Part

#### Why Parameterize?

- Prevent server-side caching from masking real performance
- Simulate different users with unique credentials
- Test with realistic, varied datasets
- Avoid unique-constraint violations on write endpoints

#### Parameterization Techniques in k6

**1. Hardcoded arrays**
```javascript
const users = ['alice', 'bob', 'carol'];
const user  = users[Math.floor(Math.random() * users.length)];
```

**2. SharedArray — Efficient JSON/CSV data loading**
```javascript
import { SharedArray } from 'k6/data';
const data = new SharedArray('name', function () {
  return JSON.parse(open('./data.json'));
});
```
SharedArray loads data **once** and shares it across all VUs — drastically reducing memory usage for large datasets.

**3. `open()` — Read files at init time**
```javascript
const raw = open('./data.csv'); // only usable in init context
```

**4. Environment variables**
```javascript
const API_KEY = __ENV.API_KEY; // k6 run -e API_KEY=secret script.js
```

#### k6 Built-in Variables

| Variable | Description |
|---|---|
| `__VU` | Current VU number (1-based) |
| `__ITER` | Current iteration number for this VU (0-based) |
| `__ENV` | Environment variables object |

### Practical Part

#### Exercise 5.1: Data-Driven Tests with SharedArray

**Create `test-data/users.json`:**
```json
[
  { "username": "emilys",   "password": "emilyspass" },
  { "username": "michaelw", "password": "michaelwpass" },
  { "username": "sophiab",  "password": "sophiabpass" }
]
```

Create `lesson5_shared_array.js`:

```javascript
import http from 'k6/http';
import { SharedArray } from 'k6/data';
import { check, sleep } from 'k6';

const users = new SharedArray('users', function () {
  return JSON.parse(open('./test-data/users.json'));
});

export const options = {
  vus: 5,
  duration: '30s',
  thresholds: { 'http_req_failed': ['rate<0.05'] },
};

export default function () {
  const user = users[(__VU - 1) % users.length]; // round-robin

  const loginRes = http.post(
    'https://dummyjson.com/auth/login',
    JSON.stringify({ username: user.username, password: user.password }),
    { headers: { 'Content-Type': 'application/json' } }
  );

  check(loginRes, {
    'login 200': (r) => r.status === 200,
    'has token': (r) => r.json('accessToken') !== undefined,
  });

  if (loginRes.status === 200) {
    const token = loginRes.json('accessToken');
    const meRes = http.get('https://dummyjson.com/auth/me', {
      headers: { Authorization: `Bearer ${token}` },
    });
    check(meRes, { 'me 200': (r) => r.status === 200 });
    console.log(`VU ${__VU}: logged in as ${user.username} (iter ${__ITER})`);
  }

  sleep(1);
}
```

#### Exercise 5.2: Random Data Generation

Create `lesson5_random_data.js`:

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = { vus: 10, duration: '20s' };

function randomInt(min, max) {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

const categories = ['smartphones', 'laptops', 'fragrances', 'skincare', 'groceries'];

export default function () {
  // Random product
  const prodRes = http.get(`https://dummyjson.com/products/${randomInt(1, 100)}`);
  check(prodRes, { 'product 200': (r) => r.status === 200 });

  // Random category
  const cat    = categories[randomInt(0, categories.length - 1)];
  const catRes = http.get(`https://dummyjson.com/products/category/${cat}`);
  check(catRes, { 'category 200': (r) => r.status === 200 });

  // Unique post per VU + iteration
  const postRes = http.post(
    'https://dummyjson.com/posts/add',
    JSON.stringify({ title: `Post by VU${__VU} iter${__ITER}`, body: 'k6 test', userId: randomInt(1, 10) }),
    { headers: { 'Content-Type': 'application/json' } }
  );
  check(postRes, { 'post 201': (r) => r.status === 201 });

  sleep(1);
}
```

#### Exercise 5.3: Environment Variables for Configuration

```bash
k6 run -e BASE_URL=https://dummyjson.com -e ENVIRONMENT=staging lesson5_env.js
```

Create `lesson5_env.js`:

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

const BASE_URL = __ENV.BASE_URL    || 'https://dummyjson.com';
const ENV_NAME = __ENV.ENVIRONMENT || 'development';
const API_KEY  = __ENV.API_KEY     || '';

export const options = { vus: 3, duration: '15s' };

export function setup() {
  console.log(`Target: ${BASE_URL} (${ENV_NAME})`);
}

export default function () {
  const res = http.get(`${BASE_URL}/products?limit=5`, {
    headers: { 'X-API-Key': API_KEY },
  });
  check(res, { '200': (r) => r.status === 200 });
  sleep(1);
}
```

#### Exercise 5.4: Rotating Users — Each VU Gets a Unique User

Create `lesson5_rotating_users.js`:

```javascript
import http from 'k6/http';
import { SharedArray } from 'k6/data';
import { check, sleep } from 'k6';

// 100 users from dummyjson (IDs 1-100)
const userIds = new SharedArray('ids', function () {
  const ids = [];
  for (let i = 1; i <= 100; i++) ids.push(i);
  return ids;
});

export const options = {
  vus: 20,
  duration: '30s',
};

export default function () {
  // Each VU gets a deterministic user that does not overlap with other VUs
  const userId = userIds[(__VU - 1) % userIds.length];

  const res = http.get(`https://dummyjson.com/users/${userId}`);
  check(res, {
    'user found': (r) => r.status === 200,
    'id matches': (r) => r.json('id') === userId,
  });

  // Fetch that user's posts
  const postsRes = http.get(`https://dummyjson.com/posts/user/${userId}`);
  check(postsRes, { 'posts 200': (r) => r.status === 200 });

  sleep(1);
}
```

### Control Questions

1. **What is the advantage of `SharedArray` over a regular JavaScript array for test data?**
   <details>
   <summary>Answer</summary>
   SharedArray stores data once in memory and shares it across all VUs via a copy-on-read mechanism. A regular array is copied into each VU's memory space, multiplying memory usage by VU count. For large datasets (thousands of rows), SharedArray drastically reduces memory consumption.
   </details>

2. **How would you ensure each VU uses a unique, non-overlapping slice of test data?**
   <details>
   <summary>Answer</summary>
   Use `__VU` as an offset: `data[(__VU - 1) % data.length]` for round-robin, or pre-partition the array: `data.slice((__VU-1) * sliceSize, __VU * sliceSize)` for strict partitioning. Combine with `__ITER` for per-iteration uniqueness across a run.
   </details>

3. **How do you pass an API key to a k6 script without hardcoding it?**
   <details>
   <summary>Answer</summary>
   Use `-e`: `k6 run -e API_KEY=mykey script.js`. Inside the script: `const key = __ENV.API_KEY`. This keeps secrets out of source files and allows different values per environment without changing the script.
   </details>

4. **When is `open()` called and what is its limitation?**
   <details>
   <summary>Answer</summary>
   `open()` is called only during the init phase (before setup/VUs start). It reads local files only — no remote URLs. It cannot be used inside `default()`, `setup()`, or `teardown()`.
   </details>

5. **What is the difference between using `__VU` vs `Math.random()` for data selection?**
   <details>
   <summary>Answer</summary>
   `__VU` is deterministic — the same VU always maps to the same data, enabling reproducibility and preventing write collisions in concurrent scenarios. `Math.random()` is non-deterministic and gives a broader distribution, better suited for read-only tests where collisions are acceptable.
   </details>

---

## Lesson 6: Groups, Tags, and Custom Metrics

### Theoretical Part

#### Groups — Named Business Transactions

Groups mark sections of code for aggregated timing metrics:

```javascript
import { group } from 'k6';

group('Login Flow', function () {
  http.post('/auth', credentials);
});

group('Product Browse', function () {
  http.get('/products');
  group('View Detail', function () {   // nested groups
    http.get('/products/1');
  });
});
```
k6 reports `group_duration{group::Login Flow}` per named group in the summary.

#### Tags — Metric Metadata for Filtering

```javascript
// Request-level tag
http.get('/api/users', { tags: { endpoint: 'users', version: 'v2' } });

// Target tagged metrics in thresholds
thresholds: {
  'http_req_duration{endpoint:users}': ['p(95)<500'],
}

// Test-wide tags — applied to ALL metrics in this run
export const options = {
  tags: { environment: 'staging', team: 'backend' },
};
```

#### Custom Metric Types

| Type | Use case | Example |
|---|---|---|
| `Counter` | Cumulative count | total errors, total logins |
| `Gauge` | Snapshot value (last wins) | active sessions, queue depth |
| `Rate` | Pass/fail ratio | success rate, SLA compliance |
| `Trend` | Distribution (avg/p95/max) | business transaction latency |

```javascript
import { Counter, Gauge, Rate, Trend } from 'k6/metrics';

const loginErrors   = new Counter('login_errors');
const successRate   = new Rate('success_rate');
const bizDuration   = new Trend('business_tx_ms', true); // true = milliseconds
const activeSess    = new Gauge('active_sessions');

// Record inside default()
loginErrors.add(1, { type: 'timeout' });
successRate.add(res.status === 200);
bizDuration.add(res.timings.duration);
activeSess.add(__VU);
```

#### Output Formats

```bash
k6 run --out json=results.json script.js          # NDJSON file
k6 run --out csv=results.csv  script.js           # CSV
k6 run --out influxdb=http://localhost:8086/k6    # InfluxDB → Grafana
k6 run --out json=r.json --out csv=r.csv script.js  # Multiple simultaneously
```

### Practical Part

#### Exercise 6.1: Groups for Business Transaction Timing

Create `lesson6_groups.js`:

```javascript
import http from 'k6/http';
import { group, check, sleep } from 'k6';

export const options = {
  vus: 10,
  duration: '30s',
  thresholds: {
    'group_duration{group::Product Catalog}': ['avg<2000'],
    'group_duration{group::User Account}':    ['avg<1500'],
    'http_req_failed': ['rate<0.01'],
  },
};

const BASE = 'https://dummyjson.com';

export default function () {
  group('Product Catalog', function () {
    const listRes = http.get(`${BASE}/products?limit=10`);
    check(listRes, { 'list 200': (r) => r.status === 200 });

    const id      = Math.floor(Math.random() * 100) + 1;
    const detRes  = http.get(`${BASE}/products/${id}`);
    check(detRes, { 'detail 200': (r) => r.status === 200 });

    group('Search', function () {
      const terms   = ['phone', 'laptop', 'cream'];
      const term    = terms[Math.floor(Math.random() * terms.length)];
      const srchRes = http.get(`${BASE}/products/search?q=${term}`);
      check(srchRes, { 'search 200': (r) => r.status === 200 });
    });
  });

  sleep(0.5);

  group('User Account', function () {
    const uid     = Math.floor(Math.random() * 100) + 1;
    const userRes = http.get(`${BASE}/users/${uid}`);
    check(userRes, { 'user 200': (r) => r.status === 200 });
  });

  sleep(1);
}
```

#### Exercise 6.2: Custom Metrics

Create `lesson6_custom_metrics.js`:

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Counter, Rate, Trend } from 'k6/metrics';

const loginErrors      = new Counter('login_errors');
const loginSuccessRate = new Rate('login_success_rate');
const loginDuration    = new Trend('login_duration_ms', true);

export const options = {
  vus: 10,
  duration: '30s',
  thresholds: {
    'login_success_rate': ['rate>0.95'],
    'login_duration_ms':  ['p(95)<3000'],
    'login_errors':       ['count<10'],
  },
};

export default function () {
  const res     = http.post(
    'https://dummyjson.com/auth/login',
    JSON.stringify({ username: 'emilys', password: 'emilyspass' }),
    { headers: { 'Content-Type': 'application/json' } }
  );
  const success = res.status === 200;
  loginSuccessRate.add(success);
  loginDuration.add(res.timings.duration);
  if (!success) loginErrors.add(1, { status: String(res.status) });

  check(res, {
    'login 200': (r) => r.status === 200,
    'has token': (r) => success && r.json('accessToken').length > 0,
  });
  sleep(1);
}
```

#### Exercise 6.3: Per-Endpoint Thresholds with Tags

Create `lesson6_tags.js`:

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  vus: 15,
  duration: '30s',
  tags: { environment: 'staging', version: 'v2' },
  thresholds: {
    'http_req_duration{service:products}': ['p(95)<1500'],
    'http_req_duration{service:users}':    ['p(95)<1000'],
    'http_req_duration{service:posts}':    ['p(95)<2000'],
    'http_req_failed': ['rate<0.01'],
  },
};

export default function () {
  http.get('https://dummyjson.com/products?limit=5',
    { tags: { service: 'products' } });

  http.get(`https://dummyjson.com/users/${Math.ceil(Math.random() * 10)}`,
    { tags: { service: 'users' } });

  http.post(
    'https://dummyjson.com/posts/add',
    JSON.stringify({ title: 'Tagged post', userId: 1 }),
    { headers: { 'Content-Type': 'application/json' }, tags: { service: 'posts' } }
  );

  sleep(1);
}
```

#### Exercise 6.4: Comprehensive Scenario with All Custom Metric Types

Create `lesson6_all_metrics.js`:

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Counter, Rate, Trend, Gauge } from 'k6/metrics';

const totalRequests   = new Counter('total_api_requests');
const slaCompliance   = new Rate('sla_compliance_rate');
const apiLatency      = new Trend('api_latency_ms', true);
const currentVUs      = new Gauge('current_vus');

const SLA_MS = 2000; // SLA = 2 second response time

export const options = {
  vus: 10,
  duration: '30s',
  thresholds: {
    'sla_compliance_rate': ['rate>0.95'],
    'api_latency_ms':      ['p(95)<2500'],
    'total_api_requests':  ['count>50'],
  },
};

export default function () {
  currentVUs.add(__VU);

  const endpoints = [
    'https://dummyjson.com/products?limit=5',
    'https://dummyjson.com/users?limit=5',
    `https://jsonplaceholder.typicode.com/posts/${Math.ceil(Math.random() * 100)}`,
  ];

  for (const url of endpoints) {
    const res = http.get(url);
    const ok  = res.status === 200;

    totalRequests.add(1);
    apiLatency.add(res.timings.duration);
    slaCompliance.add(res.timings.duration < SLA_MS);

    check(res, { '200': (r) => r.status === 200 });
  }

  sleep(1);
}
```

### Control Questions

1. **What is the difference between a `group` and a `tag` in k6?**
   <details>
   <summary>Answer</summary>
   A `group` organizes code blocks and creates a `group_duration` metric for the block — ideal for business transaction timing. A `tag` is key-value metadata attached to metrics for filtering and threshold scoping — more flexible and applies to individual requests or the entire test run.
   </details>

2. **Which custom metric type would you use to measure the percentage of transactions that completed within SLA?**
   <details>
   <summary>Answer</summary>
   `Rate` — add `true` for each request meeting the SLA, `false` for those that don't. Then set a threshold like `'my_rate': ['rate>0.99']` to require 99% compliance.
   </details>

3. **What is the difference between `Counter` and `Gauge` metrics?**
   <details>
   <summary>Answer</summary>
   `Counter` accumulates monotonically (always increases — e.g., total error count). `Gauge` is a snapshot value that can go up or down (e.g., current active connections). The summary shows Counter as total + rate; Gauge shows the last recorded value.
   </details>

4. **How do test-level tags differ from request-level tags?**
   <details>
   <summary>Answer</summary>
   Test-level tags (`options.tags`) are applied to ALL metrics produced by the test run — good for labeling results with environment, version, or team for external storage. Request-level tags (`params.tags`) apply only to that specific request's metrics, enabling per-endpoint threshold filtering.
   </details>

5. **Why output results to InfluxDB instead of relying on the default console summary?**
   <details>
   <summary>Answer</summary>
   InfluxDB stores time-series data visualizable in Grafana dashboards in real time during the test — enabling live monitoring of trends, VU count, and error rates. It also enables historical comparison across test runs, which the one-shot console summary cannot provide.
   </details>

---

## Lesson 7: Correlation, Session Management, and Dynamic Workflows

### Theoretical Part

#### What is Correlation?

Correlation is **extracting dynamic values from responses** and **injecting them into subsequent requests**. It is essential for:

- Authentication flows (JWT tokens, session cookies)
- Chained API calls (create a resource, then use its ID)
- CSRF token handling in form-based applications

#### Extracting Data from Responses

```javascript
// From JSON
const token  = res.json('accessToken');       // direct field
const id     = res.json('data.user.id');      // nested
const first  = res.json('items[0].id');       // array index

// From response headers
const cookie = res.headers['Set-Cookie'];
const loc    = res.headers['Location'];

// From response body with regex
const match  = res.body.match(/"token":"([^"]+)"/);
const token2 = match ? match[1] : null;
```

#### Cookie Handling

k6 automatically manages cookies per VU using an isolated CookieJar — mimicking separate browser sessions:

```javascript
// Auto: cookies from Set-Cookie headers are stored and resent automatically

// Manual jar access
const jar = http.cookieJar();
jar.set('https://example.com', 'sessionId', 'abc123');
const cookies = jar.cookiesForURL('https://example.com');
```

### Practical Part

#### Exercise 7.1: Full Authentication Flow with Token Reuse

Create `lesson7_auth_flow.js`:

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = { vus: 5, duration: '30s' };

const BASE  = 'https://dummyjson.com';
const JSONH = { headers: { 'Content-Type': 'application/json' } };

export default function () {
  // Step 1: Login
  const loginRes = http.post(
    `${BASE}/auth/login`,
    JSON.stringify({ username: 'emilys', password: 'emilyspass', expiresInMins: 30 }),
    JSONH
  );
  check(loginRes, { 'login 200': (r) => r.status === 200 });
  if (loginRes.status !== 200) return;

  const token  = loginRes.json('accessToken');
  const userId = loginRes.json('id');
  const auth   = { headers: { Authorization: `Bearer ${token}` } };

  sleep(0.5);

  // Step 2: Authenticated profile
  const meRes = http.get(`${BASE}/auth/me`, auth);
  check(meRes, {
    'me 200':         (r) => r.status === 200,
    'correct userId': (r) => r.json('id') === userId,
  });

  // Step 3: Protected resources
  const cartRes = http.get(`${BASE}/carts/user/${userId}`, auth);
  check(cartRes, { 'cart 200': (r) => r.status === 200 });

  // Step 4: Token refresh
  const refreshToken = loginRes.json('refreshToken');
  const refreshRes   = http.post(
    `${BASE}/auth/refresh`,
    JSON.stringify({ refreshToken, expiresInMins: 30 }),
    JSONH
  );
  check(refreshRes, {
    'refresh 200': (r) => r.status === 200,
    'new token':   (r) => r.json('accessToken') !== undefined,
  });

  sleep(1);
}
```

#### Exercise 7.2: Chained API Calls — CRUD in One Iteration

Create `lesson7_chained.js`:

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = { vus: 5, duration: '30s' };

const JP    = 'https://jsonplaceholder.typicode.com';
const JSONH = { headers: { 'Content-Type': 'application/json' } };

export default function () {
  // Create
  const createRes = http.post(`${JP}/posts`,
    JSON.stringify({ title: `VU${__VU} post`, body: 'Test', userId: 1 }), JSONH);
  check(createRes, { 'created 201': (r) => r.status === 201 });
  const postId = createRes.json('id');

  // Comment on it
  const commentRes = http.post(`${JP}/comments`,
    JSON.stringify({ postId, name: 'tester', email: 'test@test.com', body: 'Nice!' }), JSONH);
  check(commentRes, { 'comment 201': (r) => r.status === 201 });

  sleep(0.3);

  // Update
  const updateRes = http.patch(`${JP}/posts/${postId}`,
    JSON.stringify({ title: `VU${__VU} post (edited)` }), JSONH);
  check(updateRes, { 'updated 200': (r) => r.status === 200 });

  // Read all comments
  const commRes = http.get(`${JP}/posts/${postId}/comments`);
  check(commRes, { 'comments 200': (r) => r.status === 200 });

  // Delete
  const delRes = http.del(`${JP}/posts/${postId}`);
  check(delRes, { 'deleted 200': (r) => r.status === 200 });

  sleep(1);
}
```

#### Exercise 7.3: Cookie Session Simulation

Create `lesson7_cookies.js`:

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = { vus: 5, duration: '20s' };

export default function () {
  const BASE = 'https://httpbin.org';

  // Set cookies via redirect
  http.get(`${BASE}/cookies/set?sessionId=k6-${__VU}-${Date.now()}&lang=en`);

  // k6 sends them automatically on subsequent requests
  const getRes = http.get(`${BASE}/cookies`);
  check(getRes, {
    'cookies 200':      (r) => r.status === 200,
    'sessionId exists': (r) => r.json('cookies').sessionId !== undefined,
    'lang is en':       (r) => r.json('cookies').lang === 'en',
  });

  const jar     = http.cookieJar();
  const cookies = jar.cookiesForURL(BASE);
  console.log(`VU${__VU} cookie jar:`, JSON.stringify(cookies));

  sleep(1);
}
```

#### Exercise 7.4: Realistic E-Commerce User Journey

Create `lesson7_user_journey.js`:

```javascript
import http from 'k6/http';
import { check, group, sleep } from 'k6';
import { SharedArray } from 'k6/data';

const users = new SharedArray('users', () => [
  { username: 'emilys',   password: 'emilyspass' },
  { username: 'michaelw', password: 'michaelwpass' },
]);

export const options = {
  stages: [
    { duration: '30s', target: 5 },
    { duration: '1m',  target: 5 },
    { duration: '15s', target: 0 },
  ],
  thresholds: {
    'http_req_duration': ['p(95)<3000'],
    'http_req_failed':   ['rate<0.02'],
    'checks':            ['rate>0.95'],
  },
};

const BASE  = 'https://dummyjson.com';
const JSONH = { headers: { 'Content-Type': 'application/json' } };

export default function () {
  const user  = users[(__VU - 1) % users.length];
  let   token = null;

  group('1. Login', function () {
    const res = http.post(`${BASE}/auth/login`, JSON.stringify(user), JSONH);
    check(res, { 'login OK': (r) => r.status === 200 });
    if (res.status === 200) token = res.json('accessToken');
  });

  if (!token) return;
  const auth = { headers: { Authorization: `Bearer ${token}` } };

  sleep(1);

  group('2. Browse Products', function () {
    http.get(`${BASE}/products?limit=10`, auth);
    const id  = Math.floor(Math.random() * 30) + 1;
    const res = http.get(`${BASE}/products/${id}`, auth);
    check(res, { 'product OK': (r) => r.status === 200 });
  });

  sleep(Math.random() * 2 + 1);

  group('3. Search', function () {
    const q   = ['phone', 'cream', 'watch'][Math.floor(Math.random() * 3)];
    const res = http.get(`${BASE}/products/search?q=${q}`, auth);
    check(res, { 'search OK': (r) => r.status === 200 });
  });

  sleep(1);
}
```

### Control Questions

1. **What is correlation in performance testing and why is it necessary?**
   <details>
   <summary>Answer</summary>
   Correlation is extracting dynamic values (tokens, IDs, session keys) from server responses and injecting them into subsequent requests. It is necessary because modern applications use unique, time-limited values (JWTs, CSRF tokens, auto-generated IDs) that cannot be hardcoded in the test script.
   </details>

2. **How does k6 handle cookies between requests within the same VU?**
   <details>
   <summary>Answer</summary>
   Each VU has its own isolated CookieJar. Cookies from Set-Cookie response headers are stored automatically and sent with subsequent matching requests — exactly mimicking browser cookie behavior. VUs are fully isolated from each other, each simulating an independent browser session.
   </details>

3. **What happens if you forget to check whether login succeeded before using the extracted token?**
   <details>
   <summary>Answer</summary>
   Subsequent requests will use `undefined` as the token value, producing `Authorization: Bearer undefined` headers. These requests will likely return 401/403 errors, inflating the error rate with misleading results. Always verify login success and return early or handle gracefully if it fails.
   </details>

4. **How do you extract a value from a response header in k6?**
   <details>
   <summary>Answer</summary>
   Access `res.headers['Header-Name']` directly. Example: `const loc = res.headers['Location']`. Note that k6's response headers object uses the exact casing returned by the server.
   </details>

5. **Why is per-VU cookie isolation important?**
   <details>
   <summary>Answer</summary>
   It ensures each VU simulates a completely independent user with its own session. If VUs shared a cookie jar, all VUs would share one session, making the test represent a single user rather than N concurrent users — completely invalidating concurrency modeling.
   </details>

---

## Lesson 8: Advanced Scenarios — gRPC, WebSocket, and Browser Testing

### Theoretical Part

#### k6 Protocol Support

k6 goes beyond HTTP to support modern application protocols:

| Protocol | Module | Use case |
|---|---|---|
| HTTP/1.1, HTTP/2 | `k6/http` | REST APIs, web apps |
| WebSocket | `k6/ws` | Real-time chat, live feeds |
| gRPC | `k6/net/grpc` | Microservices, internal APIs |
| Browser | `k6/browser` (experimental) | UI testing, Core Web Vitals |

#### WebSocket Testing

```javascript
import ws from 'k6/ws';
import { check } from 'k6';

export default function () {
  const url  = 'wss://ws.example.com/socket';
  const res  = ws.connect(url, {}, function (socket) {
    socket.on('open',    () => { socket.send('Hello'); });
    socket.on('message', (data) => { check(data, { 'got message': (d) => d.length > 0 }); });
    socket.on('error',   (e)    => { console.error('WS error:', e); });
    socket.setTimeout(function () { socket.close(); }, 5000);
  });
  check(res, { 'status 101': (r) => r && r.status === 101 });
}
```

#### gRPC Testing

```javascript
import grpc from 'k6/net/grpc';
import { check } from 'k6';

const client = new grpc.Client();
client.load(['./protos'], 'helloworld.proto');

export default function () {
  client.connect('grpc.example.com:443', { plaintext: false });

  const res = client.invoke('helloworld.Greeter/SayHello', { name: 'k6' });
  check(res, { 'status OK': (r) => r && r.status === grpc.StatusOK });

  client.close();
}
```

#### k6 Browser (Experimental)

```javascript
import { chromium } from 'k6/browser';

export const options = {
  scenarios: {
    browser_test: {
      executor: 'shared-iterations',
      options: { browser: { type: 'chromium' } },
    },
  },
};

export default async function () {
  const browser = chromium.launch();
  const page    = browser.newPage();

  await page.goto('https://example.com');
  await page.screenshot({ path: 'screenshot.png' });

  browser.close();
}
```

### Practical Part

#### Exercise 8.1: WebSocket Load Test

Create `lesson8_websocket.js`:

```javascript
import ws   from 'k6/ws';
import { check, sleep } from 'k6';
import { Counter, Trend } from 'k6/metrics';

const messagesReceived = new Counter('ws_messages_received');
const messageLatency   = new Trend('ws_message_latency_ms', true);

export const options = {
  vus: 10,
  duration: '30s',
  thresholds: {
    'ws_messages_received': ['count>50'],
    'ws_message_latency_ms': ['avg<1000'],
  },
};

export default function () {
  // Use a public WebSocket echo server
  const url = 'wss://echo.websocket.org/';

  const res = ws.connect(url, {}, function (socket) {
    socket.on('open', function () {
      console.log(`VU ${__VU}: WebSocket connected`);

      // Send 5 messages with latency tracking
      for (let i = 0; i < 5; i++) {
        const sentAt = Date.now();
        const msg    = JSON.stringify({ seq: i, vu: __VU, ts: sentAt });
        socket.send(msg);

        socket.on('message', function (data) {
          const latency = Date.now() - sentAt;
          messagesReceived.add(1);
          messageLatency.add(latency);

          check(data, {
            'echo non-empty': (d) => d.length > 0,
            'is valid JSON':  (d) => { try { JSON.parse(d); return true; } catch { return false; } },
          });
        });

        sleep(0.5);
      }
    });

    socket.on('error', (e) => console.error('WS Error:', e.error()));

    // Close after 3 seconds
    socket.setTimeout(function () { socket.close(); }, 3000);
  });

  check(res, { 'WS connected (101)': (r) => r && r.status === 101 });

  sleep(1);
}
```

#### Exercise 8.2: Mixed Protocol Test — HTTP + WebSocket

Create `lesson8_mixed_protocol.js`:

```javascript
import http from 'k6/http';
import ws   from 'k6/ws';
import { check, group, sleep } from 'k6';

export const options = {
  scenarios: {
    http_users: {
      executor:  'constant-vus',
      vus:       10,
      duration:  '30s',
      exec:      'httpTest',
    },
    ws_users: {
      executor:  'constant-vus',
      vus:       3,
      duration:  '30s',
      exec:      'wsTest',
    },
  },
};

export function httpTest() {
  group('REST API', function () {
    const res = http.get('https://dummyjson.com/products?limit=5');
    check(res, { 'HTTP 200': (r) => r.status === 200 });
  });
  sleep(1);
}

export function wsTest() {
  const res = ws.connect('wss://echo.websocket.org/', {}, function (socket) {
    socket.on('open', () => socket.send(`ping from VU${__VU}`));
    socket.on('message', (d) => {
      check(d, { 'WS echo received': (data) => data.length > 0 });
      socket.close();
    });
  });
  check(res, { 'WS 101': (r) => r && r.status === 101 });
  sleep(1);
}
```

#### Exercise 8.3: Comprehensive HTTP/2 and Performance Headers Analysis

Create `lesson8_http2_analysis.js`:

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Trend } from 'k6/metrics';

const ttfbTrend = new Trend('time_to_first_byte_ms', true);

export const options = { vus: 5, duration: '20s' };

export default function () {
  // httpbin provides detailed request info including HTTP version
  const res = http.get('https://httpbin.org/get');

  check(res, { '200 OK': (r) => r.status === 200 });

  // Track TTFB separately
  ttfbTrend.add(res.timings.waiting);

  // Log protocol and timing breakdown
  console.log([
    `Status: ${res.status}`,
    `Protocol: ${res.proto}`,
    `TTFB: ${res.timings.waiting.toFixed(0)}ms`,
    `Total: ${res.timings.duration.toFixed(0)}ms`,
    `TLS: ${res.timings.tls_handshaking.toFixed(0)}ms`,
  ].join(' | '));

  // Batch to compare concurrent vs sequential
  const start     = Date.now();
  const responses = http.batch([
    ['GET', 'https://jsonplaceholder.typicode.com/posts/1'],
    ['GET', 'https://jsonplaceholder.typicode.com/users/1'],
    ['GET', 'https://jsonplaceholder.typicode.com/albums/1'],
  ]);
  const batchTime = Date.now() - start;

  responses.forEach((r, i) => check(r, { [`batch[${i}] 200`]: (resp) => resp.status === 200 }));
  console.log(`Batch of 3 completed in ${batchTime}ms`);

  sleep(1);
}
```

### Control Questions

1. **What does HTTP status 101 indicate in a WebSocket connection attempt?**
   <details>
   <summary>Answer</summary>
   101 Switching Protocols — the server has agreed to the WebSocket upgrade. The connection transitions from HTTP to the WebSocket protocol. If you do not receive 101, the WebSocket connection was not established.
   </details>

2. **Why do WebSocket tests use event handlers (`socket.on`) instead of sequential code?**
   <details>
   <summary>Answer</summary>
   WebSocket is event-driven and asynchronous — messages can arrive at any time. Event handlers (`open`, `message`, `close`, `error`) respond to connection lifecycle events. Sequential code cannot wait for unpredictable incoming messages without blocking.
   </details>

3. **In a mixed HTTP + WebSocket scenario using k6 scenarios, how do you prevent both from running the same `default()` function?**
   <details>
   <summary>Answer</summary>
   Use the `exec` property in each scenario definition to point to a named exported function. For example: `exec: 'httpTest'` and `exec: 'wsTest'`. Each scenario calls its own function independently.
   </details>

4. **What is `res.timings.waiting` and why is it often called TTFB?**
   <details>
   <summary>Answer</summary>
   `waiting` is the time between completing the request send and receiving the first byte of the response — Time To First Byte (TTFB). It reflects server-side processing time (minus network latency) and is the primary indicator of backend application performance.
   </details>

5. **What is the benefit of testing with HTTP/2 vs HTTP/1.1 in k6?**
   <details>
   <summary>Answer</summary>
   HTTP/2 supports multiplexing (multiple requests over one TCP connection), header compression, and server push — making it faster for parallel requests. k6 supports HTTP/2 automatically when the server advertises it. Testing with HTTP/2 gives more realistic results for modern applications.
   </details>

---

## Lesson 9: CI/CD Integration and Test Automation

### Theoretical Part

#### Running k6 in CI/CD

k6 is designed for CI/CD integration. Key capabilities:

- **Exit codes**: k6 exits 0 on pass, non-zero on threshold failure — CI-native pass/fail
- **No GUI**: k6 runs headlessly by default
- **Parameterized**: All options overridable via CLI flags and env vars
- **Output formats**: JSON, CSV, InfluxDB for result storage and trending

#### Command-Line Options Reference

```bash
# Basic run
k6 run script.js

# Override options (higher priority than script options)
k6 run --vus 100 --duration 5m script.js

# Stages via CLI
k6 run --stage 30s:100 --stage 2m:100 --stage 30s:0 script.js

# Environment variables
k6 run -e BASE_URL=https://prod.example.com -e API_KEY=secret script.js

# Output results
k6 run --out json=results.json --out csv=results.csv script.js

# Set threshold (override)
k6 run --threshold 'http_req_duration:p(95)<500' script.js

# Summary export
k6 run --summary-export=summary.json script.js

# No summary (quiet mode for CI)
k6 run --no-summary script.js

# Quiet (minimal output)
k6 run -q script.js
```

#### Interpreting k6 Exit Codes

| Exit Code | Meaning |
|---|---|
| 0 | Test passed (all thresholds met) |
| 97 | Test passed but some thresholds failed |
| 99 | Test setup error / script error |
| 108 | Interrupted (Ctrl+C or timeout) |

In CI, a non-zero exit code should fail the pipeline stage.

#### k6 Cloud (Grafana Cloud)

```bash
# Login
k6 cloud login --token your-api-token

# Run in cloud (distributed, from multiple geo regions)
k6 cloud script.js

# Stream results to cloud while running locally
k6 run --out cloud script.js
```

### Practical Part

#### Exercise 9.1: Parameterized Script for Multiple Environments

Create `lesson9_ci_script.js`:

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

// All configurable via -e flags
const BASE_URL   = __ENV.BASE_URL    || 'https://dummyjson.com';
const VUS        = parseInt(__ENV.VUS)       || 10;
const DURATION   = __ENV.DURATION            || '30s';
const P95_SLA_MS = parseInt(__ENV.P95_SLA)   || 2000;
const ERROR_RATE = parseFloat(__ENV.ERR_RATE) || 0.01;

export const options = {
  vus:      VUS,
  duration: DURATION,
  thresholds: {
    'http_req_duration': [`p(95)<${P95_SLA_MS}`],
    'http_req_failed':   [`rate<${ERROR_RATE}`],
    'checks':            ['rate>0.99'],
  },
  summaryTrendStats: ['avg', 'min', 'med', 'max', 'p(90)', 'p(95)', 'p(99)', 'count'],
};

export default function () {
  const res = http.get(`${BASE_URL}/products?limit=10`);
  check(res, {
    'status 200':     (r) => r.status === 200,
    'has products':   (r) => r.json('products').length > 0,
    'within SLA':     (r) => r.timings.duration < P95_SLA_MS,
  });
  sleep(1);
}
```

**Environment-specific run commands:**
```bash
# Smoke test
k6 run -e BASE_URL=https://dummyjson.com -e VUS=1 -e DURATION=30s lesson9_ci_script.js

# Load test
k6 run -e BASE_URL=https://dummyjson.com -e VUS=50 -e DURATION=5m -e P95_SLA=1500 lesson9_ci_script.js

# Stress test
k6 run -e BASE_URL=https://dummyjson.com -e VUS=200 -e DURATION=5m -e P95_SLA=3000 lesson9_ci_script.js
```

#### Exercise 9.2: GitHub Actions Pipeline

Create `.github/workflows/k6-performance.yml`:

```yaml
name: k6 Performance Tests

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  schedule:
    - cron: '0 3 * * *'   # Daily at 3 AM

jobs:
  smoke-test:
    name: Smoke Test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install k6
        run: |
          sudo gpg -k
          sudo gpg --no-default-keyring             --keyring /usr/share/keyrings/k6-archive-keyring.gpg             --keyserver hkp://keyserver.ubuntu.com:80             --recv-keys C5AD17C747E3415A3642D57D77C6C491D6AC1D69
          echo "deb [signed-by=/usr/share/keyrings/k6-archive-keyring.gpg]             https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list
          sudo apt-get update && sudo apt-get install k6

      - name: Run Smoke Test
        run: |
          k6 run             -e VUS=1             -e DURATION=30s             -e BASE_URL=https://dummyjson.com             --out json=smoke-results.json             --summary-export=smoke-summary.json             lesson9_ci_script.js

      - name: Upload Smoke Results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: smoke-test-results
          path: |
            smoke-results.json
            smoke-summary.json

  load-test:
    name: Load Test
    runs-on: ubuntu-latest
    needs: smoke-test
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4

      - name: Install k6
        run: |
          sudo apt-get update && sudo apt-get install -y gnupg
          sudo gpg --no-default-keyring             --keyring /usr/share/keyrings/k6-archive-keyring.gpg             --keyserver hkp://keyserver.ubuntu.com:80             --recv-keys C5AD17C747E3415A3642D57D77C6C491D6AC1D69
          echo "deb [signed-by=/usr/share/keyrings/k6-archive-keyring.gpg]             https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list
          sudo apt-get update && sudo apt-get install k6

      - name: Run Load Test
        run: |
          k6 run             -e VUS=20             -e DURATION=2m             -e P95_SLA=2000             -e ERR_RATE=0.01             --out json=load-results.json             --summary-export=load-summary.json             lesson9_ci_script.js

      - name: Check Thresholds
        run: |
          # k6 already exits non-zero if thresholds fail
          echo "Load test completed. Check job status for threshold results."

      - name: Upload Load Results
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: load-test-results
          path: |
            load-results.json
            load-summary.json
```

#### Exercise 9.3: Threshold Validation Script

Create `validate_results.py`:

```python
#!/usr/bin/env python3
# Parse k6 --summary-export JSON and validate SLA thresholds.
import json
import sys

def validate(summary_file, thresholds):
    with open(summary_file) as f:
        data = json.load(f)

    metrics = data.get('metrics', {})
    violations = []
    passed = []

    for metric_name, criteria_list in thresholds.items():
        metric = metrics.get(metric_name, {})
        for criteria in criteria_list:
            # Parse expression like "p(95)<2000" or "rate<0.01"
            if 'p(' in criteria:
                pct  = int(criteria.split('(')[1].split(')')[0])
                op   = '<' if '<' in criteria else '>'
                val  = float(criteria.split(op)[1])
                key  = f'p({pct})'
                actual = metric.get('values', {}).get(key, None)
            elif 'rate' in criteria:
                op  = '<' if '<' in criteria else '>'
                val = float(criteria.split(op)[1])
                actual = metric.get('values', {}).get('rate', None)
            elif 'avg' in criteria:
                op  = '<' if '<' in criteria else '>'
                val = float(criteria.split(op)[1])
                actual = metric.get('values', {}).get('avg', None)
            elif 'count' in criteria:
                op  = '<' if '<' in criteria else '>'
                val = float(criteria.split(op)[1])
                actual = metric.get('values', {}).get('count', None)
            else:
                continue

            if actual is None:
                violations.append(f"MISSING metric '{metric_name}' ({criteria})")
                continue

            ok = (actual < val) if op == '<' else (actual > val)
            label = f"{metric_name} {criteria} — actual: {actual:.4f}"
            if ok:
                passed.append(f"  PASS  {label}")
            else:
                violations.append(f"  FAIL  {label}")

    for p in passed:
        print(p)

    if violations:
        print("
THRESHOLD VIOLATIONS:")
        for v in violations:
            print(v)
        return 1
    else:
        print("
All thresholds passed!")
        return 0

if __name__ == '__main__':
    if len(sys.argv) < 2:
        print(f"Usage: {sys.argv[0]} <summary.json>")
        sys.exit(1)

    SLA = {
        'http_req_duration': ['p(95)<2000', 'p(99)<3000'],
        'http_req_failed':   ['rate<0.01'],
        'checks':            ['rate>0.99'],
    }
    sys.exit(validate(sys.argv[1], SLA))
```

**Usage:**
```bash
k6 run --summary-export=summary.json script.js
python validate_results.py summary.json
```

#### Exercise 9.4: Grafana Cloud k6 Integration

```bash
# Step 1: Get a free Grafana Cloud k6 token at grafana.com/products/cloud/k6

# Step 2: Authenticate
k6 cloud login --token YOUR_TOKEN_HERE

# Step 3: Run in cloud (distributed across multiple geographic regions)
k6 cloud lesson9_ci_script.js

# Step 4: Stream local results to cloud for dashboarding
k6 run --out cloud lesson9_ci_script.js
```

Benefits of Grafana Cloud k6:
- Distributed load generation from multiple geographic locations
- Real-time Grafana dashboards during test execution
- Historical comparison and trend analysis
- Team collaboration and test result sharing
- No local infrastructure required

### Control Questions

1. **What k6 exit code indicates that the test ran successfully but thresholds were breached?**
   <details>
   <summary>Answer</summary>
   Exit code 97. Exit code 0 means the test ran AND all thresholds passed. In CI pipelines, you should treat any non-zero exit code as a build failure.
   </details>

2. **How do you override the number of VUs defined in the script from the command line?**
   <details>
   <summary>Answer</summary>
   Use `--vus` flag: `k6 run --vus 100 script.js`. Note that this only works for the top-level `vus` option, not for scenarios. For scenarios, use environment variables read with `__ENV` inside the script.
   </details>

3. **Why should you run a smoke test before a full load test in CI?**
   <details>
   <summary>Answer</summary>
   A smoke test (1 VU, short duration) validates that the script, environment, and endpoints are working correctly with minimal resource usage. Catching configuration errors or endpoint failures early prevents wasting time and compute resources on a broken full load test.
   </details>

4. **What is `--summary-export` and why is it useful in CI?**
   <details>
   <summary>Answer</summary>
   `--summary-export=file.json` writes the final test summary (all metric statistics) to a JSON file. In CI, this allows post-processing: parsing metrics, comparing against baselines, generating reports, or feeding data to dashboards — all programmatically without screen-scraping console output.
   </details>

5. **What is the advantage of Grafana Cloud k6 over local k6 execution?**
   <details>
   <summary>Answer</summary>
   Cloud k6 distributes load generators across multiple geographic regions (simulating global users), eliminates local hardware limits, provides built-in Grafana dashboards for real-time and historical analysis, and enables team collaboration on test results without setting up local infrastructure.
   </details>

---

## Lesson 10: Best Practices, Test Design Patterns, and Performance Analysis

### Theoretical Part

#### k6 Script Design Principles

**1. Keep the default function focused**
- One scenario per script file, or use named exports with scenarios
- Avoid business logic in init; use setup() for data preparation

**2. Make scripts portable**
- Parameterize all environment-specific values via `__ENV`
- Never hardcode hostnames, ports, or credentials

**3. Thresholds define Done**
- Define thresholds before writing the test, not after
- Base thresholds on SLAs and acceptance criteria
- Use percentiles (p95, p99), not averages, for SLAs

**4. Check everything, threshold what matters**
- Add checks to every response to catch functional regressions
- Add thresholds to `checks` metric to gate on functional correctness
- Add thresholds to performance metrics to gate on performance

**5. Realistic load modeling**
- Use `sleep()` with realistic think times (not 0)
- Use `ramping-vus` or `ramping-arrival-rate` for gradual ramp-up
- Model realistic action mixes with multiple scenarios or probability branches

#### Common Performance Test Types Implemented in k6

| Test Type | Executor | VUs/Rate | Duration | Goal |
|---|---|---|---|---|
| Smoke | constant-vus | 1–2 VUs | 1–3 min | Verify script/endpoints work |
| Load | ramping-vus | up to expected peak | 15–30 min | Validate SLAs at expected load |
| Stress | ramping-vus | up to 2–3x expected | 30–60 min | Find breaking point |
| Spike | ramping-arrival-rate | sudden 10x increase | 5–15 min | Validate auto-scaling |
| Soak | constant-vus | moderate (50–60%) | 4–24 hours | Detect memory leaks |
| Breakpoint | ramping-vus (never stop) | continuously increasing | Until failure | Find maximum capacity |

#### Performance Analysis Checklist

After each test run, analyze:

- [ ] **Error rate**: Is it within acceptable limits?
- [ ] **p95/p99 response time**: Do they meet SLA thresholds?
- [ ] **Trend over time**: Does response time increase during the test? (indicates memory leak or saturation)
- [ ] **Check pass rate**: Are functional checks passing?
- [ ] **Throughput (RPS)**: Is the system processing expected volume?
- [ ] **Connection issues**: Are there `http_req_connecting` spikes? (TCP pool exhaustion)
- [ ] **TLS overhead**: Is `http_req_tls_handshaking` unusually high? (SSL offload issues)

### Practical Part

#### Exercise 10.1: Complete Test Suite — All Test Types for One API

Create `lesson10_test_suite.js`:

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';
import { Rate, Trend } from 'k6/metrics';

const slaCompliance = new Rate('sla_compliance');
const businessTx    = new Trend('business_tx_ms', true);

const TEST_TYPE = __ENV.TEST_TYPE || 'smoke';
const BASE      = __ENV.BASE_URL  || 'https://dummyjson.com';

// Shared thresholds
const THRESHOLDS = {
  'http_req_failed':  ['rate<0.01'],
  'checks':           ['rate>0.99'],
  'sla_compliance':   ['rate>0.95'],
  'business_tx_ms':   ['p(95)<3000'],
};

// Test profiles
const PROFILES = {
  smoke: {
    vus: 1,
    duration: '1m',
    thresholds: { ...THRESHOLDS, 'http_req_duration': ['p(95)<5000'] },
  },
  load: {
    stages: [
      { duration: '2m',  target: 50  },
      { duration: '10m', target: 50  },
      { duration: '2m',  target: 0   },
    ],
    thresholds: { ...THRESHOLDS, 'http_req_duration': ['p(95)<2000'] },
  },
  stress: {
    stages: [
      { duration: '2m',  target: 50  },
      { duration: '5m',  target: 100 },
      { duration: '5m',  target: 200 },
      { duration: '5m',  target: 300 },
      { duration: '2m',  target: 0   },
    ],
    thresholds: { ...THRESHOLDS, 'http_req_duration': ['p(95)<5000'] },
  },
  spike: {
    stages: [
      { duration: '1m',  target: 10  },
      { duration: '30s', target: 300 },  // spike
      { duration: '30s', target: 10  },  // recover
      { duration: '1m',  target: 10  },
    ],
    thresholds: { ...THRESHOLDS, 'http_req_duration': ['p(99)<10000'] },
  },
};

export const options = PROFILES[TEST_TYPE] || PROFILES.smoke;

export function setup() {
  console.log(`=== k6 Test Suite ===`);
  console.log(`Profile: ${TEST_TYPE} | Target: ${BASE}`);
  // Verify endpoint is reachable before starting
  const res = http.get(`${BASE}/products/1`);
  if (res.status !== 200) throw new Error(`Endpoint unreachable: ${res.status}`);
  return { verified: true };
}

export default function (data) {
  const start = Date.now();

  // Browse products (high frequency)
  const listRes = http.get(`${BASE}/products?limit=10`);
  check(listRes, {
    'list 200':         (r) => r.status === 200,
    'has products':     (r) => r.json('products').length > 0,
    'list SLA':         (r) => r.timings.duration < 2000,
  });
  slaCompliance.add(listRes.timings.duration < 2000);

  sleep(Math.random() * 1.5 + 0.5);

  // View single product (medium frequency)
  if (Math.random() < 0.7) {
    const id      = Math.floor(Math.random() * 100) + 1;
    const detRes  = http.get(`${BASE}/products/${id}`);
    check(detRes, { 'detail 200': (r) => r.status === 200 });
    slaCompliance.add(detRes.timings.duration < 1500);
  }

  sleep(Math.random() * 1 + 0.5);

  // Search (lower frequency)
  if (Math.random() < 0.4) {
    const q   = ['phone', 'laptop', 'cream', 'watch'][Math.floor(Math.random() * 4)];
    const res = http.get(`${BASE}/products/search?q=${q}`);
    check(res, { 'search 200': (r) => r.status === 200 });
  }

  businessTx.add(Date.now() - start);

  sleep(Math.random() + 0.5);
}

export function teardown(data) {
  console.log('Test complete. Review thresholds in summary.');
}
```

**Run commands:**
```bash
k6 run -e TEST_TYPE=smoke  lesson10_test_suite.js
k6 run -e TEST_TYPE=load   lesson10_test_suite.js
k6 run -e TEST_TYPE=stress lesson10_test_suite.js
k6 run -e TEST_TYPE=spike  lesson10_test_suite.js
```

#### Exercise 10.2: Performance Baseline and Regression Detection

Create `lesson10_baseline.js` — a reusable baseline test:

```javascript
import http from 'k6/http';
import { check, sleep } from 'k6';

export const options = {
  // Fixed, reproducible profile for baseline comparisons
  scenarios: {
    baseline: {
      executor:  'constant-vus',
      vus:       20,
      duration:  '5m',
    },
  },
  thresholds: {
    'http_req_duration': ['p(95)<2000', 'p(99)<3000'],
    'http_req_failed':   ['rate<0.01'],
    'checks':            ['rate>0.99'],
  },
  summaryTrendStats: ['avg', 'min', 'med', 'max', 'p(90)', 'p(95)', 'p(99)', 'count'],
};

const ENDPOINTS = [
  { url: 'https://dummyjson.com/products?limit=10',   weight: 50, name: 'list'   },
  { url: 'https://dummyjson.com/products/1',           weight: 30, name: 'detail' },
  { url: 'https://dummyjson.com/products/search?q=ph', weight: 20, name: 'search' },
];

function weightedChoice() {
  const r = Math.random() * 100;
  let cum = 0;
  for (const ep of ENDPOINTS) {
    cum += ep.weight;
    if (r <= cum) return ep;
  }
  return ENDPOINTS[0];
}

export default function () {
  const ep  = weightedChoice();
  const res = http.get(ep.url, { tags: { endpoint: ep.name } });
  check(res, { [`${ep.name} 200`]: (r) => r.status === 200 });
  sleep(Math.random() * 2 + 1);
}
```

**Baseline workflow:**
```bash
# Run baseline, capture summary
k6 run --summary-export=baseline_v1.json lesson10_baseline.js

# After code changes, run again
k6 run --summary-export=baseline_v2.json lesson10_baseline.js

# Compare (using Python)
python compare_baselines.py baseline_v1.json baseline_v2.json
```

Create `compare_baselines.py`:

```python
#!/usr/bin/env python3
# Compare two k6 summary-export JSON files for regression detection.
import json, sys

REGRESSION_THRESHOLD = 0.10  # 10% degradation triggers warning

def load(path):
    with open(path) as f:
        return json.load(f)['metrics']

def compare(a_metrics, b_metrics, metric, stat):
    a_val = a_metrics.get(metric, {}).get('values', {}).get(stat)
    b_val = b_metrics.get(metric, {}).get('values', {}).get(stat)
    if a_val is None or b_val is None:
        return None, None, None
    delta = (b_val - a_val) / a_val if a_val != 0 else 0
    return a_val, b_val, delta

if __name__ == '__main__':
    if len(sys.argv) != 3:
        print(f"Usage: {sys.argv[0]} baseline.json new.json")
        sys.exit(1)

    a = load(sys.argv[1])
    b = load(sys.argv[2])

    checks = [
        ('http_req_duration', 'p(95)'),
        ('http_req_duration', 'avg'),
        ('http_req_failed',   'rate'),
        ('checks',            'rate'),
        ('http_reqs',         'rate'),
    ]

    regressions = 0
    print(f"{'Metric':<35} {'Baseline':>12} {'New':>12} {'Change':>10}  Status")
    print('-' * 80)
    for metric, stat in checks:
        a_val, b_val, delta = compare(a, b, metric, stat)
        if delta is None:
            print(f"  {metric} ({stat}): N/A")
            continue
        is_error_metric = 'failed' in metric or 'error' in metric
        regressed = delta > REGRESSION_THRESHOLD if not is_error_metric else delta > 0
        icon = 'REGRESS' if regressed else 'OK'
        if regressed:
            regressions += 1
        print(f"  {metric} ({stat}): {a_val:>10.3f}  →  {b_val:>10.3f}  ({delta*100:+.1f}%)  {icon}")

    print()
    if regressions:
        print(f"WARNING: {regressions} regression(s) detected!")
        sys.exit(1)
    else:
        print("No regressions detected.")
        sys.exit(0)
```

#### Exercise 10.3: Grafana Dashboard Setup (Local InfluxDB)

```bash
# 1. Start InfluxDB and Grafana with Docker Compose
cat > docker-compose.yml << 'EOF'
version: "3"
services:
  influxdb:
    image: influxdb:1.8
    ports: ["8086:8086"]
    environment:
      INFLUXDB_DB: k6
  grafana:
    image: grafana/grafana:latest
    ports: ["3000:3000"]
    environment:
      GF_AUTH_ANONYMOUS_ENABLED: "true"
      GF_AUTH_ANONYMOUS_ORG_ROLE: Admin
EOF
docker compose up -d

# 2. Run k6 outputting to InfluxDB
k6 run --out influxdb=http://localhost:8086/k6 lesson10_test_suite.js

# 3. Open Grafana at http://localhost:3000
#    Add InfluxDB data source: http://influxdb:8086, database: k6
#    Import dashboard ID 2587 (official k6 Grafana dashboard)
```

### Control Questions

1. **What is the difference between a load test and a stress test in terms of k6 configuration?**
   <details>
   <summary>Answer</summary>
   A load test uses a ramping-vus profile that reaches and holds expected production load to validate SLAs under normal conditions. A stress test continuously increases VUs beyond expected load (2–3x or more) to find the breaking point, saturation threshold, and failure mode. Stress tests often use more aggressive ramp stages and looser thresholds since failure is expected at some point.
   </details>

2. **Why should thresholds be defined based on percentiles (p95, p99) rather than average response time?**
   <details>
   <summary>Answer</summary>
   Averages are skewed by outliers — a few very fast responses can mask many slow ones, and vice versa. Percentiles represent the actual user experience: p95 < 2000ms means 95% of users wait less than 2 seconds. This aligns with SLAs that describe user experience, not aggregate statistics.
   </details>

3. **What does it mean when `http_req_connecting` increases significantly during a load test?**
   <details>
   <summary>Answer</summary>
   It indicates TCP connection pool exhaustion — k6 (or the server) cannot reuse existing connections and must establish new TCP handshakes. This could mean: the server's connection limit is reached, connection timeouts are occurring, or keep-alive is not working. It signals server-side connection management issues.
   </details>

4. **How would you detect a memory leak in a soak test using k6?**
   <details>
   <summary>Answer</summary>
   Monitor response time (`http_req_duration`) trends over the soak test duration. A memory leak typically manifests as: gradually increasing response times under constant load, increasing garbage collection pauses, eventual OOM errors causing 500 responses. Plot p95 response time over time — a steadily rising trend under stable load indicates memory or resource exhaustion.
   </details>

5. **What is the recommended workflow for integrating k6 into a CI/CD pipeline?**
   <details>
   <summary>Answer</summary>
   1) Run a smoke test (1 VU) on every PR to validate the script and environment. 2) Run a load test on merges to main with realistic VUs and SLA thresholds. 3) Export `--summary-export` JSON for programmatic threshold checking. 4) Use k6's non-zero exit code to fail the pipeline on threshold breach. 5) Archive results as artifacts and compare against baseline for regression detection. 6) Optionally stream results to InfluxDB/Grafana for historical trending.
   </details>

---

## Appendix A: k6 CLI Quick Reference

```bash
# Installation
choco install k6                           # Windows
brew install k6                            # macOS
sudo apt-get install k6                    # Ubuntu/Debian

# Running tests
k6 run script.js                           # basic run
k6 run --vus 50 --duration 5m script.js   # override VUs and duration
k6 run --stage 1m:10 --stage 5m:50 --stage 1m:0 script.js  # stages
k6 run -e MY_VAR=value script.js           # environment variable
k6 run --out json=out.json script.js       # JSON output
k6 run --out csv=out.csv script.js         # CSV output
k6 run --out influxdb=http://host:8086/db  # InfluxDB output
k6 run --summary-export=summary.json       # export final summary
k6 run --no-summary script.js             # suppress summary output
k6 run -q script.js                       # quiet mode

# Cloud execution
k6 cloud login --token TOKEN
k6 cloud script.js
k6 run --out cloud script.js              # local run, cloud results

# Inspect
k6 inspect script.js                      # show script options without running
k6 version                                # show k6 version
```

---

## Appendix B: Built-in Metrics Reference

| Metric | Type | Unit | Description |
|---|---|---|---|
| `http_reqs` | Counter | count | Total HTTP requests made |
| `http_req_duration` | Trend | ms | Total request round-trip time |
| `http_req_failed` | Rate | 0–1 | Fraction of failed requests (non-2xx/3xx) |
| `http_req_blocked` | Trend | ms | Time blocked waiting for TCP connection slot |
| `http_req_connecting` | Trend | ms | TCP connection establishment time |
| `http_req_tls_handshaking` | Trend | ms | TLS handshake time |
| `http_req_sending` | Trend | ms | Time to send request |
| `http_req_waiting` | Trend | ms | TTFB — time to first byte |
| `http_req_receiving` | Trend | ms | Time to download response body |
| `vus` | Gauge | count | Current active virtual users |
| `vus_max` | Gauge | count | Maximum VUs allocated |
| `iterations` | Counter | count | Total completed iterations |
| `iteration_duration` | Trend | ms | Time for one full iteration |
| `checks` | Rate | 0–1 | Fraction of checks that passed |
| `data_sent` | Counter | bytes | Total bytes sent |
| `data_received` | Counter | bytes | Total bytes received |
| `group_duration` | Trend | ms | Time spent inside a group block |

---

## Appendix C: Executor Quick Reference

| Executor | Key params | Best for |
|---|---|---|
| `constant-vus` | `vus`, `duration` | Baseline, simple load |
| `ramping-vus` | `stages`, `startVUs` | Ramp-up/down patterns |
| `constant-arrival-rate` | `rate`, `timeUnit`, `duration` | Fixed RPS target |
| `ramping-arrival-rate` | `stages`, `startRate`, `timeUnit` | Variable RPS, spike |
| `per-vu-iterations` | `vus`, `iterations` | Fixed work per VU |
| `shared-iterations` | `vus`, `iterations` | Fixed total work pool |
| `externally-controlled` | — | Dynamic/manual control |

---

## Appendix D: Common k6 Code Patterns

### Pattern 1: Robust Request with Error Logging

```javascript
function safeGet(url, params = {}) {
  const res = http.get(url, params);
  if (res.status === 0) {
    console.error(`Network error on ${url}: ${res.error}`);
  } else if (res.status >= 400) {
    console.warn(`HTTP ${res.status} on ${url}: ${res.body.substring(0, 200)}`);
  }
  return res;
}
```

### Pattern 2: Weighted Random Choice

```javascript
const actions = [
  { name: 'browse',   weight: 60, fn: browseProducts },
  { name: 'search',   weight: 25, fn: searchProducts },
  { name: 'checkout', weight: 15, fn: checkout       },
];

function weightedRandom() {
  const r = Math.random() * 100;
  let cum = 0;
  for (const a of actions) {
    cum += a.weight;
    if (r <= cum) return a.fn;
  }
  return actions[0].fn;
}

export default function () {
  weightedRandom()();
  sleep(1);
}
```

### Pattern 3: Retry on Failure

```javascript
function requestWithRetry(url, retries = 3) {
  for (let i = 0; i < retries; i++) {
    const res = http.get(url);
    if (res.status === 200) return res;
    console.warn(`Retry ${i + 1}/${retries} for ${url} — got ${res.status}`);
    sleep(Math.pow(2, i)); // exponential backoff: 1s, 2s, 4s
  }
  return null;
}
```

### Pattern 4: Pagination Loop

```javascript
function fetchAllPages(baseUrl, pageSize = 10) {
  const allItems = [];
  let   skip     = 0;

  while (true) {
    const res   = http.get(`${baseUrl}?limit=${pageSize}&skip=${skip}`);
    const items = res.json('products');
    if (!items || items.length === 0) break;
    allItems.push(...items);
    skip += pageSize;
    if (items.length < pageSize) break;
    sleep(0.1);
  }
  return allItems;
}
```

### Pattern 5: Conditional Scenario Branch

```javascript
export default function () {
  // 70% browse, 20% search, 10% purchase
  const roll = Math.random() * 100;
  if (roll < 70) {
    browse();
  } else if (roll < 90) {
    search();
  } else {
    purchase();
  }
}
```

---

## Appendix E: Performance Testing Anti-Patterns to Avoid

| Anti-Pattern | Problem | Fix |
|---|---|---|
| No `sleep()` in default() | Unrealistic load, server flooding | Add think time (1–5s) |
| Hardcoded hostnames | Scripts fail when environment changes | Use `__ENV.BASE_URL` |
| No thresholds defined | Tests never fail, regressions undetected | Define SLA thresholds |
| Testing in production | Risk of outage, skewed real traffic | Use staging environment |
| Only checking status 200 | Functional bugs missed | Check field values, not just status |
| Running with GUI tools | Resource overhead skews results | k6 is CLI-only by design |
| Ignoring think time | 100% CPU load test, not real traffic | Model realistic pacing |
| Assertions that throw | Test aborts mid-run | Use `check()`, not `throw` |
| No baseline comparison | No way to detect regressions | Store and compare summaries |
| Too many `console.log()` | Massive I/O overhead, slow test | Use sparingly or remove for load tests |

---

## Final Assessment

### Capstone Project: Complete Performance Test Suite

Design and implement a full performance test suite for the DummyJSON API (`https://dummyjson.com`):

**Requirements:**

1. **Smoke Test** (1 VU, 1 min) — verify all endpoints return 200
2. **Load Test** (50 VUs, 10 min with ramp) — validate SLAs under normal load
3. **Stress Test** (ramp to 200 VUs) — find the saturation point
4. **Spike Test** (sudden 10x load) — verify recovery behavior

**Technical Requirements:**
- Use `SharedArray` for user data (at least 5 users from dummyjson)
- Implement authentication flow with token reuse
- Use at least 3 named groups for business transaction timing
- Define `tags` for per-endpoint threshold targeting
- Create at least 2 custom metrics (one `Rate`, one `Trend`)
- Define thresholds based on SLAs: p95 < 2s, error rate < 1%, check rate > 99%
- Export results with `--summary-export` for automated validation
- Integrate smoke test into a GitHub Actions workflow

**User Journeys to Implement:**
- Browse products (40%): list → view detail → search
- User profile (30%): login → view profile → view cart
- Write operations (30%): login → add product to cart → update

**Deliverables:**
- `suite/smoke.js` — smoke test script
- `suite/load.js` — load test script
- `suite/stress.js` — stress test script
- `suite/spike.js` — spike test script
- `suite/shared.js` — shared utilities, data, and configuration
- `test-data/users.json` — test user data
- `.github/workflows/k6.yml` — CI pipeline
- `validate.py` — threshold validation script
- `RESULTS.md` — analysis document with findings and recommendations

**Evaluation Criteria:**
- Script modularity and reusability
- Realistic workload modeling with proper think times
- Complete threshold coverage (performance + functional)
- Proper correlation and session management
- CI/CD integration quality
- Results analysis and actionable recommendations

---

## Course Summary

### Key Takeaways

1. **k6 is code-first** — JavaScript scripts make tests versionable, reviewable, and maintainable
2. **Checks vs Thresholds** — checks are inline assertions; thresholds are the test pass/fail gate
3. **Executors define load shape** — choose the right executor for each test type
4. **Parameterize everything** — use `__ENV`, `SharedArray`, and `open()` for data; never hardcode
5. **Correlation is essential** — always extract and reuse dynamic tokens and IDs
6. **Custom metrics** — extend k6's built-in metrics for business-level SLA tracking
7. **CI/CD integration** — k6's exit codes and summary export make automated gating trivial
8. **Realistic modeling** — think time, weighted actions, and staged ramp-ups produce valid results

### Next Steps

**Explore k6 Extensions:**
- `xk6-sql` — database load testing
- `xk6-kafka` — Kafka load testing
- `xk6-browser` — browser-based testing with Core Web Vitals

**Community Resources:**
- Official docs: [https://k6.io/docs](https://k6.io/docs)
- k6 Community Forum: [https://community.k6.io](https://community.k6.io)
- Grafana Cloud k6: [https://grafana.com/products/cloud/k6](https://grafana.com/products/cloud/k6)
- k6 GitHub: [https://github.com/grafana/k6](https://github.com/grafana/k6)
- Awesome k6: [https://github.com/grafana/awesome-k6](https://github.com/grafana/awesome-k6)

---

**Course Complete! You are now equipped to design, implement, and automate professional performance tests with k6.**

Good luck with your performance engineering journey!
