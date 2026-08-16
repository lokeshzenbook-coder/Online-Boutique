# Build stage
FROM python:3.12-slim AS builder

RUN apt-get update \
    && apt-get install -y --no-install-recommends g++ \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt ./
RUN pip install --prefix=/install -r requirements.txt

# Runtime stage
FROM python:3.12-slim

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

WORKDIR /recommendationservice
COPY --from=builder /install /usr/local
COPY . .

ENV PORT=8080

EXPOSE 8080

ENTRYPOINT ["python", "recommendation_server.py"]
