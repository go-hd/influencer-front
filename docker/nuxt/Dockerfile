FROM node:10.14.1-alpine

WORKDIR /app

RUN apk update && \
    apk add git && \
    npm install -g npm && \
    npm install -g vue-cli yarn

ENV HOST 0.0.0.0
EXPOSE 3000
