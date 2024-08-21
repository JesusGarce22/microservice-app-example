FROM python:3.6-alpine

# Instala herramientas de compilación necesarias
RUN apk add --no-cache build-base gcc

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 6379

ENV REDIS_HOST=redis
ENV REDIS_PORT=6379
ENV REDIS_CHANNEL=log_channel

CMD ["python3", "main.py"]
