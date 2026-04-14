FROM python:3.13-alpine AS builder
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1

RUN apk add --no-cache g++ linux-headers

COPY src/loadgenerator/requirements.txt .
RUN pip install --prefix="/install" -r requirements.txt

FROM python:3.13-alpine
ENV PYTHONDONTWRITEBYTECODE=1
ENV PYTHONUNBUFFERED=1
ENV GEVENT_SUPPORT=True

RUN apk add --no-cache libstdc++

WORKDIR /loadgen
COPY --from=builder /install /usr/local
COPY src/loadgenerator/locustfile.py .

ENTRYPOINT locust --host="http://${FRONTEND_ADDR}" --headless -u "${USERS:-10}" -r "${RATE:-1}" 2>&1
