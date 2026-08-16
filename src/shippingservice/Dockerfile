# Build stage
FROM golang:1.25-alpine AS builder

WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o /shippingservice .

# Runtime stage
FROM gcr.io/distroless/static

WORKDIR /src
COPY --from=builder /shippingservice /src/shippingservice

ENV GOTRACEBACK=single

EXPOSE 50051

ENTRYPOINT ["/src/shippingservice"]
