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

#### 2. Tier Your Services and Dependencies

Not all services are created equal. Tiering helps you apply the right level of rigor to the right components.

-   **Tier 1 (Critical):** Core services that are essential for the business to function. These have the tightest SLOs (e.g., 99.99%).
    -   *Example:* Authentication service, payment processing.
-   **Tier 2 (Important):** Services that are important but have workarounds or are less user-facing. They can have slightly looser SLOs (e.g., 99.9%).
    -   *Example:* Recommendation engine, user profile update page.
-   **Tier 3 (Non-Critical):** Internal tools or background processes. These can have the most lenient SLOs (e.g., 99.5% or 99%).
    -   *Example:* A batch job that generates weekly reports.

-   **Account for Dependencies:** Your service's reliability is limited by the reliability of its dependencies.
    -   If your service depends on a Tier 2 database with a 99.9% SLO, you cannot promise a 99.99% SLO for your service.
    -   Document these dependencies and their SLOs. Your SLO must be *less than or equal to* the SLO of your most critical dependency.

#### 3. Iterate and Refine

Your first SLOs won't be perfect. The goal is to start somewhere and improve over time.

-   **Start with a reasonable baseline:** Look at your historical performance over the last month. If your availability was 99.8%, setting a 99.9% SLO is a good starting point. Setting it to 99.999% immediately is unrealistic.
-   **Review regularly:** Revisit your SLOs quarterly. Are they still relevant? Are they too easy or too hard to meet? Are customers happy? Use this feedback to adjust your targets.

By focusing on user journeys and tiering your services, you can create SLOs that are both meaningful and achievable, striking the right balance between reliability and the speed of development.

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
