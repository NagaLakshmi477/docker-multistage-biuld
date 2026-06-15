
FROM python:3.9 AS builder
WORKDIR /opt/server
COPY requirements.txt .
COPY *.py .
COPY payment.ini .
RUN pip3 install -r requirements.txt

FROM python:3.9-alpine
EXPOSE 8080
RUN addgroup -S roboshop && adduser -S -G roboshop roboshop
WORKDIR /opt/server
COPY --from=builder /opt/server /opt/server
RUN apk add python3-dev build-base linux-headers pcre-dev
USER roboshop 
CMD ["uwsgi", "--ini", "payment.ini"]
