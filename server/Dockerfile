# syntax=docker/dockerfile:1

# ---- web: build the Svelte UI ----
FROM node:24-alpine AS web
WORKDIR /src
COPY web/package.json web/package-lock.json ./
RUN npm ci
COPY web .
RUN npm run build
# vite writes to ../internal/web/dist relative to /src/web; with WORKDIR /src that is /internal/web/dist.

# ---- build: compile the Go binary with the UI embedded ----
FROM golang:1.27-alpine AS build
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY . .
COPY --from=web /internal/web/dist ./internal/web/dist
ARG VERSION=1.3.0
RUN CGO_ENABLED=0 go build -trimpath -ldflags="-s -w -X github.com/chrisgreg/boop/server/internal/api.Version=${VERSION}" -o /boop ./cmd/boop

# ---- runtime: binary + CA certificates only ----
FROM alpine:3.21
RUN apk add --no-cache ca-certificates tzdata && adduser -D -u 1000 boop && mkdir -p /data && chown boop:boop /data
COPY --from=build /boop /usr/local/bin/boop
USER boop
VOLUME ["/data"]
EXPOSE 8080
ENV BOOP_PORT=8080 BOOP_DATABASE_PATH=/data/boop.db
HEALTHCHECK --interval=30s --timeout=3s CMD wget -qO- http://127.0.0.1:8080/health || exit 1
ENTRYPOINT ["boop"]
