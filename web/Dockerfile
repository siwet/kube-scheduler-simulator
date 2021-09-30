FROM node:alpine AS deps

RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories


RUN apk update && \
    apk upgrade && \
    apk add --no-cache make gcc g++ py-pip

WORKDIR /app
COPY package.json yarn.lock ./
RUN yarn config set registry https://registry.npm.taobao.org/
RUN yarn install --frozen-lockfile

FROM node:alpine AS builder
WORKDIR /app
COPY . .
COPY --from=deps /app/node_modules ./node_modules
RUN yarn config set registry https://registry.npm.taobao.org/
RUN yarn build && yarn install --production --ignore-scripts --prefer-offline

FROM node:alpine AS runner
WORKDIR /app

ENV NODE_ENV production

RUN addgroup -g 1001 -S nodejs
RUN adduser -S nuxtjs -u 1001

#COPY --from=builder /app/nuxt.config.js ./
#COPY --from=builder ./app/server ./server/
COPY --from=builder ./app/package.json ./
COPY --from=builder ./app/node_modules ./node_modules/
COPY --from=builder ./app/.nuxt ./.nuxt/
COPY --from=builder ./app/static ./static/

USER nuxtjs

EXPOSE 3000
ENV NUXT_HOST=0.0.0.0
CMD ["yarn", "start"]
