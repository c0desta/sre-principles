# Service Level Management: SLIs, SLOs, and Error Budgets

Service Level Indicators (SLIs), Service Level Objectives (SLOs), and Error Budgets are the cornerstone of Site Reliability Engineering. They provide a data-driven framework for defining, measuring, and managing the reliability of a service.

### Service Level Indicator (SLI)
An SLI is a quantitative measure of some aspect of the service. It is a direct measurement of your service's performance from the user's perspective.

-   **What to measure:** Choose metrics that reflect user happiness. Common SLIs include availability, latency, and error rate.
-   **Formula (Availability):** `(Number of successful requests / Total number of valid requests) * 100`
-   **Formula (Latency):** `(Number of requests served faster than a threshold / Total number of valid requests) * 100`

### Service Level Objective (SLO)
An SLO is a target value or range for an SLI, measured over a specific period. This is your internal goal for how the service should perform.

-   **Example:** "99.9% of homepage requests will be served in under 300ms over a rolling 28-day window."
-   **Key characteristics:** SLOs should be user-centric, measurable, and realistic. They are more aggressive than SLAs.

### Defining Meaningful SLOs: A Practical Guide

Defining the right SLO is often the most challenging part of the process. It's a balance between ensuring customer satisfaction and allowing for innovation. As you noted, being too strict can paralyze development, while being too lenient leads to unhappy users. Here’s a practical approach to get started:

#### 1. Start with Critical User Journeys (CUJs)

Instead of measuring everything, focus on what matters most to your users. A Critical User Journey is a sequence of steps a user takes to accomplish a key task in your application.

-   **How to identify them:**
    -   **Ask Product Managers:** What are the top 3-5 most important things a user does with our service?
    -   **Analyze Usage Data:** What are the most frequently used features?
    -   **Examples:**
        -   *E-commerce:* User searches for a product, adds it to the cart, and completes checkout.
        -   *Social Media:* User logs in, scrolls the feed, and posts a comment.
        -   *API Service:* A client authenticates, makes a primary data request, and receives a valid response.

-   **Build SLOs around CUJs:**
    -   **Availability SLO:** Was the user able to complete the checkout process successfully?
    -   **Latency SLO:** How long did it take for the product search results to load?

#### 2. Tier Your Services, Dependencies, and Architecture

Not all services are created equal. Your SLO must be grounded in the reality of your business needs and your system's architecture. Tiering, dependencies, and single points of failure (SPOFs) are critical inputs.

-   **Tiering and Business Impact:** Tiers directly map to SLO targets. The higher the tier, the tighter the SLO, because the business impact of failure is greater.
    -   **Tier 1 (Critical):** Direct impact on revenue or core user experience. Requires the highest SLOs (e.g., 99.99%). *Error Budget (30 days): ~4.3 minutes.*
    -   **Tier 2 (Important):** Degrades user experience but doesn't stop the business. Can have looser SLOs (e.g., 99.9%). *Error Budget (30 days): ~43 minutes.*
    -   **Tier 3 (Non-Critical):** Internal tools or asynchronous tasks. Can have the most lenient SLOs (e.g., 99.5%). *Error Budget (30 days): ~3.6 hours.*

-   **The Reliability Chain of Dependencies:** Your service's maximum possible SLO is capped by the SLO of its weakest critical dependency. Think of it as a chain—it's only as strong as its weakest link.
    -   **SLO Math:** `Your_Service_SLO ≤ min(Dependency1_SLO, Dependency2_SLO, ...)`
    -   **Practical Example:** If your service relies on a database with a 99.95% SLO and an authentication service with a 99.9% SLO, your service's SLO cannot realistically be higher than 99.9%.
    -   **Engineering Around Dependencies:** To achieve an SLO *higher* than a dependency, you must add resilience. This could mean adding a cache to survive database downtime, implementing a fallback to a secondary provider, or designing for graceful degradation.

-   **Single Points of Failure (SPOFs):** A SPOF is any component in your system that does not have redundancy. The existence of a SPOF places a hard ceiling on your availability.
    -   **Impact on SLOs:** If you have a critical component running on a single server, your SLO cannot be higher than the expected availability of that server. No amount of software perfection can fix this.
    -   **Identifying SPOFs:** Look for anything in your architecture diagram that doesn't have a backup or a failover path. This includes single load balancers, single database instances, or services running in only one availability zone.
    -   **The Goal:** To achieve high SLOs (e.g., >99.9%), you must systematically identify and eliminate single points of failure through redundancy and automated failover.

#### 3. Iterate and Refine

Your first SLOs won't be perfect. The goal is to start somewhere and improve over time.

-   **Start with a reasonable baseline:** Look at your historical performance over the last month. If your availability was 99.8%, setting a 99.9% SLO is a good starting point. Setting it to 99.999% immediately is unrealistic.
-   **Review regularly:** Revisit your SLOs quarterly. Are they still relevant? Are they too easy or too hard to meet? Are customers happy? Use this feedback to adjust your targets.

By focusing on user journeys and tiering your services, you can create SLOs that are both meaningful and achievable, striking the right balance between reliability and the speed of development.

### Implementing Availability SLIs for Different Protocols

Measuring availability isn't one-size-fits-all. The right SLI depends on the protocol and what "success" means for the user. The core formula for an availability SLI is often `(Good Events / Valid Events) * 100`, but defining "Good" and "Valid" events is the key.

#### 1. HTTP API Backend
This is the most common case. Availability is measured by the success rate of server responses.

-   **Valid Events:** All legitimate, non-malformed API requests. It's standard practice to **exclude client errors (4xx status codes)**, as these are not server faults.
-   **Good Events:** Requests that return a successful status code, typically `2xx`. Server errors (`5xx` codes) are definitively not good events.
-   **SLI Formula:** `(Count of 2xx responses) / (Total requests - Count of 4xx responses) * 100`
-   **How to Measure:** Easily measured at the load balancer, API gateway, or in application middleware. Tools like Prometheus, Datadog, or cloud-native monitoring can track these status codes automatically.

#### 2. HTTP Frontend (Web Application)
For a frontend, server uptime is not enough. The user must be able to load and interact with the page.

-   **Valid Events:** A user attempting to load a page in their browser.
-   **Good Events:** The page's core content renders successfully and becomes interactive within an acceptable time threshold.
-   **SLI Formula:** `(Count of successful page loads) / (Total page load attempts) * 100`
-   **How to Measure:**
    -   **Real User Monitoring (RUM):** This is the gold standard. A JavaScript agent in the user's browser reports on page load success, timings (like Largest Contentful Paint), and JavaScript errors. Tools include Datadog RUM, New Relic Browser, and Sentry.
    -   **Synthetic Monitoring:** Automated probes that periodically load your site from different locations to test availability and performance, catching issues even when traffic is low.

#### 3. MQTT (IoT / Messaging)
For a message broker, availability is about the ability to reliably publish and subscribe to messages.

-   **Valid Events:** A client attempting to publish a message to a topic.
-   **Good Events:** The message is successfully accepted by the broker (`PUBACK` received) and delivered to all subscribed, connected clients.
-   **SLI Formula (Publish Reliability):** `(Count of PUBACKs received) / (Total PUBLISH packets sent) * 100`
-   **How to Measure:** This requires instrumenting your MQTT broker (e.g., Mosquitto, VerneMQ) or using a managed service (e.g., AWS IoT Core) that provides these metrics. You must monitor the broker's logs or metrics endpoints for publish and acknowledgement events.

#### 4. WebRTC (Real-time Communication)
WebRTC is complex. Availability is about successfully establishing a connection and maintaining stream quality.

-   **Valid Events:** A user attempting to initiate a peer-to-peer session (e.g., start a video call).
-   **Good Events:** A session is successfully established (the ICE connection process completes) and maintains a minimum quality for its duration.
-   **SLI Formula (Session Establishment):** `(Count of successfully established sessions) / (Total session attempts) * 100`
-   **SLI for Stream Quality (Advanced):** `% of session time where packet loss < 2% AND jitter < 30ms`.
-   **How to Measure:** This relies heavily on client-side metrics. The WebRTC `getStats()` API in the browser provides detailed statistics (packet loss, jitter, round-trip time). You must collect these stats from your clients and send them to a monitoring backend to calculate your SLIs.

### Advanced SLI Examples and Dimensions

Beyond basic availability, SLOs can be defined for many other aspects of a service. Here are examples for different types of systems and a look at the "correctness" dimension.

#### 1. Asynchronous Batch Jobs / Data Pipelines
For systems that process data offline, users care about how up-to-date (fresh) and accurate the data is.

-   **SLI Type:** Freshness
-   **SLI Formula:** `% of data processing jobs that complete within X hours of the data being generated.`
-   **How to Measure:** Instrument your pipeline to record timestamps at key stages (e.g., data arrival, job completion). A monitoring system can then calculate the lag.

#### 2. Storage Systems (e.g., Object Storage)
For storage, users care that their data is safe (durability) and accessible.

-   **SLI Type:** Availability, Latency
-   **Availability SLI:** `% of API calls (e.g., PUT, GET, LIST) that complete successfully.`
-   **Latency SLI:** `% of GET requests where the time to first byte is < X ms.`
-   **How to Measure:** Use monitoring probes (canaries) that continuously write a test object, read it back, and then delete it, measuring the success and latency of each step.

#### 3. Streaming Media (HLS/DASH)
For on-demand video, the user experience is defined by a successful start and uninterrupted playback.

-   **SLI Type:** Rebuffering Ratio
-   **SLI Formula:** `(Total session time spent rebuffering) / (Total session playback time)`. A lower ratio is better.
-   **How to Measure:** This requires a video player with client-side instrumentation (e.g., Mux, JW Player Analytics) that reports playback quality metrics.

#### A New Dimension: Correctness
A service can be available and fast, but still useless if it returns the wrong data. A correctness SLI is a powerful way to measure this.

-   **What it is:** A measure of whether the data returned by a service is accurate and valid.
-   **Example:** A weather API that returns a `200 OK` but shows yesterday's forecast has a correctness problem.
-   **SLI Formula:** `% of responses that are validated as correct.`
-   **How to Measure:** This is highly context-specific and can be challenging. Methods include:
    -   **Canary Checks:** Probes that query the service with a known input and check for a known, correct output.
    -   **Out-of-Band Validation:** A separate process that periodically samples data from the live system and runs deeper validation checks against a source of truth.

### Service Level Agreement (SLA)
An SLA is a formal contract between a service provider and a customer. It defines the level of service expected and often includes penalties for failing to meet the defined objectives.

-   **Relationship to SLO:** SLAs are typically a looser version of your SLOs. They are an external promise, while SLOs are an internal target.
-   **Example:** An SLA might promise 99.5% availability, while the internal SLO is 99.9%, giving the team a buffer.

### Error Budget
The Error Budget is the amount of unreliability you are willing to tolerate. It is derived directly from your SLO.

-   **Formula:** `Error Budget = 100% - SLO`
-   **Example (Time-based):**
    -   SLO: 99.9% availability for a 30-day month (43,200 minutes).
    -   Error Budget: `(1 - 0.999) * 43,200 = 43.2 minutes` of acceptable downtime per month.
-   **Example (Request-based):**
    -   SLO: 99.95% success rate for 10 million requests in a month.
    -   Error Budget: `(1 - 0.9995) * 10,000,000 = 5,000` acceptable failed requests.

**How to Use the Error Budget:**
The error budget is a key tool for data-driven decision-making.
-   **If you have budget left:** You can ship new features, perform risky maintenance, or take other actions that might consume the budget.
-   **If you are close to exhausting the budget:** All efforts should shift towards improving reliability. This means freezing new releases, focusing on bug fixes, and strengthening tests.
