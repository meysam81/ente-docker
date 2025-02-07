FROM node:bookworm-slim AS builder

WORKDIR /ente

COPY . .
COPY apps/ .

ARG NEXT_PUBLIC_ENTE_ENDPOINT \
    NEXT_PUBLIC_ENTE_ALBUMS_ENDPOINT \
    NEXT_ENTE_PUBLIC_ACCOUNTS_ENDPOINT \
    NEXT_PUBLIC_ENTE_FAMILY_URL

ENV NEXT_PUBLIC_ENTE_ENDPOINT=${NEXT_PUBLIC_ENTE_ENDPOINT} \
    NEXT_PUBLIC_ENTE_ALBUMS_ENDPOINT=${NEXT_PUBLIC_ENTE_ALBUMS_ENDPOINT} \
    NEXT_ENTE_PUBLIC_ACCOUNTS_ENDPOINT=${NEXT_ENTE_PUBLIC_ACCOUNTS_ENDPOINT} \
    NEXT_PUBLIC_ENTE_FAMILY_URL=${NEXT_PUBLIC_ENTE_FAMILY_URL}

RUN corepack enable && \
    yarn cache clean && \
    yarn install --network-timeout 1000000000

FROM builder AS build-accounts
RUN yarn build:accounts

FROM builder AS build-auth
RUN yarn build:auth

FROM builder AS build-cast
RUN yarn build:cast

FROM builder AS build-photos
RUN yarn build:photos

FROM ghcr.io/static-web-server/static-web-server:2 AS base-static
EXPOSE 8080
ONBUILD ENV SERVER_PORT=8080 \
    SERVER_COMPRESSION_LEVEL=fastest \
    SERVER_COMPRESSION=true \
    SERVER_EXPERIMENTAL_METRICS=true \
    SERVER_FALLBACK_PAGE=index.html \
    SERVER_HEALTH=true \
    SERVER_LOG_FORWARDED_FOR=true \
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
