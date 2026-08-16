# Online Boutique — Microservices Demo

A cloud-first microservices demo application: a web-based e-commerce store where
users can browse items, add them to a cart, and place orders. The application is
made up of 11 microservices written in different languages that communicate with
each other over gRPC.

This project contains the application **source code and dependency files only**.
It is intended to be deployed to an **AWS EKS cluster**. No deployment or
infrastructure files (Kubernetes manifests, Helm charts, Terraform, CI/CD, etc.)
are included in this repository — add your own EKS deployment files.

## Architecture

| Service | Language | Description |
| --- | --- | --- |
| frontend | Go | HTTP server that serves the website |
| cartservice | C# (.NET) | Stores the shopping cart in Redis |
| productcatalogservice | Go | Product list, search, and retrieval from a JSON file |
| currencyservice | Node.js | Currency conversion |
| paymentservice | Node.js | Mock credit card charge |
| shippingservice | Go | Shipping cost estimates |
| emailservice | Python | Mock order confirmation emails |
| checkoutservice | Go | Orchestrates payment, shipping, and email |
| recommendationservice | Python | Product recommendations |
| adservice | Java | Context-based text ads |
| loadgenerator | Python/Locust | Simulates realistic user traffic |

## Directory layout

```
.
├── protos/          # Protocol Buffers definitions (shared gRPC contracts)
└── src/             # Microservice source code
    ├── adservice/               # Java (Gradle)
    ├── cartservice/             # C# (.NET)
    ├── checkoutservice/         # Go
    ├── currencyservice/         # Node.js
    ├── emailservice/            # Python
    ├── frontend/                # Go
    ├── loadgenerator/           # Python
    ├── paymentservice/          # Node.js
    ├── productcatalogservice/   # Go
    ├── recommendationservice/   # Python
    ├── shippingservice/         # Go
    └── shoppingassistantservice # Python
```

Each service directory contains its source files and dependency manifests:

- Go: `go.mod` / `go.sum`
- Node.js: `package.json` / `package-lock.json`
- Python: `requirements.txt`
- Java: `build.gradle` + Gradle wrapper
- C#: `.sln` / `.csproj`

## Building

Every service includes a `Dockerfile` for building its container image. Build
and push each image to your own container registry, then deploy to EKS.

## Deployment to AWS EKS

The frontend service supports the AWS platform. Set the environment variable
for the frontend deployment:

```yaml
env:
  - name: ENV_PLATFORM
    value: "aws"
```

The `cartservice` depends on a **Redis** instance (service name `redis-cart`,
port `6379`) — provision one as part of your EKS deployment.

Services are discovered over gRPC using their Kubernetes DNS names (e.g.
`productcatalogservice:3550`), so each service needs a `Service` resource
matching its name. See each service's source for the env-var-based endpoint
configuration.
