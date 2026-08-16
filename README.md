<div align="center">

# Online Boutique

### A Cloud-Native Microservices Demo Application

A full e-commerce store built as **11 polyglot microservices** that talk to each
other over **gRPC** — built to deploy on **AWS EKS**.

[![Go](https://img.shields.io/badge/Go-00ADD8?logo=go&logoColor=white)](https://go.dev)
[![Node.js](https://img.shields.io/badge/Node.js-339933?logo=nodedotjs&logoColor=white)](https://nodejs.org)
[![Python](https://img.shields.io/badge/Python-3776AB?logo=python&logoColor=white)](https://python.org)
[![Java](https://img.shields.io/badge/Java-ED8B00?logo=openjdk&logoColor=white)](https://openjdk.org)
[![.NET](https://img.shields.io/badge/.NET-512BD4?logo=dotnet&logoColor=white)](https://dotnet.microsoft.com)
[![gRPC](https://img.shields.io/badge/gRPC-244c5a?logo=grpc&logoColor=white)](https://grpc.io)
[![EKS](https://img.shields.io/badge/AWS-EKS-232F3E?logo=amazoneks&logoColor=white)](https://aws.amazon.com/eks)

</div>

---

## About

Online Boutique is a web-based e-commerce demo where users can **browse
products**, **add items to a cart**, and **place orders** end-to-end. Each
microservice is written in a different language, runs in its own container, and
communicates with the others over gRPC.

This repository contains the **application source code and dependency files
only**. It is ready to be deployed to an **AWS EKS cluster** — no deployment or
infrastructure files are included.

## Architecture

![Online Boutique architecture](docs/img/architecture-diagram.png)

### Services at a glance

| Service | Language | Port | Description |
| --- | --- | --- | --- |
| frontend | Go | 8080 | HTTP server that serves the website |
| cartservice | C# (.NET) | 7070 | Stores the shopping cart (Redis / in-memory) |
| productcatalogservice | Go | 3550 | Product catalog, search & retrieval |
| currencyservice | Node.js | 7000 | Currency conversion |
| paymentservice | Node.js | 50051 | Mock credit card charge |
| shippingservice | Go | 50052 | Shipping cost estimates |
| emailservice | Python | 5000 | Mock order confirmation emails |
| checkoutservice | Go | 5050 | Orchestrates payment, shipping & email |
| recommendationservice | Python | 8081 | Product recommendations |
| adservice | Java | 9555 | Context-based text ads |
| loadgenerator | Python/Locust | 8089 | Simulates realistic user traffic (optional) |

> **Note on ports:** `paymentservice` and `shippingservice` both default to
> `50051`, and `frontend` & `recommendationservice` both default to `8080`.
> When running everything on one machine the ports above avoid the conflicts.

## Getting Started

### Prerequisites

| Tool | Version | Used by |
| --- | --- | --- |
| [Go](https://go.dev/dl/) | 1.25+ | frontend, checkout, product catalog, shipping |
| [Node.js](https://nodejs.org/) | 18+ | currency, payment |
| [Python](https://python.org/downloads/) | 3.9+ | email, recommendations, loadgenerator |
| [Java](https://openjdk.org/) | 21+ | ads |
| [.NET SDK](https://dotnet.microsoft.com/download) | 10+ | cart |
| Redis *(optional)* | any | cart (falls back to in-memory) |

### Running the application locally

**Step 1 — Clone the repository**

```sh
git clone https://github.com/lokeshzenbook-coder/Online-Boutique.git
cd Online-Boutique
```

**Step 2 — Start the backend services**

Open one terminal per service and run:

<details>
<summary><b>currencyservice</b> <i>(Node.js · port 7000)</i></summary>

```sh
cd src/currencyservice
npm install
PORT=7000 node server.js
```

</details>

<details>
<summary><b>paymentservice</b> <i>(Node.js · port 50051)</i></summary>

```sh
cd src/paymentservice
npm install
PORT=50051 node index.js
```

</details>

<details>
<summary><b>productcatalogservice</b> <i>(Go · port 3550)</i></summary>

```sh
cd src/productcatalogservice
go run .
```

</details>

<details>
<summary><b>shippingservice</b> <i>(Go · port 50052)</i></summary>

```sh
cd src/shippingservice
PORT=50052 go run .
```

</details>

<details>
<summary><b>emailservice</b> <i>(Python · port 5000)</i></summary>

```sh
cd src/emailservice
pip install -r requirements.txt
python email_server.py
```

</details>

<details>
<summary><b>adservice</b> <i>(Java · port 9555)</i></summary>

```sh
cd src/adservice
./gradlew run
```

</details>

<details>
<summary><b>cartservice</b> <i>(C# · port 7070)</i></summary>

With Redis (recommended):

```sh
docker run -d --name redis-cart -p 6379:6379 redis
cd src/cartservice/src
REDIS_ADDR=localhost:6379 ASPNETCORE_URLS=http://localhost:7070 dotnet run
```

Without Redis (uses an in-memory store):

```sh
cd src/cartservice/src
ASPNETCORE_URLS=http://localhost:7070 dotnet run
```

</details>

<details>
<summary><b>recommendationservice</b> <i>(Python · port 8081)</i></summary>

```sh
cd src/recommendationservice
pip install -r requirements.txt
PORT=8081 python recommendation_server.py
```

</details>

<details>
<summary><b>checkoutservice</b> <i>(Go · port 5050)</i></summary>

```sh
cd src/checkoutservice
export CART_SERVICE_ADDR=localhost:7070
export CURRENCY_SERVICE_ADDR=localhost:7000
export PRODUCT_CATALOG_SERVICE_ADDR=localhost:3550
export SHIPPING_SERVICE_ADDR=localhost:50052
export EMAIL_SERVICE_ADDR=localhost:5000
export PAYMENT_SERVICE_ADDR=localhost:50051
go run .
```

</details>

**Step 3 — Start the frontend**

```sh
cd src/frontend
export PORT=8080
export ENV_PLATFORM=local
export PRODUCT_CATALOG_SERVICE_ADDR=localhost:3550
export CURRENCY_SERVICE_ADDR=localhost:7000
export CART_SERVICE_ADDR=localhost:7070
export RECOMMENDATION_SERVICE_ADDR=localhost:8081
export SHIPPING_SERVICE_ADDR=localhost:50052
export CHECKOUT_SERVICE_ADDR=localhost:5050
export AD_SERVICE_ADDR=localhost:9555
export SHOPPING_ASSISTANT_SERVICE_ADDR=localhost:80
go run .
```

**Step 4 — Open the app**

Visit **[http://localhost:8080](http://localhost:8080)** in your browser.

### (Optional) Generate traffic with the load generator

```sh
cd src/loadgenerator
pip install -r requirements.txt
locust --host http://localhost:8080
```

Open the Locust web UI at [http://localhost:8089](http://localhost:8089),
enter a user count, and watch your microservices get busy.

---

## Deploying to AWS EKS

The source is designed to deploy on **Amazon EKS**. Build and push each service
image to a container registry, then provision resources in your cluster.

Key deployment notes:

- Set the frontend platform flag to `aws`:

  ```yaml
  env:
    - name: ENV_PLATFORM
      value: "aws"
  ```

- `cartservice` requires a **Redis** service named `redis-cart` on port `6379`.
- Services are discovered over gRPC using their Kubernetes DNS names (e.g.
  `productcatalogservice:3550`), so each service needs a `Service` resource
  matching its name.

## Directory Layout

```
.
├── protos/                    # Protocol Buffers definitions (shared gRPC contracts)
│   └── demo.proto
└── src/                       # Microservice source code
    ├── adservice/             # Java (Gradle)
    ├── cartservice/           # C# (.NET)
    ├── checkoutservice/       # Go
    ├── currencyservice/       # Node.js
    ├── emailservice/          # Python
    ├── frontend/              # Go
    ├── loadgenerator/         # Python / Locust
    ├── paymentservice/        # Node.js
    ├── productcatalogservice/ # Go
    ├── recommendationservice/ # Python
    ├── shippingservice/       # Go
    └── shoppingassistantservice  # Python
```

Each service directory contains its source files and dependency manifests
(`go.mod`, `package.json`, `requirements.txt`, `build.gradle`, `.csproj`, etc.).

---

<div align="center">

*Built for learning cloud-native microservices, distributed systems, and gRPC.*

</div>
