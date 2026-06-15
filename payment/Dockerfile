
FROM python:3.9 AS builder
WORKDIR /opt/server
COPY requirements.txt .
RUN pip install --prefix=/install -r requirements.txt
COPY *.py .
COPY payment.ini .

FROM python:3.9-alpine
EXPOSE 8080
RUN addgroup -S roboshop && adduser -S -G roboshop roboshop
WORKDIR /opt/server
COPY --from=builder /install /usr/local
COPY --from=builder /opt/server /opt/server
USER roboshop 
CMD ["uwsgi", "--ini", "payment.ini"]
