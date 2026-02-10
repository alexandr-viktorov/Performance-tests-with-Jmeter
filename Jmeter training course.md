# JMeter Load Testing Training Course
## Complete Guide for Performance Testing Engineers

---

## Course Overview
This comprehensive training course covers Apache JMeter from fundamentals to advanced performance testing techniques. Each lesson includes theory, hands-on practice, and assessment questions.

**Prerequisites:** Basic understanding of HTTP, web applications, and testing concepts  
**Duration:** 8-10 weeks (self-paced)  
**Tools Required:** JMeter 5.5+, Java JDK 8+, text editor

---

## Lesson 1: Introduction to Performance Testing & JMeter Basics

### Theoretical Part

#### What is Performance Testing?
Performance testing evaluates how a system performs under specific workload conditions. Key types include:

- **Load Testing**: Testing system behavior under expected load
- **Stress Testing**: Testing beyond normal operational capacity
- **Spike Testing**: Testing sudden increases in load
- **Soak Testing**: Testing system stability over extended periods
- **Scalability Testing**: Determining system's ability to scale up

#### Key Performance Metrics
- **Response Time**: Time taken to receive response after sending request
- **Throughput**: Number of requests processed per unit time (requests/sec)
- **Error Rate**: Percentage of failed requests
- **Concurrent Users**: Number of simultaneous active users
- **Resource Utilization**: CPU, memory, disk, network usage

#### What is Apache JMeter?
- Open-source Java application for load testing and performance measurement
- Originally designed for web applications, now supports multiple protocols
- Can simulate heavy loads on servers, networks, or objects

#### JMeter Architecture
- **Test Plan**: Container for all test elements
- **Thread Groups**: Simulate users/load
- **Samplers**: Generate requests to servers
- **Listeners**: Collect and display test results
- **Timers**: Add delays between requests
- **Assertions**: Validate responses
- **Configuration Elements**: Set up defaults and variables

### Practical Part

#### Exercise 1.1: Install and Launch JMeter

```bash
# Download JMeter from https://jmeter.apache.org/download_jmeter.cgi
# Extract the archive
# Windows: Run bin/jmeter.bat
# Mac/Linux: Run bin/jmeter.sh
```

#### Exercise 1.2: Explore JMeter Interface

1. Launch JMeter
2. Identify key interface elements:
   - Menu bar
   - Toolbar
   - Test Plan tree (left panel)
   - Configuration panel (right panel)
3. Create a new Test Plan
4. Save it as "FirstTest.jmx"

#### Exercise 1.3: Create Your First Test

**Objective**: Send a simple HTTP request to a public API

1. Add Thread Group: Right-click Test Plan → Add → Threads → Thread Group
2. Configure Thread Group:
   - Number of Threads: 10
   - Ramp-up period: 5
   - Loop Count: 1
3. Add HTTP Sampler: Right-click Thread Group → Add → Sampler → HTTP Request
4. Configure HTTP Request:
   - Server Name: `httpbin.org`
   - Path: `/get`
5. Add Listener: Right-click Thread Group → Add → Listener → View Results Tree
6. Save and Run the test (Ctrl+R or click green Start button)
7. Observe results in View Results Tree

### Control Questions

1. **What are the three main differences between load testing and stress testing?**
   <details>
   <summary>Answer</summary>
   - Load testing tests under expected/normal conditions; stress testing tests beyond normal capacity
   - Load testing validates performance against requirements; stress testing finds breaking points
   - Load testing focuses on stability; stress testing focuses on recovery and error handling
   </details>

2. **What happens if you set Thread Group with 100 threads and ramp-up period of 50 seconds?**
   <details>
   <summary>Answer</summary>
   JMeter will start 2 threads per second (100/50), gradually reaching 100 concurrent threads after 50 seconds
   </details>

3. **Name five types of Samplers available in JMeter.**
   <details>
   <summary>Answer</summary>
   HTTP Request, FTP Request, JDBC Request, JMS Sampler, SMTP Sampler, TCP Sampler, Java Request, etc.
   </details>

4. **Why is it important to add Listeners to your test plan?**
   <details>
   <summary>Answer</summary>
   Listeners collect and visualize test results, enabling analysis of response times, throughput, errors, and other performance metrics
   </details>

5. **What protocol does JMeter use to communicate with web servers in HTTP Request samplers?**
   <details>
   <summary>Answer</summary>
   HTTP/HTTPS protocols
   </details>

---

## Lesson 2: Thread Groups and Test Execution Control

### Theoretical Part

#### Understanding Thread Groups

Thread Groups are the foundation of any JMeter test plan. They represent virtual users executing your test scenario.

**Key Parameters:**
- **Number of Threads (users)**: Total virtual users to simulate
- **Ramp-up Period**: Time to start all threads (gradual load increase)
- **Loop Count**: Number of times each thread executes the test
- **Infinite Loop**: Run until manually stopped
- **Delayed Start**: Schedule test start time

**Thread Group Types:**
1. **Thread Group**: Standard thread group with basic configuration
2. **setUp Thread Group**: Runs before regular thread groups (for test preparation)
3. **tearDown Thread Group**: Runs after all thread groups complete (for cleanup)
4. **Ultimate Thread Group (plugin)**: Advanced scheduling with multiple load stages
5. **Stepping Thread Group (plugin)**: Gradual increase of load with precise control

#### Calculating Load Patterns

**Formula for Ramp-up:**
```
Threads per second = Number of Threads / Ramp-up Period
Start delay per thread = Ramp-up Period / Number of Threads
```

**Example:**
- 300 threads, 60 second ramp-up = 5 threads/second
- Each thread starts 0.2 seconds after the previous

#### Thread Lifecycle

1. Thread starts
2. Execute samplers in sequence
3. If loop count > 1, repeat from step 2
4. Thread terminates

### Practical Part

#### Exercise 2.1: Comparing Different Ramp-up Patterns

Create three scenarios testing the same endpoint with different ramp-ups:

**Scenario A: Aggressive Ramp-up**
- Threads: 100
- Ramp-up: 10 seconds
- Loop Count: 5

**Scenario B: Moderate Ramp-up**
- Threads: 100
- Ramp-up: 50 seconds
- Loop Count: 5

**Scenario C: Gradual Ramp-up**
- Threads: 100
- Ramp-up: 100 seconds
- Loop Count: 5

1. Create three Thread Groups with configurations above
2. Add HTTP Request to each: `httpbin.org/delay/1`
3. Add "Aggregate Report" listener to Test Plan
4. Run each scenario separately
5. Compare average response times and throughput

#### Exercise 2.2: Using setUp and tearDown Thread Groups

**Objective**: Create a test that sets up test data before main test

1. Add setUp Thread Group:
   - Threads: 1
   - Add HTTP Request: POST to `httpbin.org/post` with test data
2. Add regular Thread Group:
   - Threads: 50
   - Ramp-up: 10
   - Add HTTP Request: GET `httpbin.org/get`
3. Add tearDown Thread Group:
   - Threads: 1
   - Add HTTP Request: DELETE operation
4. Run and observe execution order

#### Exercise 2.3: Duration-Based Testing

1. Create Thread Group with:
   - Threads: 50
   - Ramp-up: 10
   - Loop Count: Forever (check Infinite)
   - Duration: 300 seconds (Scheduler checkbox)
2. Add HTTP Request: `httpbin.org/delay/2`
3. Add "Summary Report" listener
4. Run for exactly 5 minutes
5. Calculate total requests sent

### Control Questions

1. **If you configure a Thread Group with 200 threads, 100-second ramp-up, and loop count 3, how many total requests will be sent if each thread has only one HTTP sampler?**
   <details>
   <summary>Answer</summary>
   600 requests (200 threads × 3 loops × 1 sampler)
   </details>

2. **What's the difference between setting "Loop Count = 10" vs "Duration (seconds) = 600"?**
   <details>
   <summary>Answer</summary>
   Loop Count executes exactly 10 iterations regardless of time; Duration runs as many iterations as possible within 600 seconds (time-bound vs iteration-bound)
   </details>

3. **When would you use a setUp Thread Group instead of a regular Thread Group?**
   <details>
   <summary>Answer</summary>
   For initialization tasks like creating test data, authentication, database setup, or any prerequisite operations needed before the actual load test
   </details>

4. **Calculate: 500 threads with 250-second ramp-up. What's the thread start rate?**
   <details>
   <summary>Answer</summary>
   2 threads per second (500/250 = 2)
   </details>

5. **Why is gradual ramp-up important in load testing?**
   <details>
   <summary>Answer</summary>
   Gradual ramp-up simulates realistic user behavior, avoids overwhelming the system immediately, helps identify at what load level performance degrades, and allows system resources (caches, connections) to warm up
   </details>

---

## Lesson 3: HTTP Samplers and Web Testing Fundamentals

### Theoretical Part

#### HTTP Request Sampler Components

**Basic Configuration:**
- **Protocol**: http or https
- **Server Name or IP**: Target server address
- **Port Number**: (default 80 for HTTP, 443 for HTTPS)
- **Method**: GET, POST, PUT, DELETE, PATCH, HEAD, OPTIONS
- **Path**: Endpoint/resource path
- **Content Encoding**: UTF-8, ISO-8859-1, etc.

**Parameters Tab:**
- Send parameters as name-value pairs
- Automatically URL-encoded for GET
- Sent in body for POST

**Body Data Tab:**
- Raw request body (JSON, XML, etc.)
- Used for POST/PUT requests with complex payloads

**Files Upload Tab:**
- Upload files with multipart/form-data

**Advanced Configuration:**
- Timeouts (connect, response)
- Follow redirects
- Use Keep-Alive
- Client implementation (Java, Apache HttpClient)

#### HTTP Header Manager

Used to add/override HTTP headers:
- Content-Type (application/json, application/xml, text/html)
- Authorization (Bearer tokens, Basic Auth)
- Accept
- User-Agent
- Custom headers

#### HTTP Cookie Manager

Manages cookies like a browser:
- Stores cookies from responses
- Sends cookies with subsequent requests
- Can manually define cookies
- Cookie Policy: compatibility, standard, netscape

#### HTTP Request Defaults

Sets default values for all HTTP Requests in scope:
- Avoids repeating server name, port, protocol
- Applied to all HTTP samplers in scope
- Can be overridden by individual samplers

### Practical Part

#### Exercise 3.1: Testing Different HTTP Methods

Create tests for RESTful API operations on JSONPlaceholder:

1. **GET Request**
   - Server: `jsonplaceholder.typicode.com`
   - Path: `/posts/1`
   - Method: GET

2. **POST Request**
   - Server: `jsonplaceholder.typicode.com`
   - Path: `/posts`
   - Method: POST
   - Body Data:
   ```json
   {
     "title": "JMeter Test",
     "body": "Testing POST request",
     "userId": 1
   }
   ```
   - Add HTTP Header Manager:
     - Name: `Content-Type`, Value: `application/json`

3. **PUT Request**
   - Path: `/posts/1`
   - Method: PUT
   - Update post content

4. **DELETE Request**
   - Path: `/posts/1`
   - Method: DELETE

5. Add "View Results Tree" to verify responses

#### Exercise 3.2: Working with Request Parameters

Test search functionality with query parameters:

1. Create HTTP Request:
   - Server: `jsonplaceholder.typicode.com`
   - Path: `/posts`
   - Method: GET
   - Parameters:
     - userId: 1

2. Create second request with multiple parameters:
   - Path: `/comments`
   - Parameters:
     - postId: 1
     - email: contains text

3. Compare URL construction in View Results Tree

#### Exercise 3.3: Using HTTP Request Defaults

Optimize test plan with defaults:

1. Add "HTTP Request Defaults":
   - Server: `jsonplaceholder.typicode.com`
   - Protocol: https

2. Create multiple HTTP Requests without repeating server:
   - Request 1: Path `/posts`
   - Request 2: Path `/users`
   - Request 3: Path `/albums`

3. Observe how defaults are applied

#### Exercise 3.4: Authentication Testing

Test API with authentication header:

1. Create HTTP Request:
   - Server: `httpbin.org`
   - Path: `/bearer`
   - Method: GET

2. Add HTTP Header Manager:
   - Name: `Authorization`
   - Value: `Bearer your-test-token-here`

3. Run and verify in response that token was sent

#### Exercise 3.5: Cookie Handling

Test cookie persistence:

1. Add HTTP Cookie Manager to Thread Group

2. Request 1 - Set Cookie:
   - Server: `httpbin.org`
   - Path: `/cookies/set?sessionid=12345`

3. Request 2 - Read Cookie:
   - Server: `httpbin.org`
   - Path: `/cookies`

4. Use View Results Tree to verify cookie was stored and sent

### Control Questions

1. **What's the difference between sending parameters via "Parameters" tab vs "Body Data" tab?**
   <details>
   <summary>Answer</summary>
   Parameters tab: sent as URL query string (GET) or form-encoded body (POST); Body Data: sends raw body content (JSON, XML) useful for APIs requiring specific content types
   </details>

2. **When should you add an HTTP Header Manager to your test plan?**
   <details>
   <summary>Answer</summary>
   When you need to send custom headers (Content-Type, Authorization, Accept, User-Agent), simulate different clients, or add authentication tokens
   </details>

3. **Why use HTTP Request Defaults?**
   <details>
   <summary>Answer</summary>
   To avoid repetition, maintain consistency, make test plans easier to maintain, and quickly change server for all requests (e.g., switching between test and production environments)
   </details>

4. **What HTTP status code indicates a successful POST request that creates a resource?**
   <details>
   <summary>Answer</summary>
   201 (Created)
   </details>

5. **How does HTTP Cookie Manager help simulate real user sessions?**
   <details>
   <summary>Answer</summary>
   It stores cookies from server responses and automatically sends them with subsequent requests, maintaining session state like a real browser, enabling authentication and session tracking
   </details>

---

## Lesson 4: Variables, Parameterization, and CSV Data Sets

### Theoretical Part

#### JMeter Variables and Properties

**Variables:**
- Scoped to thread (each thread has own copy)
- Created by configuration elements, preprocessors, extractors
- Syntax: `${variableName}`
- Can be used in any sampler field
- Not shared between threads

**Properties:**
- Global across JMeter instance
- Syntax: `${__P(propertyName)}`
- Can be set via command line: `jmeter -Jproperty=value`
- Shared between threads

**User Defined Variables:**
- Configuration element for static variables
- Evaluated once at test start
- Good for constants (server URLs, API keys)

#### Parameterization Techniques

**Why Parameterize?**
- Simulate realistic data variety
- Test with different user accounts
- Avoid caching issues
- Test data validation
- Create unique transactions

**Methods:**
1. User Defined Variables
2. CSV Data Set Config
3. Random Variable
4. Counter
5. Functions (__Random, __UUID, __time)

#### CSV Data Set Config

Reads data from CSV file for parameterization:

**Configuration:**
- **Filename**: Path to CSV file
- **Variable Names**: Column names (comma-separated)
- **Delimiter**: Usually comma (,) or semicolon (;)
- **Recycle on EOF**: Return to beginning when file ends
- **Stop thread on EOF**: Stop when data exhausted
- **Sharing mode**: 
  - All threads: Shared across all threads
  - Current thread group: Shared in group
  - Current thread: Each thread has own pointer

### Practical Part

#### Exercise 4.1: User Defined Variables

1. Add "User Defined Variables" config element

2. Define variables:
   - `BASE_URL`: jsonplaceholder.typicode.com
   - `API_VERSION`: v1
   - `USER_ID`: 1

3. Create HTTP Request using variables:
   - Server: `${BASE_URL}`
   - Path: `/${API_VERSION}/posts`
   - Parameters: userId = `${USER_ID}`

4. Run and verify in View Results Tree

#### Exercise 4.2: Creating and Using CSV Data File

**Step 1: Create CSV File**

Create `users.csv` in JMeter bin directory:
```csv
username,password,email
user1,pass123,user1@example.com
user2,pass456,user2@example.com
user3,pass789,user3@example.com
user4,pass012,user4@example.com
user5,pass345,user5@example.com
```

**Step 2: Configure CSV Data Set**

1. Add "CSV Data Set Config"
   - Filename: `users.csv`
   - Variable Names: `username,password,email`
   - Delimiter: `,`
   - Recycle on EOF: True
   - Stop thread on EOF: False
   - Sharing mode: All threads

**Step 3: Use Variables**

1. Create HTTP POST Request:
   - Server: `httpbin.org`
   - Path: `/post`
   - Body Data:
   ```json
   {
     "username": "${username}",
     "password": "${password}",
     "email": "${email}"
   }
   ```

2. Set Thread Group: 10 threads, 1 loop

3. Add "View Results Tree"

4. Run and observe different users in each request

#### Exercise 4.3: Random Data Generation

Create test with random data:

1. Add "Random Variable" config element:
   - Variable Name: `randomNumber`
   - Minimum: 1
   - Maximum: 100

2. Add HTTP Request:
   - Server: `jsonplaceholder.typicode.com`
   - Path: `/posts/${randomNumber}`

3. Run with 20 threads to see random post IDs

#### Exercise 4.4: Counter Implementation

Implement unique transaction IDs:

1. Add "Counter" config element:
   - Start: 1000
   - Increment: 1
   - Maximum: 9999
   - Variable Name: `transactionID`

2. Add HTTP Request with header:
   - Add HTTP Header Manager
   - Name: `X-Transaction-ID`
   - Value: `TXN-${transactionID}`

3. Run and verify unique IDs in View Results Tree

#### Exercise 4.5: Advanced CSV Scenario - Realistic Load Test

Create realistic scenario with multiple data types:

**Create `test_data.csv`:**
```csv
userId,productId,quantity,couponCode
1,101,2,SAVE10
2,102,1,SAVE20
3,103,5,SAVE10
1,104,1,NODISCOUNT
4,105,3,SAVE20
```

1. Configure CSV Data Set with file

2. Create HTTP POST Request simulating purchase:
```json
{
  "userId": "${userId}",
  "items": [{
    "productId": "${productId}",
    "quantity": ${quantity}
  }],
  "coupon": "${couponCode}",
  "timestamp": "${__time(yyyy-MM-dd HH:mm:ss)}"
}
```

3. Set Thread Group: 100 threads, 10 loops

4. Add "Summary Report" listener

5. Analyze data variety in results

### Control Questions

1. **What's the difference between ${variable} and ${__P(property)}?**
   <details>
   <summary>Answer</summary>
   Variables are thread-scoped (each thread has own copy); Properties are global across entire JMeter instance and shared between all threads
   </details>

2. **When should you use "Stop thread on EOF: True" in CSV Data Set Config?**
   <details>
   <summary>Answer</summary>
   When you want each thread to process unique data and stop when data is exhausted, useful for tests where data reuse is not acceptable (e.g., testing user registration)
   </details>

3. **If you have 10 rows in CSV and 100 threads with "Recycle on EOF: True", what happens?**
   <details>
   <summary>Answer</summary>
   The CSV data will be reused 10 times; data cycles back to beginning when file end is reached, so same data is used multiple times
   </details>

4. **How would you generate a unique ID for each request using JMeter functions?**
   <details>
   <summary>Answer</summary>
   Use ${__UUID()} function or combination of ${__time()} and ${__threadNum} and ${__counter()}
   </details>

5. **What are three benefits of using CSV parameterization in load testing?**
   <details>
   <summary>Answer</summary>
   1) Tests with realistic, varied data; 2) Avoids server-side caching issues; 3) Allows testing different user scenarios/permissions; 4) Enables data-driven testing
   </details>

---

## Lesson 5: Assertions and Response Validation

### Theoretical Part

#### What are Assertions?

Assertions validate that server responses meet expected criteria. They determine test pass/fail status.

**Key Concepts:**
- Assertions check response data, headers, codes, or duration
- Failed assertion marks sample as failed
- Multiple assertions can be added to one sampler
- Can be added at different scopes (sampler, thread group, test plan)

#### Types of Assertions

**1. Response Assertion**
- Most versatile assertion type
- Can check response code, message, headers, body, URL
- Supports patterns: contains, matches, equals, substring
- Supports regex patterns
- Can test "NOT" conditions

**2. JSON Assertion**
- Validates JSON responses using JSONPath
- Checks specific values in JSON structure
- Supports expected values and regex
- Can validate data types

**3. XPath Assertion (XML)**
- Uses XPath expressions for XML validation
- Validates XML structure and values
- Supports XML namespaces

**4. Size Assertion**
- Validates response size in bytes
- Useful for detecting unexpectedly large/small responses

**5. Duration Assertion**
- Validates response time threshold
- Marks sample failed if exceeds specified milliseconds

**6. HTML Assertion**
- Validates HTML structure
- Checks for valid HTML syntax
- Can check errors and warnings

**7. MD5Hex Assertion**
- Compares response MD5 hash
- Ensures response content hasn't changed

#### Assertion Scope

Assertions can be added at:
- **Sampler level**: Applies only to that sampler
- **Thread Group level**: Applies to all samplers in group
- **Test Plan level**: Applies to all samplers in test plan

#### Best Practices

- Always validate HTTP response codes (200, 201, etc.)
- Check for error messages in response
- Validate critical data fields
- Use Duration Assertion for SLA validation
- Balance thoroughness with performance overhead
- Assertions consume resources - don't overuse

### Practical Part

#### Exercise 5.1: Basic Response Assertions

Test API response validation:

1. Create HTTP Request:
   - Server: `jsonplaceholder.typicode.com`
   - Path: `/posts/1`
   - Method: GET

2. Add "Response Assertion":
   - Apply to: Main sample only
   - Field to Test: Response Code
   - Pattern Matching Rules: Equals
   - Patterns to Test: `200`

3. Add second "Response Assertion":
   - Field to Test: Response Body
   - Pattern Matching Rules: Contains
   - Patterns to Test: `userId`

4. Run and verify in View Results Tree (should pass)

5. Modify path to `/posts/999` and run again (should fail response body assertion)

#### Exercise 5.2: JSON Assertion with JSONPath

Validate specific JSON fields:

1. Create HTTP Request:
   - Server: `jsonplaceholder.typicode.com`
   - Path: `/users/1`

2. Add "JSON Assertion":
   - Assert JSON Path exists: `$.name`
   - Expected Value: `Leanne Graham`

3. Add second "JSON Assertion":
   - Assert JSON Path: `$.address.city`
   - Expected Value: `Gwenborough`

4. Add third "JSON Assertion":
   - Assert JSON Path: `$.id`
   - Expected Value: `1`
   - Expect null: false
   - Invert assertion: false

5. Run and verify all assertions pass

#### Exercise 5.3: Duration Assertion for SLA

Implement performance SLA checking:

1. Create HTTP Request:
   - Server: `jsonplaceholder.typicode.com`
   - Path: `/posts`

2. Add "Duration Assertion":
   - Duration in milliseconds: `2000`

3. Set Thread Group: 50 threads, 5-second ramp-up, 10 loops

4. Add "View Results Tree" and "Summary Report"

5. Run test and check how many requests fail duration assertion

6. Adjust threshold and re-test

#### Exercise 5.4: Complex Validation Scenario

Create comprehensive validation:

1. HTTP POST Request:
   - Server: `httpbin.org`
   - Path: `/post`
   - Body Data:
   ```json
   {
     "testId": 12345,
     "status": "active"
   }
   ```

2. Add Response Assertions:
   - Assertion 1: Response code = 200
   - Assertion 2: Response body contains "testId"
   - Assertion 3: Response body contains "12345"

3. Add JSON Assertion:
   - JSONPath: `$.json.status`
   - Expected: `active`

4. Add Duration Assertion: 3000ms

5. Add Size Assertion:
   - Response Size: > 100 bytes

6. Run and review assertion results

#### Exercise 5.5: Negative Testing with Assertions

Test error handling:

1. Create HTTP Request for invalid endpoint:
   - Server: `jsonplaceholder.typicode.com`
   - Path: `/invalid-endpoint`

2. Add Response Assertion:
   - Response Code: `404`
   - This should PASS (expecting error)

3. Create second request for valid endpoint:
   - Path: `/posts/1`

4. Add Response Assertion with "NOT":
   - Field: Response Body
   - Pattern Matching: Contains
   - NOT: checked
   - Pattern: `error`
   - This validates response does NOT contain "error"

5. Run both and observe results

### Control Questions

1. **What happens when an assertion fails in JMeter?**
   <details>
   <summary>Answer</summary>
   The sample/request is marked as failed (shows red in listeners), failure message is displayed, but the test continues running unless configured otherwise
   </details>

2. **Why would you add a Duration Assertion to your test plan?**
   <details>
   <summary>Answer</summary>
   To validate SLA requirements, ensure response times meet performance criteria, identify slow requests, and automatically flag performance regressions
   </details>

3. **What's the JSONPath expression to validate a nested field: `{"user": {"profile": {"age": 25}}}`?**
   <details>
   <summary>Answer</summary>
   `$.user.profile.age`
   </details>

4. **Should assertions be added to every sampler in a load test? Why or why not?**
   <details>
   <summary>Answer</summary>
   Not necessarily. Assertions add overhead and consume resources. Add assertions for critical validations and SLA checks, but avoid over-assertion in high-load tests. Balance validation needs with performance impact.
   </details>

5. **How can you validate that a response does NOT contain certain text?**
   <details>
   <summary>Answer</summary>
   Use Response Assertion with "NOT" checkbox enabled, then specify the pattern that should NOT appear in the response
   </details>

---

## Lesson 6: Timers, Controllers, and Test Flow Control

### Theoretical Part

#### Timers

Timers add delays between requests to simulate real user behavior. Without timers, JMeter executes requests as fast as possible.

**Types of Timers:**

**1. Constant Timer**
- Fixed delay between requests
- Simple, predictable
- Use: Uniform think time

**2. Uniform Random Timer**
- Random delay within range
- Offset + Random delay (0 to specified max)
- Use: More realistic variability

**3. Gaussian Random Timer**
- Random delay with Gaussian distribution
- Constant delay offset + Gaussian random
- Use: Natural user behavior simulation

**4. Poisson Random Timer**
- Based on Poisson distribution
- Simulates random arrivals
- Use: Realistic inter-arrival times

**5. Constant Throughput Timer**
- Controls request rate (requests/minute)
- Attempts to maintain target throughput
- Use: Rate limiting

**6. Precise Throughput Timer (plugin)**
- More accurate throughput control
- Target throughput and ramp-up time
- Use: Precise load shaping

#### Controllers

Controllers define test execution logic and flow.

**Logic Controllers:**

**1. Simple Controller**
- Organizes samplers (folder-like)
- No logic, just grouping
- Helps structure complex tests

**2. Loop Controller**
- Repeats child elements N times
- Independent of thread group loops
- Use: Repeat specific workflow

**3. Once Only Controller**
- Executes children once per thread
- Even in loops
- Use: Login, initialization

**4. If Controller**
- Conditional execution based on expression
- Uses JavaScript evaluation
- Use: Dynamic scenarios

**5. Switch Controller**
- Selects child based on value
- Like switch/case statement
- Use: Different user types

**6. Random Controller**
- Randomly selects one child element
- Equal probability by default
- Use: Varied user behavior

**7. Random Order Controller**
- Executes all children in random order
- Use: Eliminate order dependencies

**8. Interleave Controller**
- Alternates between children
- Each iteration uses next child
- Use: Rotate through scenarios

**9. Transaction Controller**
- Groups samplers as single transaction
- Measures total time
- Can include/exclude sub-sample times
- Use: Business transaction timing

**10. Throughput Controller**
- Controls execution percentage
- Can execute certain % of time
- Use: Weighted scenarios

### Practical Part

#### Exercise 6.1: Adding Think Time with Timers

Simulate realistic user behavior:

1. Create Thread Group: 10 threads, 10 loops

2. Add HTTP Requests (no delays):
   - Request 1: `jsonplaceholder.typicode.com/posts`
   - Request 2: `jsonplaceholder.typicode.com/users`
   - Request 3: `jsonplaceholder.typicode.com/comments`

3. Run without timers, note execution time

4. Add "Uniform Random Timer" at Thread Group level:
   - Random Delay Maximum: 3000
   - Constant Delay Offset: 1000
   - (Think time: 1-4 seconds)

5. Run again, observe slower execution and more realistic timing

#### Exercise 6.2: Constant Throughput Timer

Control request rate:

1. Create Thread Group: 100 threads, 10-second ramp-up

2. Add HTTP Request: `httpbin.org/delay/1`

3. Add "Constant Throughput Timer":
   - Target throughput: 60 (requests per minute = 1/sec)

4. Add "jp@gc - Transactions per Second" listener (plugin) or "Summary Report"

5. Run for 60 seconds

6. Verify throughput stays around 1 req/sec

#### Exercise 6.3: Transaction Controller

Group related requests as business transaction:

1. Add "Transaction Controller":
   - Name: "User Login Flow"
   - Generate parent sample: checked

2. Add children to Transaction Controller:
   - HTTP Request 1: GET login page
   - HTTP Request 2: POST credentials
   - HTTP Request 3: GET dashboard

3. Add "View Results Tree"

4. Run and observe:
   - Individual request times
   - Total transaction time

#### Exercise 6.4: If Controller for Conditional Logic

Create dynamic test based on response:

1. Add HTTP Request: `httpbin.org/status/200`

2. Add "Regular Expression Extractor":
   - Reference Name: `statusCode`
   - Regular Expression: `(\d{3})`
   - Template: `$1$`

3. Add "If Controller":
   - Condition: `${__groovy(vars.get("statusCode")=="200")}`

4. Add HTTP Request as child of If Controller:
   - `httpbin.org/get` (executes only if status 200)

5. Add "View Results Tree" and run

#### Exercise 6.5: Random Controller for User Behavior Variety

Simulate users taking different paths:

1. Add "Random Controller"

2. Add three HTTP Requests as children:
   - Request A: `jsonplaceholder.typicode.com/posts/1`
   - Request B: `jsonplaceholder.typicode.com/users/1`
   - Request C: `jsonplaceholder.typicode.com/albums/1`

3. Set Thread Group: 30 threads, 3 loops

4. Add "Summary Report"

5. Run and observe approximately equal distribution

#### Exercise 6.6: Once Only Controller for Login

Simulate login once per user session:

1. Add "Once Only Controller"

2. Add Login HTTP Request inside:
   - POST `httpbin.org/post`
   - Body: `{"username":"testuser","password":"pass123"}`

3. Add regular HTTP Requests outside controller:
   - Request 1: `httpbin.org/get`
   - Request 2: `httpbin.org/headers`

4. Set Thread Group: 5 threads, 10 loops

5. Run and verify login happens only once per thread (5 times total)

#### Exercise 6.7: Complex Scenario with Multiple Controllers

Build realistic e-commerce test:

```
Thread Group (50 users, 30-sec ramp-up, 5 loops)
├── Once Only Controller (Login)
│   └── HTTP Request: POST /login
├── Uniform Random Timer (2-5 sec think time)
├── Transaction Controller: "Browse Products"
│   ├── HTTP Request: GET /products
│   ├── Random Controller
│   │   ├── HTTP Request: GET /products/category/electronics
│   │   ├── HTTP Request: GET /products/category/books
│   │   └── HTTP Request: GET /products/category/clothing
│   └── Loop Controller (3 iterations)
│       └── HTTP Request: GET /products/${RANDOM_ID}
└── If Controller: "${__Random(1,10)}" > "7"
    └── Transaction Controller: "Purchase"
        ├── HTTP Request: POST /cart/add
        ├── HTTP Request: GET /cart
        └── HTTP Request: POST /checkout
```

### Control Questions

1. **What's the difference between adding a timer at Thread Group level vs Sampler level?**
   <details>
   <summary>Answer</summary>
   Thread Group level: timer applies to ALL samplers in the thread group; Sampler level: timer applies only to that specific sampler
   </details>

2. **Why use a Uniform Random Timer instead of Constant Timer?**
   <details>
   <summary>Answer</summary>
   To simulate more realistic user behavior with natural variations in think time, avoiding synchronized request patterns that don't occur in real usage
   </details>

3. **What does "Generate parent sample" do in Transaction Controller?**
   <details>
   <summary>Answer</summary>
   Creates a single parent sample representing the entire transaction, making it easier to measure end-to-end transaction time rather than only seeing individual request times
   </details>

4. **When would you use Once Only Controller?**
   <details>
   <summary>Answer</summary>
   For operations that should happen only once per virtual user session, like login, setup, initialization, or any action that shouldn't be repeated in loops
   </details>

5. **How does Constant Throughput Timer differ from controlling thread count?**
   <details>
   <summary>Answer</summary>
   Thread count controls concurrent users; Constant Throughput Timer controls request rate (requests per unit time). Timer adds delays to achieve target throughput regardless of thread count
   </details>

---

## Lesson 7: Extractors, Pre-Processors, and Post-Processors

### Theoretical Part

#### Request-Response Processing Flow

```
Pre-Processor → Sampler → Post-Processor → Assertion
```

#### Regular Expression Extractor (Post-Processor)

Extracts data from responses using regex patterns.

**Configuration:**
- **Apply to**: Which sample to extract from
- **Field to check**: Response body, headers, URL, code, message
- **Reference Name**: Variable name to store extracted value
- **Regular Expression**: Regex pattern with capturing groups
- **Template**: Format of extracted value (`$1$`, `$2$`)
- **Match No**: Which match to use (0=random, 1=first, -1=all)
- **Default Value**: Value if extraction fails

**Common Regex Patterns:**
- `"id":(\d+)` - Extract numeric ID from JSON
- `<title>(.*?)</title>` - Extract text between tags
- `token=(.*?)&` - Extract parameter value
- `"sessionId":"(.*?)"` - Extract session ID from JSON

#### JSON Extractor (Post-Processor)

Extracts values from JSON responses using JSONPath.

**Configuration:**
- **Names of variables**: Variable names (semicolon-separated)
- **JSON Path expressions**: JSONPath queries
- **Match No**: Which match (0=random, -1=all, N=specific)
- **Default Values**: If extraction fails

**JSONPath Examples:**
- `$.user.name` - Direct field
- `$..price` - All price fields recursively
- `$.items[0].id` - First item's ID
- `$.users[*].email` - All user emails

#### XPath Extractor (Post-Processor)

Extracts values from XML/HTML responses.

#### CSS/JQuery Extractor (Post-Processor)

Extracts values using CSS selectors or JQuery expressions.

#### Boundary Extractor (Post-Processor)

Extracts text between left and right boundaries.

#### Pre-Processors

Execute before sampler runs:

**1. User Parameters**
- Sets user-specific variables per iteration

**2. HTML Link Parser**
- Parses HTML for links and forms

**3. HTTP URL Re-writing Modifier**
- Modifies URLs for session handling

**4. BeanShell/JSR223 Pre-Processor**
- Custom scripting before request

#### Post-Processors

Execute after sampler completes:

**1. Extractors** (covered above)

**2. BeanShell/JSR223 Post-Processor**
- Custom scripting after response

**3. Debug Post-Processor**
- Displays variables for debugging

### Practical Part

#### Exercise 7.1: Regular Expression Extractor

Extract and reuse data from responses:

**Scenario: Get post ID and use in next request**

1. Request 1 - Get Posts:
   - Server: `jsonplaceholder.typicode.com`
   - Path: `/posts`

2. Add "Regular Expression Extractor":
   - Apply to: Main sample only
   - Field: Response Body
   - Reference Name: `postId`
   - Regular Expression: `"id":(\d+)`
   - Template: `$1$`
   - Match No: 1 (first match)
   - Default: `NOT_FOUND`

3. Add "Debug Sampler" to verify extraction

4. Request 2 - Use Extracted ID:
   - Path: `/posts/${postId}`

5. Add "View Results Tree"

6. Run and verify ID was extracted and used

#### Exercise 7.2: JSON Extractor

Extract multiple values from JSON response:

1. HTTP Request:
   - Server: `jsonplaceholder.typicode.com`
   - Path: `/users/1`

2. Add "JSON Extractor":
   - Names of variables: `userName;userEmail;userCity`
   - JSON Path expressions: `$.name;$.email;$.address.city`
   - Match No: 0
   - Default Values: `NAME_ERROR;EMAIL_ERROR;CITY_ERROR`

3. Add HTTP POST to display extracted data:
   - Server: `httpbin.org`
   - Path: `/post`
   - Body:
   ```json
   {
     "extractedName": "${userName}",
     "extractedEmail": "${userEmail}",
     "extractedCity": "${userCity}"
   }
   ```

4. Run and verify extraction in View Results Tree

#### Exercise 7.3: Correlation - Extract and Reuse Session Token

Simulate authentication with session management:

1. Request 1 - Login:
   - Server: `httpbin.org`
   - Path: `/post`
   - Method: POST
   - Body: `{"username":"user1","password":"pass123"}`

2. Add Response Assertion to simulate token return:
   - (Note: httpbin echoes data back in JSON format)

3. Add JSON Extractor:
   - Variable Name: `authToken`
   - JSON Path: `$.json.username` (simulating token)
   - Match No: 0
   - Default: `NO_TOKEN`

4. Request 2 - Authenticated Request:
   - Server: `httpbin.org`
   - Path: `/get`
   - Add HTTP Header Manager:
     - Name: `Authorization`
     - Value: `Bearer ${authToken}`

5. Run and verify token was extracted and used

#### Exercise 7.4: Extract All Values with Match No = -1

Extract multiple items from array:

1. HTTP Request:
   - Server: `jsonplaceholder.typicode.com`
   - Path: `/posts?userId=1`

2. Add JSON Extractor:
   - Variable Name: `postTitles`
   - JSON Path: `$[*].title`
   - Match No: -1 (all matches)

3. Add Loop Controller:
   - Loop Count: `${postTitles_matchNr}` (built-in counter)

4. Inside Loop, add HTTP Request:
   - Server: `httpbin.org`
   - Path: `/post`
   - Body: `{"title": "${postTitles_${__counter(TRUE,titleCounter)}}"}`

5. Run and observe all titles being used

#### Exercise 7.5: Boundary Extractor

Extract value between specific strings:

1. HTTP Request:
   - Server: `httpbin.org`
   - Path: `/html`

2. Add "Boundary Extractor":
   - Reference Name: `htmlTitle`
   - Left Boundary: `<title>`
   - Right Boundary: `</title>`
   - Match No: 1
   - Default: `NO_TITLE`

3. Add Debug Sampler to view extraction

4. Run and verify title extraction

#### Exercise 7.6: Complex Extraction Chain

Multi-step data extraction:

1. Request 1 - Get User List:
   - Path: `/users`
   - JSON Extractor: Extract all user IDs

2. Loop through user IDs:
   - Request 2 - Get User Details:
     - Path: `/users/${userId}`
     - JSON Extractor: Extract user's company name
   
3. Request 3 - Search posts by company:
   - Use extracted company data

4. Validate entire chain works

### Control Questions

1. **What's the difference between Match No: 0 and Match No: 1 in extractors?**
   <details>
   <summary>Answer</summary>
   Match No: 0 = random match; Match No: 1 = first match; Match No: -1 = all matches (creates numbered variables)
   </details>

2. **When should you use JSON Extractor vs Regular Expression Extractor?**
   <details>
   <summary>Answer</summary>
   JSON Extractor for JSON responses (cleaner, easier, less error-prone); Regular Expression Extractor for non-JSON formats, complex patterns, or when you need more flexibility
   </details>

3. **What happens if extraction fails and no default value is set?**
   <details>
   <summary>Answer</summary>
   The variable will contain the literal string of the variable reference (e.g., "${variableName}") instead of an actual value, which can cause subsequent requests to fail
   </details>

4. **How do you extract all values from a JSON array and use them in subsequent requests?**
   <details>
   <summary>Answer</summary>
   Use Match No: -1, which creates variables with index suffixes (var_1, var_2, etc.) and var_matchNr counter. Use Loop Controller with ${var_matchNr} as loop count
   </details>

5. **What's the order of execution: Pre-Processor, Sampler, Post-Processor, Assertion?**
   <details>
   <summary>Answer</summary>
   Pre-Processor → Sampler → Post-Processor → Assertion
   </details>

---

## Lesson 8: Listeners, Reporting, and Results Analysis

### Theoretical Part

#### Understanding Listeners

Listeners collect test results and provide visualization. They're essential for analysis but consume significant memory.

**Performance Impact:**
- Listeners consume memory and CPU
- "View Results Tree" especially resource-intensive
- Should be disabled for high-load tests
- Use only in development/debugging
- Enable selectively for production tests

#### Common Listeners

**1. View Results Tree**
- Shows individual request/response details
- Great for debugging
- VERY resource-intensive
- Use only with small tests

**2. Summary Report**
- Aggregated statistics per sampler
- Shows: samples, average, min, max, std dev, error %, throughput, KB/sec
- Minimal resource usage
- Good for overall test summary

**3. Aggregate Report**
- Similar to Summary Report
- Includes median and 90th percentile
- Aggregates all data

**4. Graph Results**
- Real-time graphical display
- Shows response time trends
- Resource-intensive
- Better for post-analysis

**5. Response Time Graph**
- Plots response times over duration
- Interval-based averaging
- Good for trend visualization

**6. Simple Data Writer**
- Writes results to file without GUI display
- Minimal overhead
- Essential for large tests
- Can be analyzed later

**7. Backend Listener**
- Sends data to external systems (InfluxDB, Graphite)
- Real-time monitoring integration
- Supports continuous testing

#### Key Metrics to Analyze

**Response Time Metrics:**
- Average: Mean response time
- Median (50th percentile): Middle value
- 90th Percentile: 90% of requests faster than this
- 95th/99th Percentile: For SLA validation
- Min/Max: Range of response times
- Standard Deviation: Response time variability

**Throughput Metrics:**
- Throughput: Requests per second/minute
- KB/sec: Data transfer rate
- Hits per Second: Total transactions per second

**Error Metrics:**
- Error Rate %: Percentage of failed requests
- Error Count: Total number of failures

**Additional Metrics:**
- Active Threads: Concurrent users at any time
- Latency: Time to first byte
- Connect Time: Connection establishment time

#### Analysis Best Practices

1. **Compare against baseline**: Know normal performance
2. **Identify trends**: Watch for degradation over time
3. **Find bottlenecks**: Which requests are slow?
4. **Correlation analysis**: Link metrics (response time vs load)
5. **Percentile analysis**: Don't rely only on averages
6. **Error pattern detection**: When/why do failures occur?

### Practical Part

#### Exercise 8.1: Comparing Listener Types

Understand listener overhead:

1. Create Thread Group: 100 threads, 10-second ramp-up, 50 loops

2. Add HTTP Request: `jsonplaceholder.typicode.com/posts`

3. **Test 1 - With View Results Tree:**
   - Add "View Results Tree"
   - Run test, note execution time and memory usage

4. **Test 2 - Without Listeners:**
   - Disable View Results Tree
   - Clear results
   - Run test again
   - Compare execution time

5. **Test 3 - With Simple Data Writer:**
   - Add "Simple Data Writer"
   - Filename: `results.jtl`
   - Run test
   - Compare performance

#### Exercise 8.2: Understanding Aggregate Report

Comprehensive metric analysis:

1. Create realistic test:
   - Thread Group: 100 threads, 30-sec ramp-up, 20 loops
   - Add multiple HTTP Requests:
     - Fast endpoint: `/posts/1`
     - Medium endpoint: `/posts`
     - Slow endpoint: `/posts?_limit=100`

2. Add "Aggregate Report"

3. Add "Uniform Random Timer": 1000-3000ms

4. Run complete test

5. Analyze Aggregate Report:
   - Which endpoint has highest average response time?
   - What's the 90th percentile for each?
   - Which has highest throughput?
   - Any errors?
   - What's the standard deviation (consistency)?

#### Exercise 8.3: Response Time Analysis

Identify performance issues:

1. Create test with deliberate slow endpoints:
   - Request 1: `httpbin.org/delay/1`
   - Request 2: `httpbin.org/delay/2`
   - Request 3: `httpbin.org/delay/3`

2. Thread Group: 50 threads, 10-second ramp-up, 5 loops

3. Add Listeners:
   - "Response Time Graph"
   - "Summary Report"
   - "jp@gc - Response Times vs Threads" (plugin)

4. Run test

5. Analysis tasks:
   - Which request consistently slowest?
   - How does response time change with load?
   - Are there response time spikes?

#### Exercise 8.4: Generating CSV Results for External Analysis

Create data for external tools:

1. Add "Simple Data Writer":
   - Filename: `load_test_results.jtl`
   - Configure (check boxes):
     - Save as XML: False (CSV format)
     - Save Label: True
     - Save Response Code: True
     - Save Response Message: True
     - Save Successful: True
     - Save Thread Name: True
     - Save Time Stamp: True
     - Save Elapsed Time: True
     - Save Connect Time: True
     - Save Latency: True

2. Create comprehensive test

3. Run test

4. Open CSV file in Excel/spreadsheet

5. Perform manual analysis:
   - Create pivot tables
   - Calculate custom percentiles
   - Generate charts
   - Filter by error conditions

#### Exercise 8.5: SLA Validation Report

Create report validating performance requirements:

**Given SLA Requirements:**
- Average response time < 2000ms
- 95th percentile < 3000ms
- Error rate < 1%
- Throughput > 50 req/sec

1. Create test simulating production load

2. Add relevant listeners:
   - "Aggregate Report"
   - "Summary Report"

3. Run test (minimum 5 minutes)

4. Create custom analysis:
   - Calculate if SLA requirements met
   - Document any violations
   - Identify root causes

#### Exercise 8.6: Backend Listener Integration (Advanced)

Send results to external system:

1. Setup InfluxDB (optional, for practice)

2. Add "Backend Listener":
   - Backend Listener Implementation: org.apache.jmeter.visualizers.backend.influxdb.InfluxdbBackendListenerClient
   - Configure InfluxDB connection

3. Run test

4. Visualize in Grafana/InfluxDB

(Alternative: Use GraphiteBackendListenerClient)

#### Exercise 8.7: Comparative Performance Analysis

Compare different scenarios:

1. **Scenario A: Baseline**
   - 50 users, 10-second ramp-up
   - No think time
   - Run for 5 minutes

2. **Scenario B: Realistic Think Time**
   - 50 users, 10-second ramp-up
   - 2-5 second think time
   - Run for 5 minutes

3. **Scenario C: High Load**
   - 200 users, 30-second ramp-up
   - 2-5 second think time
   - Run for 5 minutes

4. For each scenario:
   - Save results to separate CSV files
   - Document key metrics
   - Compare performance

5. Create comparison report showing:
   - Response time differences
   - Throughput variations
   - Error rate changes
   - System behavior under different loads

### Control Questions

1. **Why should you disable "View Results Tree" listener during large load tests?**
   <details>
   <summary>Answer</summary>
   It consumes enormous amounts of memory storing all request/response data, can cause OutOfMemory errors, significantly impacts test performance, and skews results. Use only for debugging small tests.
   </details>

2. **What's the difference between average response time and 90th percentile response time?**
   <details>
   <summary>Answer</summary>
   Average can be skewed by outliers; 90th percentile means 90% of requests were faster than this value, providing better insight into typical user experience and excluding worst-case outliers
   </details>

3. **If your aggregate report shows 200 samples with 5% error rate, how many requests failed?**
   <details>
   <summary>Answer</summary>
   10 requests failed (200 × 0.05 = 10)
   </details>

4. **What does high standard deviation in response times indicate?**
   <details>
   <summary>Answer</summary>
   Inconsistent performance with high variability; some requests are much faster/slower than average, indicating potential instability, resource contention, or intermittent issues
   </details>

5. **Which listener should you use for minimal performance overhead in production-like load tests?**
   <details>
   <summary>Answer</summary>
   Simple Data Writer (writing to CSV/JTL file) or Backend Listener (sending to external system), with GUI listeners disabled
   </details>

---

## Lesson 9: Advanced Scenarios and Real-World Applications

### Theoretical Part

#### Realistic Test Scenario Design

**Key Principles:**
1. **User Journey Modeling**: Simulate actual user workflows
2. **Realistic Think Time**: Users don't act instantly
3. **Data Variety**: Different users, different data
4. **Pacing**: Control request rate realistically
5. **Mix of Operations**: Read-heavy vs write-heavy
6. **Session Management**: Login, session maintenance, logout
7. **Error Handling**: Retry logic, conditional paths

#### Common Real-World Scenarios

**E-Commerce Application:**
- Browse products (70%)
- Search products (20%)
- Add to cart (15%)
- Checkout (5%)
- View order history (10%)

**Banking Application:**
- Login (100%)
- Check balance (60%)
- Transfer funds (20%)
- View transactions (40%)
- Pay bills (10%)
- Logout (80%)

**Social Media Platform:**
- Login (100%)
- View feed (90%)
- Post content (30%)
- Like/comment (50%)
- View profile (40%)
- Upload media (20%)

#### Workload Modeling

**Load Patterns:**

**1. Constant Load**
- Steady number of users
- Basic baseline testing
- Validate normal operations

**2. Ramp-Up**
- Gradual increase to peak
- Find breaking point
- Observe degradation

**3. Spike**
- Sudden load increase
- Test auto-scaling
- Flash sale simulation

**4. Soak/Endurance**
- Sustained load over hours/days
- Memory leaks
- Resource exhaustion

**5. Step Load**
- Incremental increases
- Find capacity limits
- Identify thresholds

#### Distributed Testing

For testing beyond single machine capacity:

**Master-Slave Configuration:**
- Master: Controls test, collects results
- Slaves: Generate load
- Scales to thousands of users
- Requires network configuration

**Cloud-Based Load Testing:**
- AWS, Azure, GCP instances
- BlazeMeter, Flood.io
- Distributed globally
- Pay-per-use

### Practical Part

#### Exercise 9.1: E-Commerce User Journey

Create realistic shopping scenario:

```
Test Plan: E-Commerce Load Test
├── HTTP Cookie Manager
├── HTTP Header Manager (User-Agent, Accept)
├── CSV Data Set: users.csv (username, password)
├── CSV Data Set: products.csv (productId, productName)
│
├── Thread Group: "Shoppers" (100 users, 60-sec ramp-up, 5 loops)
│   ├── Uniform Random Timer (2000-5000ms think time)
│   │
│   ├── Transaction Controller: "User Login"
│   │   ├── HTTP Request: GET /login
│   │   ├── Regular Expression Extractor: Extract CSRF token
│   │   ├── HTTP Request: POST /login (username, password, token)
│   │   └── Response Assertion: Login successful
│   │
│   ├── Transaction Controller: "Browse Homepage"
│   │   ├── HTTP Request: GET /
│   │   └── Duration Assertion: < 2000ms
│   │
│   ├── Transaction Controller: "Search Products"
│   │   ├── If Controller: ${__Random(1,10)} <= 7 (70% execute)
│   │   │   ├── HTTP Request: GET /search?q=${searchTerm}
│   │   │   └── JSON Extractor: Extract product IDs
│   │
│   ├── Transaction Controller: "View Product Details"
│   │   ├── Loop Controller (3 iterations)
│   │   │   ├── HTTP Request: GET /product/${productId}
│   │   │   ├── Constant Timer: 3000ms
│   │   │   └── Response Assertion: 200 OK
│   │
│   ├── Transaction Controller: "Add to Cart"
│   │   ├── If Controller: ${__Random(1,10)} <= 5 (50% execute)
│   │   │   ├── HTTP Request: POST /cart/add (productId, quantity)
│   │   │   └── JSON Assertion: Success message
│   │
│   ├── Transaction Controller: "View Cart"
│   │   ├── If Controller: ${addToCartSuccess} == true
│   │   │   └── HTTP Request: GET /cart
│   │
│   ├── Transaction Controller: "Checkout Process"
│   │   ├── If Controller: ${__Random(1,10)} <= 2 (20% execute)
│   │   │   ├── HTTP Request: GET /checkout
│   │   │   ├── HTTP Request: POST /checkout/shipping
│   │   │   ├── HTTP Request: POST /checkout/payment
│   │   │   ├── HTTP Request: POST /checkout/confirm
│   │   │   └── Response Assertion: Order confirmed
│   │
│   └── Transaction Controller: "Logout"
│       └── HTTP Request: POST /logout
│
└── Listeners
    ├── Summary Report
    ├── Aggregate Report
    └── Simple Data Writer: results.jtl
```

**Implementation Steps:**

1. Create CSV files:
```csv
# users.csv
username,password
user1,pass1
user2,pass2
user3,pass3

# products.csv
productId,searchTerm
101,laptop
102,phone
103,headphones
```

2. Implement transaction controllers with samplers

3. Add conditional logic for realistic behavior

4. Configure think times between actions

5. Add proper assertions

6. Run test and analyze transaction timings

#### Exercise 9.2: API Load Test with Spike Pattern

Simulate sudden traffic spike:

1. **Install Ultimate Thread Group plugin**
   (Plugins Manager → Available Plugins → jpgc-casutg)

2. **Configure Ultimate Thread Group:**
   ```
   Row 1: Start 10 threads, initial delay 0s, startup 60s, hold 180s, shutdown 30s
   Row 2: Start 500 threads, initial delay 120s, startup 10s, hold 60s, shutdown 20s (SPIKE)
   Row 3: Start 10 threads, initial delay 210s, startup 60s, hold 180s, shutdown 30s
   ```

3. **Add API requests:**
   - GET /api/products
   - POST /api/orders
   - GET /api/users/${userId}

4. **Add listeners:**
   - jp@gc - Active Threads Over Time
   - jp@gc - Response Times Over Time
   - jp@gc - Transactions per Second

5. **Run and analyze:**
   - How did system respond to spike?
   - Did response times increase?
   - Any errors during spike?
   - Recovery time after spike?

#### Exercise 9.3: Banking Application - Complex Workflow

Implement banking transaction flow:

1. **Login (100% users):**
   - 2FA simulation
   - Session token extraction

2. **Random user actions with weights:**
   - Check Balance (60%): Simple GET
   - View Transactions (40%): Paginated results
   - Transfer Money (20%): POST with validation
   - Pay Bill (10%): Complex multi-step

3. **Implement with Throughput Controller:**
   ```
   Thread Group (200 users)
   ├── Once Only Controller
   │   └── Login Flow
   ├── Loop Controller (10 iterations)
   │   ├── Throughput Controller (60%)
   │   │   └── Check Balance
   │   ├── Throughput Controller (40%)
   │   │   └── View Transactions
   │   ├── Throughput Controller (20%)
   │   │   └── Transfer Money Flow
   │   └── Throughput Controller (10%)
   │       └── Pay Bill Flow
   ```

4. **Add complex assertions:**
   - Balance changes correctly
   - Transaction history updates
   - Sufficient funds validation

#### Exercise 9.4: Soak Test (Endurance)

Test application stability over extended period:

1. **Configure Thread Group:**
   - Threads: 100
   - Ramp-up: 300 seconds (5 minutes)
   - Duration: 14400 seconds (4 hours)

2. **Add realistic workflow**

3. **Monitor:**
   - Memory usage over time
   - Response time degradation
   - Error rate increases
   - Resource leaks

4. **Add scheduled reports:**
   - Save results every 15 minutes
   - Monitor for gradual degradation

#### Exercise 9.5: Distributed Load Test

Scale beyond single machine:

**Master Configuration:**
1. Edit jmeter.properties:
   ```
   remote_hosts=slave1_ip:1099,slave2_ip:1099
   server.rmi.ssl.disable=true
   ```

**Slave Configuration:**
1. Start JMeter server on each slave:
   ```bash
   ./jmeter-server
   ```

**Run Distributed Test:**
1. From master GUI: Run → Remote Start All
2. Or command line:
   ```bash
   jmeter -n -t test.jmx -R slave1_ip,slave2_ip -l results.jtl
   ```

3. Configure test to split load across slaves

4. Aggregate results from all slaves

#### Exercise 9.6: Performance Baseline and Regression Testing

Establish performance baseline:

1. **Create baseline test:**
   - Standardized scenario
   - Consistent load (50 users, 30-min duration)
   - Same environment

2. **Run baseline:**
   - Document all metrics
   - Save results: `baseline_v1.0.jtl`

3. **After code changes:**
   - Run same test
   - Save: `regression_v1.1.jtl`

4. **Compare results:**
   - Response time changes
   - Throughput differences
   - New errors
   - Resource utilization changes

5. **Create comparison report:**
   - Use scripting to compare CSV files
   - Flag regressions (e.g., >10% slower)
   - Document improvements

### Control Questions

1. **Why is it important to model realistic user behavior in load tests?**
   <details>
   <summary>Answer</summary>
   To accurately simulate production conditions, identify real bottlenecks, avoid unrealistic scenarios that don't stress the system properly, validate SLAs against actual usage patterns, and make informed capacity planning decisions
   </details>

2. **What's the difference between spike testing and stress testing?**
   <details>
   <summary>Answer</summary>
   Spike testing: sudden, dramatic increase in load for short duration to test system's ability to handle unexpected traffic surges; Stress testing: gradually increasing load beyond normal capacity to find breaking point and system limits
   </details>

3. **In the e-commerce scenario, why use Throughput Controller instead of If Controller for action probabilities?**
   <details>
   <summary>Answer</summary>
   Throughput Controller provides more precise percentage-based execution control, ensures exact execution ratios over test duration, better for modeling user behavior percentages consistently
   </details>

4. **What should you monitor during a soak test to detect memory leaks?**
   <details>
   <summary>Answer</summary>
   Gradual increase in response times, progressive increase in error rates, server memory usage (heap) continuously growing, eventual OutOfMemory errors, degrading throughput over time
   </details>

5. **When would you need distributed (master-slave) load testing?**
   <details>
   <summary>Answer</summary>
   When single machine cannot generate sufficient load, testing for thousands of concurrent users, geographic distribution needed, network bandwidth limitations of single machine, avoiding bottleneck on load generator side
   </details>

---

## Lesson 10: CI/CD Integration and Best Practices

### Theoretical Part

#### Running JMeter in Non-GUI Mode

GUI mode is for development; non-GUI for execution.

**Command Line Syntax:**
```bash
jmeter -n -t test_plan.jmx -l results.jtl -e -o report_folder
```

**Key Parameters:**
- `-n`: Non-GUI mode
- `-t`: Test plan file
- `-l`: Results log file
- `-e`: Generate HTML report
- `-o`: Output folder for HTML report
- `-j`: JMeter log file
- `-J`: Define JMeter property
- `-G`: Define property file
- `-R`: Remote hosts for distributed testing

**Examples:**
```bash
# Basic execution
jmeter -n -t LoadTest.jmx -l results.jtl

# With HTML report
jmeter -n -t LoadTest.jmx -l results.jtl -e -o html-report

# Override properties
jmeter -n -t LoadTest.jmx -l results.jtl -Jusers=100 -Jrampup=60

# Distributed testing
jmeter -n -t LoadTest.jmx -R server1,server2 -l results.jtl
```

#### CI/CD Integration

**Integration Points:**
1. **Automated execution**: Run tests on code commits/merges
2. **Performance gates**: Fail builds on regression
3. **Trend analysis**: Track metrics over time
4. **Reporting**: Publish results to team
5. **Alerting**: Notify on failures

**Common CI/CD Tools:**
- Jenkins
- GitLab CI
- GitHub Actions
- Azure DevOps
- CircleCI
- Bamboo

#### Jenkins Integration

**Jenkins Performance Plugin:**
- Parses JMeter results
- Generates trend graphs
- Sets performance thresholds
- Fails builds on violations

**Pipeline Example:**
```groovy
pipeline {
    agent any
    stages {
        stage('Performance Test') {
            steps {
                sh 'jmeter -n -t test.jmx -l results.jtl -e -o report'
            }
        }
        stage('Publish Report') {
            steps {
                perfReport sourceDataFiles: 'results.jtl',
                    errorFailedThreshold: 5,
                    errorUnstableThreshold: 2
            }
        }
    }
}
```

#### Performance Thresholds

Define acceptable limits:
```
- Average response time < 2000ms
- 90th percentile < 3000ms
- Error rate < 1%
- Throughput > 100 req/sec
```

**Threshold Strategies:**
1. **Hard limits**: Absolute values
2. **Relative comparison**: % change from baseline
3. **Degradation detection**: Trend analysis

#### Best Practices

**1. Test Plan Organization:**
- Modular test plans
- Reusable components
- Clear naming conventions
- Version control

**2. Environment Management:**
- Separate test environments
- Consistent test data
- Isolated infrastructure
- Database state management

**3. Test Data:**
- Sufficient variety
- Realistic distribution
- Data cleanup
- Data privacy compliance

**4. Results Management:**
- Consistent storage
- Long-term archival
- Trend databases
- Easy comparison

**5. Reporting:**
- Automated generation
- Stakeholder-friendly format
- Actionable insights
- Historical comparison

**6. Maintenance:**
- Regular test reviews
- Update for application changes
- Parameterize environment-specific values
- Document test scenarios

### Practical Part

#### Exercise 10.1: Running Tests in Non-GUI Mode

Execute test from command line:

1. **Create simple test plan** (if not already created)

2. **Save test plan:** `APILoadTest.jmx`

3. **Run in command line:**
   ```bash
   jmeter -n -t APILoadTest.jmx -l results.jtl -j jmeter.log
   ```

4. **Review results file:**
   ```bash
   # View CSV results
   cat results.jtl | head -20
   ```

5. **Generate HTML report:**
   ```bash
   jmeter -n -t APILoadTest.jmx -l results.jtl -e -o html-report
   ```

6. **Open HTML report** in browser: `html-report/index.html`

7. **Compare:** GUI execution time vs non-GUI execution time

#### Exercise 10.2: Parameterized Test Execution

Override test parameters from command line:

1. **Add User Defined Variables to test plan:**
   - `THREADS`: 50
   - `RAMPUP`: 30
   - `DURATION`: 300
   - `SERVER`: test-server.com

2. **Use variables in Thread Group:**
   - Number of Threads: `${__P(THREADS,50)}`
   - Ramp-up: `${__P(RAMPUP,30)}`
   - Duration: `${__P(DURATION,300)}`

3. **Use in HTTP Request:**
   - Server: `${__P(SERVER,test-server.com)}`

4. **Run with different parameters:**
   ```bash
   # Low load test
   jmeter -n -t test.jmx -JTHREADS=10 -JRAMPUP=10 -l low-load.jtl

   # Medium load test
   jmeter -n -t test.jmx -JTHREADS=50 -JRAMPUP=30 -l med-load.jtl

   # High load test
   jmeter -n -t test.jmx -JTHREADS=200 -JRAMPUP=60 -l high-load.jtl

   # Production server test
   jmeter -n -t test.jmx -JSERVER=prod-server.com -l prod-test.jtl
   ```

5. **Compare results** from different runs

#### Exercise 10.3: GitHub Actions Integration

Create automated performance test:

**Create `.github/workflows/performance-test.yml`:**

```yaml
name: Performance Testing

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  schedule:
    - cron: '0 2 * * *'  # Daily at 2 AM

jobs:
  performance-test:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2

    - name: Setup Java
      uses: actions/setup-java@v2
      with:
        java-version: '11'
        distribution: 'adopt'

    - name: Download JMeter
      run: |
        wget https://archive.apache.org/dist/jmeter/binaries/apache-jmeter-5.5.tgz
        tar -xzf apache-jmeter-5.5.tgz
        
    - name: Run JMeter Test
      run: |
        apache-jmeter-5.5/bin/jmeter -n -t tests/LoadTest.jmx \
          -l results/results.jtl \
          -e -o results/html-report \
          -JTHREADS=50 -JDURATION=300

    - name: Upload Results
      uses: actions/upload-artifact@v2
      with:
        name: jmeter-results
        path: results/

    - name: Check Performance Thresholds
      run: |
        # Simple threshold check (you can enhance this)
        ERROR_RATE=$(awk -F',' '{if($8=="false") count++} END {print (count/NR)*100}' results/results.jtl)
        if (( $(echo "$ERROR_RATE > 5" | bc -l) )); then
          echo "Error rate $ERROR_RATE% exceeds threshold!"
          exit 1
        fi
```

#### Exercise 10.4: Jenkins Pipeline Integration

**Create Jenkinsfile:**

```groovy
pipeline {
    agent any
    
    parameters {
        choice(name: 'TEST_TYPE', choices: ['smoke', 'load', 'stress'], description: 'Test type to run')
        string(name: 'THREADS', defaultValue: '50', description: 'Number of threads')
        string(name: 'DURATION', defaultValue: '300', description: 'Test duration in seconds')
    }
    
    environment {
        JMETER_HOME = '/opt/jmeter'
        TEST_PLAN = "tests/${params.TEST_TYPE}-test.jmx"
    }
    
    stages {
        stage('Setup') {
            steps {
                echo "Running ${params.TEST_TYPE} test with ${params.THREADS} threads"
                sh 'mkdir -p results'
            }
        }
        
        stage('Execute JMeter Test') {
            steps {
                sh """
                    ${JMETER_HOME}/bin/jmeter -n \
                        -t ${TEST_PLAN} \
                        -l results/results-${BUILD_NUMBER}.jtl \
                        -e -o results/html-report-${BUILD_NUMBER} \
                        -JTHREADS=${params.THREADS} \
                        -JDURATION=${params.DURATION}
                """
            }
        }
        
        stage('Performance Analysis') {
            steps {
                script {
                    // Parse results and check thresholds
                    def results = readFile("results/results-${BUILD_NUMBER}.jtl")
                    // Add threshold checking logic
                }
                
                // Publish HTML report
                publishHTML([
                    reportDir: "results/html-report-${BUILD_NUMBER}",
                    reportFiles: 'index.html',
                    reportName: "JMeter Report ${BUILD_NUMBER}"
                ])
            }
        }
        
        stage('Performance Plugin Report') {
            steps {
                perfReport(
                    sourceDataFiles: "results/results-${BUILD_NUMBER}.jtl",
                    errorFailedThreshold: 5,
                    errorUnstableThreshold: 2,
                    relativeFailedThresholdPositive: 20,
                    relativeUnstableThresholdPositive: 10
                )
            }
        }
    }
    
    post {
        always {
            archiveArtifacts artifacts: 'results/**/*', fingerprint: true
        }
        failure {
            emailext(
                subject: "Performance Test Failed: ${env.JOB_NAME} - Build ${env.BUILD_NUMBER}",
                body: "Performance test failed. Check console output.",
                to: "team@example.com"
            )
        }
    }
}
```

#### Exercise 10.5: Threshold Validation Script

Create automated threshold checker:

**Create `check_thresholds.py`:**

```python
#!/usr/bin/env python3
import csv
import sys
from statistics import mean, quantiles

def analyze_results(jtl_file):
    response_times = []
    errors = 0
    total = 0
    
    with open(jtl_file, 'r') as f:
        reader = csv.DictReader(f)
        for row in reader:
            total += 1
            response_times.append(int(row['elapsed']))
            if row['success'] == 'false':
                errors += 1
    
    # Calculate metrics
    avg_response_time = mean(response_times)
    percentiles = quantiles(response_times, n=100)
    p90 = percentiles[89]
    p95 = percentiles[94]
    error_rate = (errors / total) * 100
    
    # Define thresholds
    THRESHOLDS = {
        'avg_response_time': 2000,
        'p90': 3000,
        'p95': 4000,
        'error_rate': 1.0
    }
    
    # Check violations
    violations = []
    
    if avg_response_time > THRESHOLDS['avg_response_time']:
        violations.append(f"Average response time {avg_response_time}ms exceeds {THRESHOLDS['avg_response_time']}ms")
    
    if p90 > THRESHOLDS['p90']:
        violations.append(f"90th percentile {p90}ms exceeds {THRESHOLDS['p90']}ms")
    
    if p95 > THRESHOLDS['p95']:
        violations.append(f"95th percentile {p95}ms exceeds {THRESHOLDS['p95']}ms")
    
    if error_rate > THRESHOLDS['error_rate']:
        violations.append(f"Error rate {error_rate:.2f}% exceeds {THRESHOLDS['error_rate']}%")
    
    # Report results
    print("Performance Test Results:")
    print(f"  Total Requests: {total}")
    print(f"  Average Response Time: {avg_response_time:.2f}ms")
    print(f"  90th Percentile: {p90}ms")
    print(f"  95th Percentile: {p95}ms")
    print(f"  Error Rate: {error_rate:.2f}%")
    print()
    
    if violations:
        print("THRESHOLD VIOLATIONS:")
        for violation in violations:
            print(f"  ❌ {violation}")
        return 1
    else:
        print("✅ All thresholds passed!")
        return 0

if __name__ == "__main__":
    if len(sys.argv) != 2:
        print("Usage: python check_thresholds.py <jtl_file>")
        sys.exit(1)
    
    exit_code = analyze_results(sys.argv[1])
    sys.exit(exit_code)
```

**Usage:**
```bash
jmeter -n -t test.jmx -l results.jtl
python check_thresholds.py results.jtl
```

#### Exercise 10.6: Performance Trend Database

Store and track performance metrics over time:

**Create `store_metrics.py`:**

```python
#!/usr/bin/env python3
import csv
import sqlite3
from datetime import datetime
from statistics import mean, quantiles

def store_results(jtl_file, build_number, db_file='performance_trends.db'):
    # Parse JTL file
    response_times = []
    errors = 0
    total = 0
    
    with open(jtl_file, 'r') as f:
        reader = csv.DictReader(f)
        for row in reader:
            total += 1
            response_times.append(int(row['elapsed']))
            if row['success'] == 'false':
                errors += 1
    
    # Calculate metrics
    avg_rt = mean(response_times)
    p90 = quantiles(response_times, n=100)[89]
    p95 = quantiles(response_times, n=100)[94]
    error_rate = (errors / total) * 100
    
    # Store in database
    conn = sqlite3.connect(db_file)
    cursor = conn.cursor()
    
    cursor.execute('''
        CREATE TABLE IF NOT EXISTS performance_metrics (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            timestamp TEXT,
            build_number TEXT,
            total_requests INTEGER,
            avg_response_time REAL,
            p90_response_time INTEGER,
            p95_response_time INTEGER,
            error_rate REAL
        )
    ''')
    
    cursor.execute('''
        INSERT INTO performance_metrics 
        (timestamp, build_number, total_requests, avg_response_time, 
         p90_response_time, p95_response_time, error_rate)
        VALUES (?, ?, ?, ?, ?, ?, ?)
    ''', (
        datetime.now().isoformat(),
        build_number,
        total,
        avg_rt,
        p90,
        p95,
        error_rate
    ))
    
    conn.commit()
    conn.close()
    
    print(f"Metrics stored for build {build_number}")

if __name__ == "__main__":
    import sys
    if len(sys.argv) != 3:
        print("Usage: python store_metrics.py <jtl_file> <build_number>")
        sys.exit(1)
    
    store_results(sys.argv[1], sys.argv[2])
```

### Control Questions

1. **Why should JMeter tests be run in non-GUI mode for actual load testing?**
   <details>
   <summary>Answer</summary>
   GUI consumes significant resources (memory, CPU), impacts test accuracy, creates performance overhead, can cause OutOfMemory errors. Non-GUI mode is lightweight, accurate, and suitable for high-load testing
   </details>

2. **How can you override JMeter properties from the command line?**
   <details>
   <summary>Answer</summary>
   Use -J flag: `jmeter -JTHREADS=100 -JSERVER=prod.com -n -t test.jmx -l results.jtl` Combined with ${__P(propertyName,defaultValue)} in test plan
   </details>

3. **What are the key components of a good performance testing CI/CD pipeline?**
   <details>
   <summary>Answer</summary>
   1) Automated test execution on commits/merges; 2) Parameterized test runs; 3) Threshold validation; 4) Automated reporting; 5) Trend tracking; 6) Failure notifications; 7) Artifact archiving
   </details>

4. **Why is trend analysis important in performance testing?**
   <details>
   <summary>Answer</summary>
   Identifies performance regression over time, detects gradual degradation, validates optimization efforts, enables proactive capacity planning, provides data-driven decision making
   </details>

5. **What strategies can you use to handle test data in CI/CD pipelines?**
   <details>
   <summary>Answer</summary>
   1) Version control test data with test plans; 2) Generate test data programmatically; 3) Use containerized databases with known states; 4) Implement data cleanup in setUp/tearDown; 5) Use separate test environment with isolated data
   </details>

---

## Appendix A: JMeter Configuration and Optimization

### JMeter Memory Settings

**Edit `jmeter.bat` (Windows) or `jmeter` (Linux/Mac):**

```bash
# Default
HEAP="-Xms1g -Xmx1g -XX:MaxMetaspaceSize=256m"

# For larger tests
HEAP="-Xms2g -Xmx4g -XX:MaxMetaspaceSize=512m"

# For very large tests
HEAP="-Xms4g -Xmx8g -XX:MaxMetaspaceSize=1g"
```

### Important jmeter.properties Settings

```properties
# Disable unnecessary components in CLI mode
jmeterengine.force.system.exit=true
jmeter.save.saveservice.output_format=csv
jmeter.save.saveservice.print_field_names=true

# For distributed testing
server.rmi.ssl.disable=true
remote_hosts=127.0.0.1

# Thread settings
server.rmi.port=1099

# Logging
log_level.jmeter=INFO
log_level.jmeter.engine=DEBUG
```

### Performance Optimization Tips

1. **Use CSV output**, not XML
2. **Disable unnecessary listeners** during execution
3. **Use Simple Data Writer** instead of View Results Tree
4. **Increase Java heap** for large tests
5. **Use assertions judiciously**
6. **Avoid using Debug Sampler** in production tests
7. **Use Non-GUI mode** for actual testing
8. **Implement proper think times**

---

## Appendix B: Useful JMeter Functions

### Common Functions

```
${__time(yyyy-MM-dd HH:mm:ss)}  # Current timestamp
${__Random(1,100)}               # Random number 1-100
${__RandomString(10,abcd)}       # Random string length 10
${__UUID()}                      # Generate UUID
${__threadNum}                   # Current thread number
${__counter(TRUE,myCounter)}     # Counter
${__eval(${var}*2)}             # Evaluate expression
${__groovy(vars.get("myVar"))}  # Groovy expression
${__base64Encode(text)}         # Base64 encode
${__urlencode(text)}            # URL encode
${__property(propName)}          # Get property
${__P(propName,default)}        # Get property with default
${__V(varName)}                 # Evaluate variable name
```

---

## Appendix C: Regular Expression Examples

### Common Patterns for Extraction

```regex
# Extract JSON value
"id":(\d+)
"name":"(.*?)"

# Extract from HTML
<title>(.*?)</title>
<div class="price">(.*?)</div>

# Extract session tokens
sessionId=([^&]+)
JSESSIONID=([^;]+)

# Extract from headers
Location: https://.*?/([^/]+)$

# Extract email
([a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,})

# Extract UUID
([0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12})
```

---

## Appendix D: JSONPath Examples

```jsonpath
# Root object
$

# Direct field
$.user.name

# Array element by index
$.users[0].email

# All elements in array
$.users[*].name

# Nested field
$.data.user.profile.age

# Conditional
$.users[?(@.age > 25)]

# Recursive descent
$..price
```

---

## Appendix E: Recommended Plugins

Essential plugins from JMeter Plugins Manager:

1. **Custom Thread Groups**
   - Ultimate Thread Group
   - Stepping Thread Group
   - Concurrency Thread Group

2. **Graphs and Listeners**
   - Response Times vs Threads
   - Transactions per Second
   - Active Threads Over Time
   - Response Times Over Time

3. **Samplers**
   - Dummy Sampler
   - Parallel Controller

4. **Other Tools**
   - Parameterized Controller
   - Throughput Shaping Timer

---

## Course Summary and Next Steps

### Key Takeaways

1. **Performance testing** validates system behavior under load
2. **JMeter** is powerful, flexible, and open-source
3. **Realistic scenarios** require proper modeling: think time, data variety, user journeys
4. **Correlation and parameterization** are essential for dynamic applications
5. **Assertions** validate functional and performance requirements
6. **Non-GUI execution** is mandatory for actual load tests
7. **CI/CD integration** enables continuous performance validation
8. **Analysis and reporting** drive actionable improvements

### Continuing Your Learning

**Resources:**
- Official JMeter Documentation: https://jmeter.apache.org/usermanual/
- JMeter Plugins: https://jmeter-plugins.org/
- BlazeMeter University: Free JMeter courses
- Community Forums: Stack Overflow, JMeter Users mailing list

**Practice Projects:**
1. Test your own applications
2. Contribute to open-source test scenarios
3. Build reusable test frameworks
4. Create custom plugins
5. Implement full CI/CD pipelines

**Advanced Topics to Explore:**
- WebSocket testing
- MQTT protocol testing
- Custom protocol development
- Advanced scripting (Groovy, BeanShell)
- Cloud-based load testing
- Performance engineering best practices

---

## Final Assessment

Create a comprehensive performance test for an e-commerce API:

**Requirements:**
1. Implement realistic user journey (browse, search, add to cart, checkout)
2. Use CSV parameterization for users and products
3. Implement proper correlation for session tokens
4. Add comprehensive assertions
5. Configure realistic workload (500 concurrent users, 30-minute duration)
6. Include think times and proper pacing
7. Create CI/CD integration script
8. Generate HTML report
9. Implement threshold validation
10. Document all scenarios and findings

**Deliverables:**
- Complete JMeter test plan (.jmx)
- Test data CSV files
- CI/CD pipeline configuration
- Threshold validation script
- Test results and HTML report
- Analysis document with recommendations

**Evaluation Criteria:**
- Test plan organization and structure
- Realistic scenario modeling
- Proper use of JMeter elements
- Correlation and parameterization
- Assertion coverage
- CI/CD integration quality
- Reporting and documentation

---

**Course Complete! You are now equipped to perform professional load testing with Apache JMeter.**

Good luck with your performance testing journey!
