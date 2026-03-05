# СТАДИЯ 1: Обрезка монорепы (Prune)
FROM node:20-alpine AS builder
RUN apk add --no-cache libc6-compat
# Устанавливаем turbo глобально
RUN npm add -g turbo@^2.8.13
WORKDIR /app
# Копируем ВЕСЬ монорепозиторий
COPY . .
# Эта команда создает папку out/, в которой лежит ТОЛЬКО код apps/web и его зависимостей (packages/ui)
RUN turbo prune web --docker

# СТАДИЯ 2: Установка зависимостей (Installer)
FROM node:20-alpine AS installer
RUN apk add --no-cache libc6-compat
WORKDIR /app

# Включаем pnpm
RUN corepack enable pnpm

# Копируем только файлы манифестов (package.json, pnpm-lock.yaml), чтобы закэшировать установку пакетов
COPY .gitignore .gitignore
COPY --from=builder /app/out/json/ .
COPY --from=builder /app/out/pnpm-lock.yaml ./pnpm-lock.yaml

RUN pnpm install --frozen-lockfile  

# Копируем реальный код (который меняется часто)
COPY --from=builder /app/out/full/ .
COPY turbo.json turbo.json

# Собираем ТОЛЬКО web (и его зависимости)
RUN pnpm turbo run build --filter=web...

# СТАДИЯ 3: Запуск (Runner)
FROM node:20-alpine AS runner
WORKDIR /app

# Безопасность: не запускаем под root
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs
USER nextjs

COPY --from=installer /app/apps/web/next.config.js .
COPY --from=installer /app/apps/web/package.json .

# Копируем скомпилированное Next.js приложение (результат output: 'standalone')
COPY --from=installer --chown=nextjs:nodejs /app/apps/web/.next/standalone ./
COPY --from=installer --chown=nextjs:nodejs /app/apps/web/.next/static ./apps/web/.next/static
COPY --from=installer --chown=nextjs:nodejs /app/apps/web/public ./apps/web/public

CMD ["node", "apps/web/server.js"]
