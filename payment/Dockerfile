FROM python:3.9 AS builder
EXPOSE 8080
WORKDIR /opt/server
COPY requirements.txt .
COPY *.py .
COPY payment.ini .
RUN pip3 install -r requirements.txt

FROM 3.9.23-alphine3.22
EXPOSE 8080
RUN addgroup -S roboshop && adduser -S -G roboshop roboshop
WORKDIR /opt/server
COPY --from=builder /opt/server /opt/server
USER roboshop
CMD ["uwsgi", "--ini", "payment.ini"]