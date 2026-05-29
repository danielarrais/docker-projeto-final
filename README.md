# Integrantes Grupo 6
Laís Uchôa
Alan Garcia
José Gomes
Daniel Arrais
Tiago Sobreira
Túlio Coimbra

# Conduit RealWorld Example App - Dockerized

Este projeto é um fork do repositório original [TonyMckes/conduit-realworld-example-app](https://github.com/TonyMckes/conduit-realworld-example-app). Foram adicionados os arquivos de configuração do Docker (`Dockerfile` para backend e frontend) e o arquivo `docker-compose.yml` para facilitar a orquestração e o deploy da aplicação em containers.

Esta é uma implementação da especificação RealWorld (Medium.com clone) utilizando uma arquitetura moderna e containerizada. A aplicação consiste em um Backend em Node.js (Express/Sequelize) e um Frontend em React 19.

## Arquitetura da Aplicação

A aplicação segue uma estrutura de Monorepo utilizando npm Workspaces, o que facilita a gestão de dependências compartilhadas entre o backend e o frontend.

- Backend: API REST robusta utilizando Express.js, com persistência em PostgreSQL através do Sequelize ORM.
- Frontend: Single Page Application (SPA) moderna com React 19, utilizando Vite para um ambiente de desenvolvimento rápido e builds otimizados.
- Banco de Dados: PostgreSQL 15 para armazenamento relacional persistente.
- Proxy/Web Server: Nginx atuando como servidor de arquivos estáticos para o frontend e proxy reverso para a API.

## Estratégia de Containerização - Backend

### Imagem Base e Eficiência
- **Node.js Alpine**: Utilizamos `node:22-alpine` para garantir uma imagem final extremamente leve e com menor superfície de ataque.
- **Camadas de Cache**: A cópia seletiva dos arquivos `package.json` antes do código-fonte otimiza o cache do Docker, acelerando builds subsequentes.

### Processo de Build e Segurança
- **Build Multi-stage**: Separamos o ambiente de compilação do ambiente de execução. Ferramentas como `python3` e `g++` são usadas apenas no estágio inicial para dependências nativas e descartadas na imagem final.
- **Node Modules de Produção**: Apenas as dependências necessárias para a execução são mantidas, reduzindo o tamanho total do container.

### Resiliência
- **Healthcheck**: Implementamos uma verificação ativa via `curl` para garantir que a API esteja pronta para processar requisições antes de ser considerada "saudável" pela orquestração.

## Estratégia de Containerização - Frontend

### Build e Performance
- **Vite Build**: O build é realizado em um container isolado, gerando artefatos estáticos minificados e otimizados para produção.
- **Isolamento de Código**: O código-fonte do frontend nunca chega ao servidor de produção, apenas o bundle final compilado.

### Servidor Web e Roteamento
- **Nginx Alpine**: Utilizamos o Nginx como servidor de alta performance para entrega de conteúdo estático.
- **Suporte a SPA**: Configuração customizada do Nginx para lidar com o roteamento do React (History API), garantindo que atualizações de página funcionem corretamente.
- **Proxy Reverso Integrado**: O Nginx redireciona chamadas `/api` internamente para o container do backend, unificando o ponto de entrada e eliminando problemas de CORS.

## Orquestração do Ambiente (Docker Compose)

O arquivo `docker-compose.yml` gerencia a stack completa com as seguintes definições:

### Redes e Isolamento
- **backend_db_net**: Rede privada que isola o tráfego do banco de dados, protegendo-o de acessos externos.
- **frontend_backend_net**: Rede dedicada para a comunicação entre o servidor web e a API.

### Persistência de Dados
- **PostgreSQL Volume**: Uso de volume nomeado para garantir que os dados do banco persistam mesmo após a remoção dos containers.

### Controle de Fluxo
- **Dependências Inteligentes**: O uso de `depends_on` com `service_healthy` garante que a stack suba na ordem correta, evitando erros de "database not ready" ou "api unreachable".

## Configuração

A aplicação utiliza um arquivo `.env` na raiz do projeto para gerenciar variáveis de ambiente sensíveis e configurações de infraestrutura.

1.  Crie um arquivo `.env` baseado no exemplo:
    ```bash
    cp .env.example .env
    ```
2.  (Opcional) Ajuste as variáveis conforme necessário (senhas, portas, chaves JWT).

## Como Rodar

Para subir a aplicação completa, certifique-se de ter o arquivo `.env` configurado e execute:

```bash
docker-compose up --build
```

O frontend estará disponível em: http://localhost:8080
O backend estará disponível em: http://localhost:3001
