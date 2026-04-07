############################
# Build stage
############################
FROM node:20-alpine AS build

WORKDIR /app

COPY package.json package-lock.json ./
RUN npm config set fetch-retries 5 \
  && npm config set fetch-retry-mintimeout 20000 \
  && npm config set fetch-retry-maxtimeout 120000 \
  && npm config set registry https://registry.npmjs.org/ \
  && npm ci --no-audit --no-fund

COPY . .
RUN npm run build

############################
# Runtime stage
############################
FROM nginx:1.27-alpine AS runtime

COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build /app/dist/teslo-shop/browser /usr/share/nginx/html

EXPOSE 80
