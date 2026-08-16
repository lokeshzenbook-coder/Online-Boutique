# Build stage
FROM golang:1.25-alpine AS builder

WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o /checkoutservice .

# Runtime stage
FROM gcr.io/distroless/static

COPY --from=builder /checkoutservice /checkoutservice

ENV GOTRACEBACK=single

EXPOSE 5050

ENTRYPOINT ["/checkoutservice"]
