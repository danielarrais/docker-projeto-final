# Stage 1: Builder
FROM node:22-alpine AS builder

WORKDIR /app

# Instalar ferramentas de build (necessário para bcrypt e outras dependências nativas)
RUN apk add --no-cache python3 make g++

# Copiar arquivos de configuração do workspace e pacotes
COPY package.json package-lock.json ./
COPY backend/package.json ./backend/
COPY frontend/package.json ./frontend/

# Instalar todas as dependências para garantir que scripts e builds funcionem
RUN npm install

# Copiar o restante do código
COPY . .

# Stage 2: Production Dependencies
FROM node:22-alpine AS prod-deps

WORKDIR /app

# Instalar ferramentas de build novamente para compilar dependências de produção se necessário
RUN apk add --no-cache python3 make g++

COPY package.json package-lock.json ./
COPY backend/package.json ./backend/

# Instalar apenas dependências de produção do backend
RUN npm install --omit=dev --workspace=backend

# Stage 3: Runtime
FROM node:22-alpine

# Instalar curl para healthcheck
RUN apk add --no-cache curl

WORKDIR /app

# Copiar apenas os node_modules de produção
COPY --from=prod-deps /app/node_modules ./node_modules

# Copiar o código do backend
COPY --from=builder /app/backend ./

EXPOSE 3001

ENV NODE_ENV=production
ENV PORT=3001

HEALTHCHECK --interval=10s --timeout=5s --retries=5 --start-period=20s \
  CMD curl -f http://localhost:3001/api/tags || exit 1

CMD ["node", "index.js"]
