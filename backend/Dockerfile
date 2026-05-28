# Stage 1: Builder
FROM node:22-alpine AS builder

WORKDIR /app

# Instalar ferramentas de build
RUN apk add --no-cache python3 make g++

# Copiar arquivos de configuração do workspace e pacotes
COPY package.json package-lock.json ./
COPY backend/package.json ./backend/
COPY frontend/package.json ./frontend/

# Instalar dependências (Workspaces resolvem tudo aqui)
RUN npm install

# Copiar o restante do código
COPY . .

# Stage 2: Runtime
FROM node:22-alpine

# Instalar curl para healthcheck
RUN apk add --no-cache curl

WORKDIR /app

# Copiar os node_modules da raiz do builder para a raiz do runtime
COPY --from=builder /app/node_modules ./node_modules

# Copiar o conteúdo da pasta backend para a raiz do container de execução
COPY --from=builder /app/backend ./

EXPOSE 3001

ENV NODE_ENV=production
ENV PORT=3001

# O backend usa sequelize.sync({ alter: true }) ao iniciar
HEALTHCHECK --interval=10s --timeout=5s --retries=5 --start-period=20s \
  CMD curl -f http://localhost:3001/api/tags || exit 1

CMD ["node", "index.js"]
