# Stage 1: Build App
FROM node AS builder
WORKDIR /sa-frontend
COPY package.json .
COPY package-lock.json .
RUN npm install
COPY . .
# Env variable to pass URL of WebApp Laod Balancer
ENV SA_WEBAPP_API_URL $SA_WEBAPP_API_URL
RUN npm run build

# Stage 2: Build Nginx App
FROM nginx
WORKDIR /usr/share/nginx/html
RUN rm -rf ./*
COPY --from=builder /sa-frontend/build .