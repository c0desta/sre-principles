# Toil Reduction: The SRE Way

Toil is the kind of work that is manual, repetitive, automatable, tactical, devoid of enduring value, and scales linearly as a service grows. A core principle of SRE is to progressively eliminate toil to free up engineers for more valuable, long-term engineering work.

### Characteristics of Toil

To identify toil, look for work with these characteristics:

-   **Manual:** A human has to perform the task by hand.
-   **Repetitive:** The task is performed over and over again.
-   **Automatable:** A machine could perform the task more reliably and faster.
-   **Tactical:** It's reactive work, often driven by interrupts, rather than strategic or proactive.
-   **No Enduring Value:** After the task is done, the service is in the same state as before. It hasn't been improved.
-   **O(n) Scaling:** The amount of work scales with the size or usage of the service. More users or more machines mean more toil.

**Examples of Toil:**
-   Manually provisioning a new customer account.
-   Restarting a crashed server by hand.
-   Applying database schema changes manually.
-   Copy-pasting monitoring alerts into a ticket.

### Why Toil is Harmful

-   **Burnout:** High levels of toil lead to engineer burnout and low morale.
-   **Slows Innovation:** Engineers spending time on toil are not spending time on feature development or improvements.
-   **Increased Risk:** Manual processes are prone to human error, which can cause outages.
-   **Scalability Limits:** You can't hire your way out of a problem that scales linearly with your service.

### Strategies for Toil Reduction

The goal for an SRE team is to keep toil to less than 50% of their time. The rest should be spent on engineering projects that reduce future toil or add service features.

1.  **Measure Everything:**
    -   Track the time spent on operational tasks. Categorize work as either "toil" or "engineering."
    -   Use this data to identify the biggest sources of toil and prioritize what to automate first.

2.  **Automate, Automate, Automate:**
    -   Write scripts and tools to automate repetitive tasks.
    -   Invest in configuration management (e.g., Ansible, Puppet) and Infrastructure as Code (e.g., Terraform, CloudFormation) to automate infrastructure management.

3.  **Improve Documentation and Runbooks:**
    -   While a task is still manual, ensure it's well-documented in a runbook.
    -   A good runbook is a stepping stone to automation. If you can write down the steps, you can often script them.

4.  **Build Self-Service Tools:**
    -   Empower other teams (like developers or customer support) to perform common tasks safely through self-service tools. This reduces the interrupt load on the SRE team.
    -   *Example:* A web portal for developers to create test environments.

5.  **Set an "Error Budget" for Toil:**
    -   Just as you have an error budget for reliability, you can have a "toil budget." If a team exceeds its toil budget for a quarter, the next quarter's priority must be reducing that toil.
