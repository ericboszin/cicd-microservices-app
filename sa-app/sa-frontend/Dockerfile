# Stage 1: Build App
FROM node:latest AS builder
WORKDIR /sa-frontend
COPY package.json .
COPY package-lock.json .
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Build Nginx App
FROM nginx
WORKDIR /usr/share/nginx/html
RUN rm -rf ./*
COPY --from=builder /sa-frontend/build .