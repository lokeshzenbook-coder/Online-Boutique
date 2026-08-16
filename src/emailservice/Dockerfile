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

WORKDIR /email_server
COPY --from=builder /install /usr/local
COPY . .

EXPOSE 5000

ENTRYPOINT ["python", "email_server.py"]
