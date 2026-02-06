FROM node:25-alpine AS builder
WORKDIR /usr/src/app

COPY --chown=node:node .yarn/ .yarn/
COPY --chown=node:node src/ src/

COPY --chown=node:node yarn.lock .
COPY --chown=node:node package.json .
COPY --chown=node:node .yarnrc.yml .
COPY --chown=node:node tsup.config.ts .
COPY --chown=node:node tsconfig.json .
COPY --chown=node:node entrypoint.sh entrypoint.sh

RUN apk add --no-cache ca-certificates curl

RUN yarn install --immutable \
    && yarn tsup

RUN yarn workspaces focus --all --production \
    && yarn cache clean \
    && rm -rf /tmp/* /var/tmp/*

FROM node:25-alpine AS runner
WORKDIR /usr/src/app

COPY --chown=node:node --from=builder /usr/src/app/dist ./dist
COPY --chown=node:node --from=builder /usr/src/app/node_modules ./node_modules
COPY --chown=node:node --from=builder /usr/src/app/package.json ./package.json
COPY --chown=node:node entrypoint.sh ./entrypoint.sh

RUN chmod +x ./entrypoint.sh \
    && chown -R node:node /usr/src/app \
    && rm -rf /usr/src/app/.yarn /usr/src/app/.yarnrc.yml /usr/src/app/tsconfig.json /usr/src/app/tsup.config.ts /usr/src/app/yarn.lock

ENV NODE_ENV=production

USER node

ENTRYPOINT ["./entrypoint.sh"]
