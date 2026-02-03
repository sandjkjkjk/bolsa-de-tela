# 1. Base y Prune
FROM oven/bun:1 AS base
WORKDIR /app
RUN bun install turbo --global
COPY . .
RUN turbo prune --scope=api --docker

# 2. Instalar y Construir
FROM oven/bun:1 AS build
WORKDIR /app

# Copiamos solo lo necesario para instalar dependencias
COPY --from=base /app/out/json/ .
COPY --from=base /app/out/bun.lock ./bun.lock

# AGREGADO: --ignore-scripts evita que Prisma intente ejecutarse sin el schema
RUN bun install --ignore-scripts

# Ahora copiamos el resto de los archivos (incluyendo el schema.prisma)
COPY --from=base /app/out/full/ .

# Generar Prisma Client
RUN bun x prisma generate --schema=apps/api/prisma/schema.prisma

# Construir la API
RUN bun run build --filter=api

# 3. Runner
FROM oven/bun:1-slim AS runner
WORKDIR /app

ENV NODE_ENV=production
ENV PORT=4000

# Copiamos dependencias y el código compilado
COPY --from=build /app/node_modules ./node_modules
COPY --from=build /app/apps/api/dist ./apps/api/dist
COPY --from=build /app/apps/api/package.json ./apps/api/package.json
COPY --from=build /app/apps/api/prisma ./apps/api/prisma
# IMPORTANTE: Copiamos el archivo de configuración a la raíz de ejecución de la app
COPY --from=build /app/apps/api/prisma.config.ts ./apps/api/prisma.config.ts

# Nos movemos a la carpeta de la aplicación para que Prisma encuentre el config.ts localmente
WORKDIR /app/apps/api

EXPOSE 4000

WORKDIR /app

# Ejecutamos prisma apuntando explícitamente al config de la api
CMD ["sh", "-c", "bun x prisma db push --config apps/api/prisma.config.ts && bun apps/api/dist/main.js"]
