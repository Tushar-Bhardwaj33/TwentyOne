# syntax=docker/dockerfile:1

ARG NODE_VERSION=22.13.1

# --- Build Stage ---
FROM node:${NODE_VERSION}-slim AS builder
WORKDIR /app

# Install dependencies (npm ci for deterministic builds)
COPY --link package.json package-lock.json ./
RUN --mount=type=cache,target=/root/.npm \
    npm ci

# Copy the rest of the application source
COPY --link . .

# Build the app (Vite + TypeScript)
RUN npm run build

# Remove dev dependencies to reduce image size
RUN rm -rf node_modules && \
    npm ci --omit=dev

# --- Production Stage ---
FROM node:${NODE_VERSION}-slim AS final
WORKDIR /app

# Create a non-root user
RUN addgroup --system appgroup && adduser --system --ingroup appgroup appuser

# Copy built app and production dependencies
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./
COPY --from=builder /app/index.html ./
COPY --from=builder /app/public ./public

ENV NODE_ENV=production
ENV NODE_OPTIONS="--max-old-space-size=4096"
USER appuser

EXPOSE 4173

CMD ["npx", "vite", "preview", "--host", "0.0.0.0", "--port", "4173"]
