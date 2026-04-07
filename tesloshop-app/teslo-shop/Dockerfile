############################
# Build stage
############################
FROM node:20-alpine AS build

WORKDIR /app

# Install dependencies (cached layer)
COPY package.json yarn.lock ./
RUN yarn config set network-timeout 300000 -g \
  && yarn install --frozen-lockfile --non-interactive

# Copy source and build
COPY . .
RUN yarn build

############################
# Production dependencies stage
############################
FROM node:20-alpine AS prod-deps

WORKDIR /app
COPY package.json yarn.lock ./
RUN yarn config set network-timeout 300000 -g \
  && yarn install --frozen-lockfile --production=true --non-interactive \
  && yarn cache clean

############################
# Runtime stage
############################
FROM node:20-alpine AS runtime

ENV NODE_ENV=production
WORKDIR /app

COPY --from=prod-deps /app/node_modules ./node_modules
COPY --from=build /app/dist ./dist
COPY --from=build /app/static ./static

EXPOSE 3000

CMD ["node","dist/main"]
