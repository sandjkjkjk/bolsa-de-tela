# 1. Base y Prune
FROM oven/bun:1 AS base
WORKDIR /app
RUN bun install turbo --global
COPY . .
# Extrae solo lo necesario para "web"
RUN turbo prune --scope=web --docker

# 2. Instalar y Construir
FROM oven/bun:1 AS build
WORKDIR /app

# --- CAMBIO AQUÍ ---
COPY --from=base /app/out/json/ .
# Intentamos copiar bun.lock. Si tu archivo es bun.lockb, cambia la linea de abajo.
COPY --from=base /app/out/bun.lock ./bun.lock
# -------------------

RUN bun install

COPY --from=base /app/out/full/ .

# Argumentos de Build (Variables de entorno)
ARG NEXT_PUBLIC_API_URL
ARG NEXT_PUBLIC_SUPABASE_URL=https://example.com
ARG NEXT_PUBLIC_SUPABASE_ANON_KEY=example-key
ENV NEXT_PUBLIC_API_URL=$NEXT_PUBLIC_API_URL
ENV NEXT_PUBLIC_SUPABASE_URL=$NEXT_PUBLIC_SUPABASE_URL
ENV NEXT_PUBLIC_SUPABASE_ANON_KEY=$NEXT_PUBLIC_SUPABASE_ANON_KEY

# Construimos el frontend
RUN bun run build --filter=web

# 3. Runner (Imagen ligera de Node o Bun)
FROM oven/bun:1-slim AS runner
WORKDIR /app

ENV NODE_ENV=production
ENV PORT=3000
ENV HOSTNAME="0.0.0.0"

# Copiamos solo los archivos Standalone que genera Next.js
COPY --from=build /app/apps/web/public ./apps/web/public
COPY --from=build /app/apps/web/.next/static ./apps/web/.next/static
COPY --from=build /app/apps/web/.next/standalone ./

# Exponemos el puerto
EXPOSE 3000

# Arrancamos server.js (ubicado dentro de la estructura standalone)
CMD ["bun", "apps/web/server.js"]
