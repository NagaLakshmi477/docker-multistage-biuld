# FROM python:3.9 AS builder
# EXPOSE 8080
# WORKDIR /opt/server
# COPY requirements.txt .
# COPY *.py .
# COPY payment.ini .
# RUN pip3 install -r requirements.txt

# FROM python:3.9.23-alpine3.22
# EXPOSE 8080
# RUN addgroup -S roboshop && adduser -S -G roboshop roboshop
# WORKDIR /opt/server
# COPY --from=builder /opt/server /opt/server
# USER roboshop
# CMD ["uwsgi", "--ini", "payment.ini"]

# Builder stage
FROM python:3.9 AS builder
WORKDIR /opt/server

# Install dependencies
COPY requirements.txt .
RUN pip install --prefix=/install -r requirements.txt

# Copy source code
COPY *.py .
COPY payment.ini .

# Final stage
FROM python:3.9-alpine
EXPOSE 8080

# Create non-root user
RUN addgroup -S roboshop && adduser -S -G roboshop roboshop

WORKDIR /opt/server

# Copy installed dependencies from builder
COPY --from=builder /install /usr/local
# Copy source code
COPY --from=builder /opt/server /opt/server

USER roboshop

CMD ["uwsgi", "--ini", "payment.ini"]
