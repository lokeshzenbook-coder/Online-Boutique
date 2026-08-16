# Build stage
FROM golang:1.25-alpine AS builder

WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o /productcatalogservice .

# Runtime stage
FROM gcr.io/distroless/static

WORKDIR /src
COPY --from=builder /productcatalogservice ./server
COPY products.json .

ENV GOTRACEBACK=single

EXPOSE 3550

ENTRYPOINT ["/src/server"]
