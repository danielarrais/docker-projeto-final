# Stage 1: Builder
FROM node:22-alpine AS builder

WORKDIR /app

# Copiar arquivos de configuração do workspace e pacotes
COPY package.json package-lock.json ./
COPY backend/package.json ./backend/
COPY frontend/package.json ./frontend/

# Instalar todas as dependências a partir da raiz (Workspaces)
RUN npm install

# Copiar o restante do código
COPY . .

# Buildar o frontend
WORKDIR /app/frontend
RUN npm run build

# Stage 2: Runtime (Nginx)
FROM nginx:alpine

# Instalar curl para healthcheck
RUN apk add --no-cache curl

# Copiar os arquivos estáticos gerados no stage anterior
COPY --from=builder /app/frontend/dist /usr/share/nginx/html

# Configurar Nginx para SPA e Proxy para o Backend
RUN echo 'server { \
    listen 80; \
    location / { \
        root /usr/share/nginx/html; \
        index index.html; \
        try_files $uri $uri/ /index.html; \
    } \
    # Proxy para o backend \
    location /api { \
        proxy_pass http://backend:3001/api; \
        proxy_http_version 1.1; \
        proxy_set_header Upgrade $http_upgrade; \
        proxy_set_header Connection "upgrade"; \
        proxy_set_header Host $host; \
        proxy_cache_bypass $http_upgrade; \
    } \
}' > /etc/nginx/conf.d/default.conf

EXPOSE 80

HEALTHCHECK --interval=10s --timeout=5s --retries=3 \
  CMD curl -f http://localhost:80 || exit 1

CMD ["nginx", "-g", "daemon off;"]
