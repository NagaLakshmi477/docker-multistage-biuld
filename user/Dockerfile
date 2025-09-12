# it will delete automatcllay
FROM node:20-alpine3.21 AS builder
WORKDIR /opt/server
COPY package.json .
COPY *.js .
RUN npm install


# below we are not installing anything
FROM node:20-alpine3.21
RUN addgroup -S roboshop && adduser -S -G roboshop roboshop
ENV MONGO="true"\
    REDIS_URL="redis://redis:6379"\
    MONGO_URL="mongodb://mongodb:27017/users"
WORKDIR /opt/server
USER roboshop
# copying the output from builder
COPY --from=builder /opt/server /opt/server
CMD [ "node" ,"server.js"]
