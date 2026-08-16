<p align="center">
  <img src="docs/img/logo.svg" width="320" alt="Online Boutique" />
</p>

<h1 align="center">Online Boutique</h1>

<p align="center">
  A cloud-native e-commerce demo built as <b>11 polyglot microservices</b>
  communicating over <b>gRPC</b> — ready to deploy on <b>AWS EKS</b> or spin up
  locally with <b>Docker Compose</b>.
</p>

<p align="center">
  <a href="https://go.dev"><img alt="Go" src="https://img.shields.io/badge/Go-00ADD8?style=flat-square&logo=go&logoColor=white"></a>
  <a href="https://nodejs.org"><img alt="Node.js" src="https://img.shields.io/badge/Node.js-339933?style=flat-square&logo=nodedotjs&logoColor=white"></a>
  <a href="https://python.org"><img alt="Python" src="https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white"></a>
  <a href="https://openjdk.org"><img alt="Java" src="https://img.shields.io/badge/Java-ED8B00?style=flat-square&logo=openjdk&logoColor=white"></a>
  <a href="https://dotnet.microsoft.com"><img alt=".NET" src="https://img.shields.io/badge/.NET-512BD4?style=flat-square&logo=dotnet&logoColor=white"></a>
  <a href="https://grpc.io"><img alt="gRPC" src="https://img.shields.io/badge/gRPC-244c5a?style=flat-square&logo=grpc&logoColor=white"></a>
  <a href="https://docs.docker.com/compose/"><img alt="Docker Compose" src="https://img.shields.io/badge/Docker%20Compose-2496ED?style=flat-square&logo=docker&logoColor=white"></a>
  <a href="https://aws.amazon.com/eks/"><img alt="AWS EKS" src="https://img.shields.io/badge/AWS%20EKS-232F3E?style=flat-square&logo=amazoneks&logoColor=white"></a>
</p>

---

## What is this?

**Online Boutique** is a working e-commerce storefront where users can browse
products, manage a cart, and place orders from start to finish. It's a learning
project that demonstrates:

- A **polyglot microservice architecture** — Go, Node.js, Python, Java & .NET
- **Synchronous gRPC** communication between services
- **Containerized** services, each with its own `Dockerfile`
- Ready for orchestration on **Kubernetes / AWS EKS**

> This repository ships the application **source code, dependency manifests and
> Dockerfiles**. It is kept free of deployment/infrastructure files so you can
> add your own.

## Features

- **Browse & search** a product catalog
- **Multi-currency** pricing, with live conversion
- **Shopping cart** backed by Redis
- **Full checkout flow** — payment, shipping & email confirmation (all mocked)
- **Product recommendations** & contextual ads
- **Load testing** built in with Locust
- **Single-command startup** with Docker Compose

## Architecture

<p align="center">
  <img src="docs/img/architecture-diagram.png" alt="Online Boutique architecture" width="720" />
</p>

### Services at a glance

| Service | Language | Port | Description |
| --- | --- | --- | --- |
| frontend | Go | 8080 | HTTP server that serves the website |
| cartservice | C# (.NET) | 7070 | Stores the shopping cart (Redis) |
| productcatalogservice | Go | 3550 | Product catalog, search & retrieval |
| currencyservice | Node.js | 7000 | Currency conversion |
| paymentservice | Node.js | 50051 | Mock credit card charge |
| shippingservice | Go | 50051 | Shipping cost estimates |
| emailservice | Python | 5000 | Mock order confirmation emails |
| checkoutservice | Go | 5050 | Orchestrates payment, shipping & email |
| recommendationservice | Python | 8080 | Product recommendations |
| adservice | Java | 9555 | Context-based text ads |
| loadgenerator | Python/Locust | 8089 | Simulates realistic user traffic (optional) |

> **Note on ports:** with Docker Compose every service runs in its own
> container on the same network, so `paymentservice` and `shippingservice` can
> both listen on `50051`, and `frontend` & `recommendationservice` on `8080`
> without conflict. Only `frontend` (and the optional `loadgenerator` UI) are
> published to the host.

## Getting Started

### Prerequisites

| Tool | Version | Notes |
| --- | --- | --- |
| [Docker](https://docs.docker.com/get-docker/) | 24+ | Container runtime |
| [Docker Compose](https://docs.docker.com/compose/) | v2 | Included with Docker Desktop / Docker Engine plugin |
| [git](https://git-scm.com/) | any | To clone the repository |

> Every service ships with a `Dockerfile` in its own source directory, so the
> compose file below can build all images from scratch.

### Running everything with Docker Compose

Spin up the **entire application — all 11 microservices plus Redis — with a
single command.**

**Step 1 — Clone the repository**

```sh
git clone https://github.com/lokeshzenbook-coder/Online-Boutique.git
cd Online-Boutique
```

**Step 2 — Create a `docker-compose.yml`**

Create a file named `docker-compose.yml` in the project root with this content:

```yaml
version: "3.9"

services:
  redis-cart:
    image: redis:7-alpine
    restart: always

  adservice:
    build: ./src/adservice
    environment:
      - PORT=9555
    restart: always

  cartservice:
    build: ./src/cartservice/src
    environment:
      - REDIS_ADDR=redis-cart:6379
      - ASPNETCORE_URLS=http://0.0.0.0:7070
    depends_on:
      - redis-cart
    restart: always

  checkoutservice:
    build: ./src/checkoutservice
    environment:
      - PORT=5050
      - CART_SERVICE_ADDR=cartservice:7070
      - CURRENCY_SERVICE_ADDR=currencyservice:7000
      - PRODUCT_CATALOG_SERVICE_ADDR=productcatalogservice:3550
      - SHIPPING_SERVICE_ADDR=shippingservice:50051
      - EMAIL_SERVICE_ADDR=emailservice:5000
      - PAYMENT_SERVICE_ADDR=paymentservice:50051
    restart: always

  currencyservice:
    build: ./src/currencyservice
    environment:
      - PORT=7000
    restart: always

  emailservice:
    build: ./src/emailservice
    environment:
      - PORT=5000
    restart: always

  frontend:
    build: ./src/frontend
    environment:
      - PORT=8080
      - ENV_PLATFORM=local
      - PRODUCT_CATALOG_SERVICE_ADDR=productcatalogservice:3550
      - CURRENCY_SERVICE_ADDR=currencyservice:7000
      - CART_SERVICE_ADDR=cartservice:7070
      - RECOMMENDATION_SERVICE_ADDR=recommendationservice:8080
      - SHIPPING_SERVICE_ADDR=shippingservice:50051
      - CHECKOUT_SERVICE_ADDR=checkoutservice:5050
      - AD_SERVICE_ADDR=adservice:9555
      - SHOPPING_ASSISTANT_SERVICE_ADDR=shoppingassistantservice:80
    ports:
      - "8080:8080"
    depends_on:
      - productcatalogservice
      - currencyservice
      - cartservice
      - recommendationservice
      - shippingservice
      - checkoutservice
      - adservice
    restart: always

  paymentservice:
    build: ./src/paymentservice
    environment:
      - PORT=50051
    restart: always

  productcatalogservice:
    build: ./src/productcatalogservice
    environment:
      - PORT=3550
    restart: always

  recommendationservice:
    build: ./src/recommendationservice
    environment:
      - PORT=8080
    restart: always

  shippingservice:
    build: ./src/shippingservice
    environment:
      - PORT=50051
    restart: always

  loadgenerator:
    build: ./src/loadgenerator
    environment:
      - FRONTEND_ADDR=frontend:8080
    ports:
      - "8089:8089"
    depends_on:
      - frontend
    restart: always
```

**Step 3 — Start everything**

```sh
docker compose up --build -d
```

Compose builds and starts every service in the background. The first run takes
a few minutes while the images are built.

**Step 4 — Open the app**

Visit **[http://localhost:8080](http://localhost:8080)** in your browser.

**Step 5 — Check status / logs**

```sh
docker compose ps        # list running services
docker compose logs -f   # follow logs
```

**Step 6 — Stop everything**

```sh
docker compose down
```

Add `-v` to also remove the named volumes:

```sh
docker compose down -v
```

### (Optional) Load testing

The `loadgenerator` service simulates realistic shopping traffic against the
frontend. After `docker compose up`, open the Locust web UI at
[http://localhost:8089](http://localhost:8089), enter a user count, and watch
your microservices get busy.

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

## Tech Stack

| Layer | Technology |
| --- | --- |
| Frontend | Go + HTML templates |
| Backend (Go) | checkoutservice, productcatalogservice, shippingservice, frontend |
| Backend (Node.js) | currencyservice, paymentservice |
| Backend (Python) | emailservice, recommendationservice, loadgenerator |
| Backend (Java) | adservice |
| Backend (C# / .NET) | cartservice |
| Service communication | gRPC |
| Data store | Redis |
| Containerization | Docker |
| Orchestration target | AWS EKS / Kubernetes |
| Load testing | Locust |

## Directory Layout

```
.
├── protos/                    # Protocol Buffers definitions (shared gRPC contracts)
│   └── demo.proto
├── docs/
│   └── img/                   # README assets (logo, architecture diagram)
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

Each service directory contains its source files, dependency manifests
(`go.mod`, `package.json`, `requirements.txt`, `build.gradle`, `.csproj`, etc.),
and a `Dockerfile` for building its container image.

---

<p align="center">
  Built for learning <b>cloud-native microservices</b>, <b>distributed
  systems</b>, and <b>gRPC</b>.<br/>
  <i>Happy shopping!</i>
</p>
