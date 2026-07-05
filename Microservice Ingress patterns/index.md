# Microservice Ingress patterns
- [What are Microservices? (The Kitchen Analogy)](#what-are-microservices-the-kitchen-analogy)
- [The Ingress Problem: Who Opens the Door?](#the-ingress-problem-who-opens-the-door)
- [The Front Doors: Gateway and BFF](#the-front-doors-gateway-and-bff)
  - [The Centralized API Gateway (The Host at the Door)](#the-centralized-api-gateway-the-host-at-the-door)
  - [The Backend for Frontend (BFF) (The Special Menus)](#the-backend-for-frontend-bff-the-special-menus)
- [Other Ways to Route Traffic](#other-ways-to-route-traffic)
  - [The Two-Tier Gateway (The Security Guard and The Host)](#the-two-tier-gateway-the-security-guard-and-the-host)
  - [The Microgateway (Separate Team Doors)](#the-microgateway-separate-team-doors)
  - [Data Federation (The Master Menu and GraphQL)](#data-federation-the-master-menu-and-graphql)
  - [Service Mesh Ingress (The All-In-One Neighborhood)](#service-mesh-ingress-the-all-in-one-neighborhood)
  - [Asynchronous Ingress (The Order Drop-Box)](#asynchronous-ingress-the-order-drop-box)
  - [Direct Client-to-Service (No Door)](#direct-client-to-service-no-door)
- [Summary Table](#summary-table)
- [Sources](#sources)

## What are Microservices? (The Kitchen Analogy)

Imagine a giant restaurant kitchen.
* In a **Monolith**, one single chef cooks the soup, flips the burgers, and bakes the cake all by themselves, If the chef gets sick, the restaurant closes!
* In **Microservices**, we have many small, independent chefs. One chef *only* makes soup. Another *only* bakes cakes, They work faster, and if the cake baker drops a tray, the soup chef keeps cooking (implementing the single responsibility aspect from S.O.L.I.D).

## The Ingress Problem: Who Opens the Door?

"Ingress" just means **"how a guest enters the restaurant."**

If hungry customers walk right into the kitchen to grab food from different chefs, it becomes a messy disaster.
We need a system at the front door to take orders and pass them to the right chef.

## The Front Doors: Gateway and BFF

### The Centralized API Gateway (The Host at the Door)
Think of this as a single host standing at the front door.
Every customer talks to this one host.
The host checks your ID, takes your order, runs to the kitchen to grab your food, and brings it back to you.
* **Good:** Simple for the customer, Safe for the kitchen.
* **Bad:** If the host gets tired, everyone waits in a long line.

### The Backend for Frontend (BFF) (The Special Menus)
Instead of one host for everyone, we create different hosts for different types of customers.
* A **Phone Host** helps people ordering on mobile phones (gives them small, quick meals).
* A **Computer Host** helps people sitting down with laptops (gives them big, detailed meals).
* **Good:** Everyone gets exactly what it needs.

## Other Ways to Route Traffic

### The Two-Tier Gateway (The Security Guard and The Host)
You pass through two separate doors.
* **Tier 1 (The Edge):** A fast global proxy (like **Cloudflare** or **AWS CloudFront**) acts as the outside security guard.
It handles **Authentication**—checking *who* you are by validating an **OIDC** token or handling **TLS Termination** (locking the network door).
* **Tier 2 (The Hub):** Inside, an internal gateway (like **Kong** or **Apisix**) handles **Authorization**—checking *what* you are allowed to do and routing you to the right room.
* **Why use it?** It keeps the outside security separate from inside application rules.

### The Microgateway (Separate Team Doors)
Instead of one massive front door for the whole company, every team gets their own small, lightweight gateway instance (using tools like **Envoy** or **Spring Cloud Gateway**).
* **How it works:** The main router looks at the URL path (like `/orders/`) and instantly passes you to the Orders Team's private gateway.
That team manages their own security rules without bothering anyone else.
* **Why use it?** Teams don't have to wait in line to change shared configuration files.

### Data Federation (The Master Menu and GraphQL)
Instead of asking three different chefs for your burger, fries, and drink, you ask a single smart engine called a **GraphQL Federated Gateway** (like **Apollo Router**).
* **How it works:** The gateway reads one complex request from the frontend, automatically splits it into sub-requests, fetches the data from different services in parallel, and merges everything into a single, clean JSON response.
* **Why use it?** Bypasses the need to write custom aggregation code in multiple BFFs.

### Service Mesh Ingress (The All-In-One Neighborhood)
When all your internal microservices talk to each other using a **Service Mesh** (like **Istio** or **Linkerd**), you can use a native Mesh Ingress Controller at the perimeter.
* **How it works:** The moment a request hits the edge, it immediately gets wrapped in **mTLS (Mutual TLS)**.
This ensures encrypted, secure, and fully tracked end-to-end communication from the outside door all the way to the deepest database.
* **Why use it?** You get perfect tracking, tracing, and network security across the whole system using one unified configuration tool.

### Asynchronous Ingress (The Order Drop-Box)
You don't wait around for a response.
The ingress point is an event broker or ingestion proxy (like a **Kafka REST Proxy** or **AWS SQS**).
* **How it works:** The user sends data, the gateway instantly replies with a `202 Accepted` status code, and drops the payload into a message queue.
Downstream microservices process the data later at their own pace.
* **Why use it?** Perfect for high-throughput traffic shocks (like IoT sensors or clickstream data) because the queue handles the heavy pressure.

### Direct Client-to-Service (No Door)
The client application completely bypasses any gateway or middleman and talks straight to public-facing microservice endpoints over the internet.
* **How it works:** Every single service gets its own public cloud Load Balancer and public DNS record.
* **Why use it?** It removes network hops for absolute lowest latency, but it means *every single microservice* must independently implement its own security, CORS headers, and rate limiting.

## Summary Table

| Pattern | How it works | Best for... |
| :--- | :--- | :--- |
| **API Gateway** | One host handles everyone | Simple web apps |
| **BFF** | Different hosts for phones vs. computers | Apps with mobile and web versions |
| **Two-Tier** | Security guard first, host second | Big companies that need high safety |
| **Microgateway** | Every team gets their own door | Large teams who want independence |
| **Data Federation**| One screen mixes data from everywhere | Complex apps with lots of data |
| **Async Ingress** | Drop the order in a box and leave | Sending lots of data very fast |
| **Direct Client** | Walk right up to the chef | Apps that need to be ultra-fast |

## Sources

- [Gateway](https://en.wikipedia.org/wiki/API_management)
- [BFF](https://en.wikipedia.org/wiki/Front_end_and_back_end)
