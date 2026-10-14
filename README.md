# CST8915 – Full-stack Cloud-native Development

## Midterm Project: Extending the Algonquin Pet Store with an Order Analytics Microservice

| | |
|---|---|
| **Semester** | Fall 2026 |
| **Release Week** | Week 6 (week beginning October 12, 2026) |
| **Due Date** | Week 7 (week beginning October 19, 2026); see Brightspace for the exact date and time. All times are Eastern Time (America/Toronto). |
| **Weight** | 15% of final grade |
| **Type** | Group Assignment (3 students) |
| **Submission** | Brightspace |

---

## Overview

In this group assignment, you will extend the **Algonquin Pet Store** application by building a new microservice: the **Order Analytics Service**. This service will **consume order messages from RabbitMQ** and provide analytics through a REST API.

Currently, the `order-service` publishes orders to a RabbitMQ queue (`order_queue`), but no service consumes those messages — the orders go into the queue and sit there. Your task is to complete this architectural gap by building a service that listens to the queue, processes the orders, and exposes useful analytics endpoints.

You will test the full application **locally** and then deploy all services to **Azure** using **Azure App Service** (for backend services) and **Azure Static Web Apps** (for the store-front), following the same patterns you learned in Labs 2 and 3.

---

## Services

### Existing Services (provided)

| Service | Language | Port | Role |
|---------|----------|------|------|
| **store-front** | Vue.js | 8080 (dev) / 80 (prod) | Customer-facing web application |
| **order-service** | Node.js | 3000 | Receives orders via REST, publishes to RabbitMQ |
| **product-service** | Python | 3030 | Serves product catalog via REST |
| **RabbitMQ** | — | 5672 / 15672 | Message broker (backing service) |

### New Service (you build this)

| Service | Language | Port | Role |
|---------|----------|------|------|
| **order-analytics-service** | Your choice | 4000 | Consumes orders from RabbitMQ, provides analytics API |

---

## RabbitMQ Message Format

The `order-service` publishes messages to a queue named `order_queue` using the default exchange. Each message is a JSON object with the following structure:

```json
{
  "product": {
    "id": 1,
    "name": "Cat Food",
    "price": 9.99
  },
  "quantity": 2,
  "totalPrice": 19.98
}
```

**Queue details:**
- **Queue name:** `order_queue`
- **Durable:** `true`
- **Exchange:** Default (direct to queue via `sendToQueue()`)

Use the updated `order-service` from Labs 2 and 3 as your starting point. Keep its durable queue declaration; your consumer must use matching queue settings.

---

## Requirements

### Part 1: Build the Order Analytics Service

Build a new microservice that connects to RabbitMQ, consumes orders from `order_queue`, and exposes the following REST API endpoints:

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/health` | Health check — returns service status |
| `GET` | `/orders` | List all consumed orders |
| `GET` | `/analytics/summary` | Total orders, total revenue, and average order value |
| `GET` | `/analytics/products` | Orders grouped by product name with count and revenue |
| `GET` | `/analytics/top-products` | Top 3 products ranked by total quantity sold |

**Expected response examples:**

`GET /analytics/summary`
```json
{
  "totalOrders": 47,
  "totalRevenue": 892.53,
  "averageOrderValue": 18.99
}
```

`GET /analytics/products`
```json
[
  { "productName": "Cat Food", "orderCount": 20, "totalRevenue": 199.80 },
  { "productName": "Dog Leash", "orderCount": 15, "totalRevenue": 374.85 },
  { "productName": "Fish Tank", "orderCount": 12, "totalRevenue": 317.88 }
]
```

`GET /analytics/top-products`
```json
[
  { "productName": "Cat Food", "totalQuantity": 42 },
  { "productName": "Dog Leash", "totalQuantity": 31 },
  { "productName": "Fish Tank", "totalQuantity": 18 }
]
```

#### Technology Choice

You may implement the service in any programming language of your choice. However, keep in mind that the service must be deployable to **Azure App Service**.

#### Storage and Restart Expectations

**In-memory storage is acceptable.** When using in-memory storage, `/orders` and the analytics endpoints cover orders processed since the service last started, and their data resets after a restart. Persistence across restarts is not required.

Use **one application instance and one consumer process** when using in-memory storage so all orders and analytics are held in the same process. Once messages are acknowledged, they are removed from RabbitMQ and cannot be replayed from the queue to rebuild lost in-memory data.

#### RabbitMQ Consumer Pattern

Your service must connect to RabbitMQ and continuously listen for messages. It is up to your group to decide **how** the service consumes messages — research the available RabbitMQ messaging patterns and choose the one that best fits your design. Be prepared to justify your choice in your report.

#### 12-Factor App Compliance

Your new service must follow the first four factors of the 12-Factor App methodology:

| Factor | Requirement |
|--------|-------------|
| **I. Codebase** | Own GitHub repository for the order-analytics-service |
| **II. Dependencies** | All dependencies declared in a manifest file (`package.json`, `requirements.txt`, etc.) |
| **III. Config** | RabbitMQ connection string and port configured via environment variables / `.env` file |
| **IV. Backing Services** | RabbitMQ treated as an attached resource via connection string |

---

### Part 2: Test Locally

Before deploying to Azure, verify the **entire application** works on your local machine (or a VM):

1. Start RabbitMQ
2. Start all four services: `product-service`, `order-service`, `store-front`, and your new `order-analytics-service`
3. Place several orders through the `store-front`
4. Verify that the `order-analytics-service` consumes the orders and the analytics endpoints return correct data

---

### Part 3: Deploy to Azure

Deploy the complete Algonquin Pet Store (including your new service) to Azure using the following architecture:

| Service | Azure Service | Notes |
|---------|--------------|-------|
| **RabbitMQ** | Azure VM | Same setup as Lab 2/3 |
| **order-service** | Azure App Service | Node.js runtime |
| **product-service** | Azure App Service | Python runtime (or your chosen language) |
| **order-analytics-service** | Azure App Service | Your chosen runtime |
| **store-front** | Azure Static Web Apps | With GitHub Actions CI/CD |

**Requirements:**
- All services must be properly configured and communicate correctly once deployed
- Demonstrate the full workflow: place orders via the deployed `store-front` and verify analytics via the deployed `order-analytics-service`
- A working Azure demonstration is required; continuous operation between demonstrations is not required. You may start or warm up the analytics service before recording your demo.

---

## Video Presentation

Each group must prepare **a short video presentation (5 to 10 minutes)** covering:

1. A brief overview of your order-analytics-service (language choice, architecture decisions)
2. The RabbitMQ consumer pattern you chose and why
3. A live demo of the application running **locally** — place orders, then show analytics endpoints
4. A live demo of the application running on **Azure** — show all deployed services, place orders, and verify analytics
5. A walkthrough of your architecture diagram

| Requirement | Details |
|-------------|---------|
| **Length** | 5 to 10 minutes |
| **Participation** | All group members must participate and present |
| **Upload** | YouTube (unlisted) |
| **Link** | Include the video link in your `README.md` file on GitHub |
| **Recording Tool** | [OBS Studio](https://obsproject.com/) (free, cross-platform) |

---

## Group Work Guidelines

- Each group consists of **3 students**
- Groups will be created in Brightspace and students will be **randomly assigned**
- Each group is responsible for dividing tasks equally and collaborating to complete the assignment
- Your submission must reflect the collective effort and understanding of all group members
- **Include a section** in your submission describing each member's contributions

---

## Submission Instructions

**Deliverables:** Source code, deployment evidence, architecture diagram, video link, and contribution statement.

**GitHub Repository:**
1. Create a **private** GitHub repository for your group
2. Add `ramymohamed10` as a collaborator (Settings → Collaborators → Add people)
3. Your repository must include:
   - Source code for the `order-analytics-service`
   - A `README.md` file with:
     - YouTube demo video link
     - Architecture diagram (showing all services and how they communicate)
     - Links to all service repositories (`order-service`, `product-service`, `store-front`, `order-analytics-service`)
     - Your RabbitMQ consumer pattern choice and justification
     - Each group member's contributions
   - (Optional) Notes about setup challenges or lessons learned

> **Reference:** [Inviting collaborators to a personal repository](https://docs.github.com/en/enterprise-server@3.10/account-and-profile/setting-up-and-managing-your-personal-account-on-github/managing-access-to-your-personal-repositories/inviting-collaborators-to-a-personal-repository)

Submit your **GitHub Repository URL** on Brightspace by the due date.

> **Warning:** It is your responsibility to ensure the instructor is added as a collaborator before the deadline. Submissions without collaborator access at the time of marking will not be accepted.

---

## Marking Rubric

| Criteria | Weight | Description |
|----------|:------:|-------------|
| **Order Analytics Service Implementation** | 25% | Service correctly consumes from RabbitMQ and all REST API endpoints return accurate data |
| **12-Factor App Compliance** | 15% | Separate repository, explicit dependencies, environment-based configuration, RabbitMQ as attached resource |
| **Local Testing** | 15% | Full application runs locally with all services communicating correctly |
| **Azure Deployment** | 25% | All services deployed and working on Azure (App Service + Static Web Apps + VM for RabbitMQ) |
| **Video Presentation** | 20% | Clear explanation, live demos (local + Azure), architecture walkthrough, and equal participation |
| | **100%** | |

---

## Academic Integrity

This assignment **permits** the use of generative AI tools (e.g., ChatGPT, GitHub Copilot, Claude Code) to assist with coding, research, brainstorming, or debugging, provided that:

1. **Transparency** — You disclose how AI was used and cite it properly in your report
2. **Original Thought** — The work primarily reflects your own understanding and effort
3. **Critical Evaluation** — You verify the accuracy and relevance of AI-generated content

Directly submitting AI-generated work without modification, evaluation, or citation violates Algonquin College's Academic Integrity Policy AA48 and could result in a zero or further disciplinary action.

Collaboration **within** your group is expected. Collaboration **between** groups is not permitted.

---

## Late Submission Policy

In accordance with the course late-assignment policy, late submissions receive a deduction of **10% per day** and a **zero after 5 days**.

The repository version available at the time of your Brightspace submission will be marked. Subsequent repository updates must be resubmitted through Brightspace; resubmissions after the deadline are subject to the same late-submission policy.

---

## Questions?

Email: mohamer@algonquincollege.com
