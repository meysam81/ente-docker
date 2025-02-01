FROM oven/bun:1 AS builder

WORKDIR /ente

COPY . .
COPY apps/ .

ARG NEXT_PUBLIC_ENTE_ENDPOINT
ARG NEXT_PUBLIC_ENTE_ALBUMS_ENDPOINT

ENV NEXT_PUBLIC_ENTE_ENDPOINT=${NEXT_PUBLIC_ENTE_ENDPOINT}
ENV NEXT_PUBLIC_ENTE_ALBUMS_ENDPOINT=${NEXT_PUBLIC_ENTE_ALBUMS_ENDPOINT}

RUN bun i git://github.com/dimsemenov/photoswipe#v5-beta && \
    bun i

FROM builder AS build-accounts
RUN bun run build:accounts

FROM builder AS build-auth
RUN bun run build:auth

FROM builder AS build-cast
RUN bun run build:cast

FROM builder AS build-photos
RUN bun run build:photos

FROM ghcr.io/static-web-server/static-web-server:2 AS base-static
EXPOSE 8080
ONBUILD ENV SERVER_PORT=8080 \
    SERVER_COMPRESSION_LEVEL=fastest \
    SERVER_ERROR_PAGE_404=index.html \
    SERVER_FALLBACK_PAGE=index.html \
    SERVER_HEALTH=true \
    SERVER_LOG_LEVEL=info \
    SERVER_LOG_REMOTE_ADDRESS=true

FROM base-static AS accounts
COPY --from=build-accounts /ente/apps/accounts/out /public

FROM base-static AS auth
COPY --from=build-auth /ente/apps/auth/out /public

FROM base-static AS cast
COPY --from=build-cast /ente/apps/cast/out /public

FROM base-static AS photos
COPY --from=build-photos /ente/apps/photos/out /public
