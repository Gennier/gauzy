ARG NODE_OPTIONS
ARG NODE_ENV
ARG API_HOST
ARG API_PORT
ARG DB_HOST
ARG DB_NAME
ARG DB_PORT
ARG HOST
ARG PORT

FROM node:alpine AS dependencies

LABEL maintainer="ever@ever.co"

ENV CI=true

RUN apk --update add bash && \
    apk add --no-cache dos2unix g++ git make python vips-dev && \
    mkdir /srv/gauzy && chown -R node:node /srv/gauzy

COPY wait .deploy/api/entrypoint.sh /
RUN chmod +x /wait /entrypoint.sh && dos2unix /entrypoint.sh

USER node:node

WORKDIR /srv/gauzy

COPY --chown=node:node apps/gauzy/package.json ./apps/gauzy/
COPY --chown=node:node apps/api/package.json ./apps/api/

COPY --chown=node:node packages/common/package.json ./packages/common/
COPY --chown=node:node packages/common-angular/package.json ./packages/common-angular/
COPY --chown=node:node packages/config/package.json ./packages/config/
COPY --chown=node:node packages/contracts/package.json ./packages/contracts/
COPY --chown=node:node packages/core/package.json ./packages/core/
COPY --chown=node:node packages/plugin/package.json ./packages/plugin/
COPY --chown=node:node packages/plugins/integration-ai/package.json ./packages/plugins/integration-ai/
COPY --chown=node:node packages/plugins/integration-hubstaff/package.json ./packages/plugins/integration-hubstaff/
COPY --chown=node:node packages/plugins/integration-upwork/package.json ./packages/plugins/integration-upwork/
COPY --chown=node:node packages/plugins/product-reviews/package.json ./packages/plugins/product-reviews/

COPY --chown=node:node lerna.json package.json yarn.lock ./

RUN yarn install --frozen-lockfile && yarn cache clean

FROM node:alpine AS prodDependencies

RUN apk --update add bash && \
    apk add --no-cache dos2unix g++ git make python vips-dev && \
    mkdir /srv/gauzy && chown -R node:node /srv/gauzy

USER node:node

WORKDIR /srv/gauzy

COPY --chown=node:node apps/api/package.json ./apps/api/

COPY --chown=node:node packages/common/package.json ./packages/common/
COPY --chown=node:node packages/config/package.json ./packages/config/
COPY --chown=node:node packages/contracts/package.json ./packages/contracts/
COPY --chown=node:node packages/core/package.json ./packages/core/
COPY --chown=node:node packages/plugin/package.json ./packages/plugin/
COPY --chown=node:node packages/plugins/integration-ai/package.json ./packages/plugins/integration-ai/
COPY --chown=node:node packages/plugins/integration-hubstaff/package.json ./packages/plugins/integration-hubstaff/
COPY --chown=node:node packages/plugins/integration-upwork/package.json ./packages/plugins/integration-upwork/
COPY --chown=node:node packages/plugins/product-reviews/package.json ./packages/plugins/product-reviews/

COPY --chown=node:node package.json yarn.lock ./

RUN yarn install --frozen-lockfile --production && yarn cache clean

RUN rm -r node_modules/@angular

FROM node:alpine AS development

USER node:node

WORKDIR /srv/gauzy

COPY --chown=node:node --from=dependencies /wait /entrypoint.sh /
COPY --chown=node:node --from=dependencies /srv/gauzy .
COPY . .

FROM node:alpine AS build

WORKDIR /srv/gauzy

RUN mkdir dist

COPY --chown=node:node --from=development /srv/gauzy .

ENV NODE_OPTIONS=${NODE_OPTIONS:-"--max-old-space-size=2048"}
ENV NODE_ENV=${NODE_ENV:-production}

RUN yarn build:package:api
RUN yarn build:api:prod

FROM node:alpine AS production

WORKDIR /srv/gauzy

COPY --chown=node:node --from=dependencies /wait ./wait
COPY --chown=node:node --from=dependencies /entrypoint.sh .
COPY --chown=node:node --from=prodDependencies /srv/gauzy/node_modules/ ./node_modules/
COPY --chown=node:node --from=build /srv/gauzy/packages/ ./packages/
COPY --chown=node:node apps/api/src/assets apps/api/src/assets
COPY --chown=node:node --from=build /srv/gauzy/dist/apps/api .

RUN npm install pm2 -g && mkdir /import && chown node:node /import && \
    touch ormlogs.log && chown node:node ormlogs.log && chown node:node wait && \
    chmod +x wait && chown -R node:node apps/
    
USER node:node

ENV NODE_OPTIONS=${NODE_OPTIONS:-"--max-old-space-size=2048"}
ENV NODE_ENV=${NODE_ENV:-production}
ENV API_HOST=${API_HOST:-api}
ENV API_PORT=${API_PORT:-3000}
ENV DB_HOST=${DB_HOST:-db}
ENV DB_NAME=${DB_NAME:-postgres}
ENV DB_PORT=${DB_PORT:-5432}
ENV HOST=${HOST:-0.0.0.0}
ENV PORT=${PORT:-3000}

EXPOSE ${PORT}

CMD [ "pm2-runtime", "main.js" ]
