FROM node:20-alpine AS base


RUN apk update
RUN apk add --no-cache libc6-compat
RUN yarn add turbo

COPY . .
RUN yarn --ignore-scripts

CMD yarn turbo db:push --filter=@repo/db