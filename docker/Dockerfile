FROM node:18-alpine AS base

FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app

COPY yarn.lock* ./
RUN \
  if [ -f yarn.lock ]; then yarn --frozen-lockfile; \
  else echo "Lockfile not found." && exit 1; \
  fi

FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

RUN npx create-next-app \
    && yarn install --silent \
    && yarn add sharp \
    && yarn build
FROM base AS runner
WORKDIR /app

ENV NODE_ENV production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public

RUN mkdir .next
RUN chown nextjs:nodejs .next

# COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

ENV PORT 3000
ENV HOSTNAME "0.0.0.0"

FROM nginx:1.23.1-alpine AS nginx
COPY --from=builder /app/dist /usr/share/nginx/html
COPY --from=builder nginx.conf.template /etc/nginx/conf.d/default.conf
COPY . .
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]