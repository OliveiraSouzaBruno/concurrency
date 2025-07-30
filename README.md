# Concurrency Demo Project

Este projeto demonstra conceitos de concorrência em uma aplicação Spring Boot com gerenciamento de estoque de produtos. A aplicação foi projetada para testar cenários de alta concorrência usando Artillery para simulação de carga.

## 📋 Visão Geral

O projeto consiste em:
- **API REST** para gerenciamento de produtos (Spring Boot + JPA)
- **Banco PostgreSQL** para persistência de dados
- **Testes de carga** com Artillery para simular concorrência
- **Endpoint especial** `/add-to-cart` que simula redução de estoque com processamento assíncrono

## 🛠️ Pré-requisitos

### Opção 1: Usando Docker (Recomendado)
- Docker
- Docker Compose
- Java 21
- Maven 3.6.3

### Opção 2: Instalação Nativa
- PostgreSQL 15+
- Java 21
- Maven 3.6.3
- Node.js (para Artillery)

## 🐳 Configuração com Docker

### Instalando Docker e Docker Compose

- [Instalar Docker](https://docs.docker.com/get-docker/)
- [Instalar Docker Compose](https://docs.docker.com/compose/install/)

### Subindo o Banco de Dados

1. **Navegue até o diretório do banco:**
   ```bash
   cd database
   ```

2. **Suba o container PostgreSQL:**
   ```bash
   docker-compose up -d
   ```

3. **Verifique se o container está rodando:**
   ```bash
   docker-compose ps
   ```

O banco de dados ficará disponível em:
- **Host:** localhost
- **Porta:** 5432
- **Database:** concurrency
- **Usuário:** postgres
- **Senha:** 123456789

## 🗄️ Configuração Nativa do PostgreSQL

Se preferir usar PostgreSQL nativo:

**Crie a base de dados:**
   ```sql
   sudo -u postgres psql
   CREATE DATABASE concurrency;
   CREATE USER postgres WITH PASSWORD '123456789';
   GRANT ALL PRIVILEGES ON DATABASE concurrency TO postgres;
   \q
   ```

## 🚀 Executando a Aplicação

1. **Navegue até o diretório da aplicação:**
   ```bash
   cd demo
   ```

2. **Execute a aplicação:**
   ```bash

   # Ou usando Maven instalado
   mvn spring-boot:run
   
   # Ou abra em sua IDE de preferencia (IntelliJ, Eclipse)
   ```

A aplicação estará disponível em: `http://localhost:8080`

## 📊 Endpoints da API

### Produtos
- `GET /api/products` - Lista todos os produtos
- `GET /api/products/{id}` - Busca produto por ID
- `POST /api/products` - Cria novo produto
- `PUT /api/products/{id}` - Atualiza produto
- `DELETE /api/products/{id}` - Remove produto

### Concorrência
- `POST /api/products/{id}/add-to-cart` - Simula adição ao carrinho (reduz estoque)

### Exemplo de Produto (JSON):
```json
{
  "name": "Produto Teste",
  "price": 29.99,
  "stock": 100
}
```

## 🎯 Testando Concorrência com Artillery

### Instalando Artillery

```bash
# Instalar globalmente
npm install -g artillery

# Ou usando npx (sem instalação global)
npx artillery@latest run artillery/load-test.yml
```

### Executando Testes de Carga

1. **Certifique-se que a aplicação está rodando** em `http://localhost:8080`

2. **Crie um produto para testar:**
   ```bash
   curl -X POST http://localhost:8080/api/products \
     -H "Content-Type: application/json" \
     -d '{"name":"Produto Teste","price":29.99,"stock":100}'
   ```
   ou pode criar a partir do postman com a collection concurrency.postman_collection.json (Basta importar no seu postman)

3. **Execute o teste de carga:**
   ```bash
   # Do diretório raiz do projeto
   artillery run artillery/load-test.yml
   ou
   # Do diretório artillery
   artillery run load-test.yml
   ```

### Configuração do Teste
O arquivo `artillery/load-test.yml` está configurado para:
- **Duração:** 1 segundo
- **Requisições:** 50 simultâneas
- **Endpoint:** `/api/products/1/add-to-cart`

## 📈 Monitoramento e Análise

### Verificando Resultados
Após executar os testes, você pode:

1. **Verificar o estoque final:**
   ```bash
   curl http://localhost:8080/api/products/1
   ```

2. **Analisar logs da aplicação** para observar o comportamento da concorrência

3. **Verificar no banco de dados:**
   ```bash
   # Se usando Docker
   docker exec -it concurrency-db psql -U postgres -d concurrency
   SELECT * FROM product WHERE id = 1;
   ```
   ou Utilize sua IDE de preferencia recomendado: dbeaver

## 🔧 Funcionalidades de Concorrência

O projeto implementa:
- **ThreadPool fixo** com 20 threads para processamento assíncrono
- **CompletableFuture** para operações não-bloqueantes simulanto varios PODs ou ate mesmo um consumo de fila SQS
- **Transações JPA** para consistência de dados
- **Simulação de race conditions** no gerenciamento de estoque

## 📚 Postman Collection

O arquivo `concurrency.postman_collection.json` contém uma coleção completa de requisições para testar todos os endpoints da API.

## 🛑 Parando os Serviços

### Docker
```bash
cd database
docker-compose down
```

### Aplicação
Pressione `Ctrl+C` no terminal onde a aplicação está rodando ou se estiver rodando via IDE utilize o parar.

## 📝 Notas Importantes

- O endpoint `/add-to-cart` usa processamento **assíncrono** intencionalmente para demonstrar cenários de concorrência
- Em produção, operações críticas como controle de estoque devem usar mecanismos de sincronização adequados
- O projeto é para fins educacionais e demonstração de conceitos de concorrência
- Podemos notar que dessa forma o estoque dificilmente fica consistente com a quantidade de chamadas, o que pode ocorrer um overselling e desacasamento entre o estoque fisico e o estoque da aplicação

## 🤝 Comandos Rápidos

```bash
# Setup completo com Docker
cd database && docker-compose up -d && cd ../demo && ./mvnw spring-boot:run

# Teste rápido de carga
artillery run artillery/load-test.yml

# Parar tudo
docker-compose down -v
```
