# Build stage
FROM node:20-alpine AS builder

# Required by some dependencies' post-install scripts
RUN apk add --no-cache python3 make g++

WORKDIR /app
COPY package*.json ./
RUN npm install --omit=dev

# Runtime stage
FROM node:20-alpine

WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .

EXPOSE 50051

CMD ["node", "index.js"]
