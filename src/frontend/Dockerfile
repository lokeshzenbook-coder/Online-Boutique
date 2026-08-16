# Build stage
FROM golang:1.25-alpine AS builder

WORKDIR /app

COPY go.mod go.sum ./
RUN go mod download

COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-s -w" -o /frontend .

# Runtime stage
FROM gcr.io/distroless/static

WORKDIR /src
COPY --from=builder /frontend /src/server
COPY templates ./templates
COPY static ./static

ENV GOTRACEBACK=single

EXPOSE 8080

ENTRYPOINT ["/src/server"]
