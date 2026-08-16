# Build stage
FROM python:3.12-slim AS builder

COPY requirements.txt ./
RUN pip install --prefix=/install -r requirements.txt

# Runtime stage
FROM python:3.12-slim

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    GEVENT_SUPPORT=True

WORKDIR /loadgen
COPY --from=builder /install /usr/local
COPY locustfile.py .

EXPOSE 8089

CMD locust --host="http://${FRONTEND_ADDR}" -u "${USERS:-10}" -r "${RATE:-1}"
