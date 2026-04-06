# notification-app-java-php

Duas implementações da mesma API REST de notificações — uma em **Java (Spring Boot)** e outra em **PHP (Laravel)** — compartilhando a mesma rede Docker e estrutura de endpoints.

## Estrutura

```
.
├── docker-compose.yml       # Orquestração unificada
├── api/
│   ├── java/                # Spring Boot + PostgreSQL
│   └── php/                 # Laravel + PostgreSQL
```

## Como executar

### Pré-requisitos

- Docker
- Docker Compose v2+

### Subir o ambiente completo

```bash
docker compose up --build
```

Isso irá:
- Construir as imagens das duas APIs
- Subir dois bancos PostgreSQL isolados
- Executar as migrations automaticamente (PHP/Laravel)
- Criar as tabelas via JPA (Java/Spring Boot)
- Expor as aplicações na rede `notification-app-network`

### Endpoints disponíveis

| Serviço             | Host                       | Swagger UI                              |
|---------------------|----------------------------|-----------------------------------------|
| Java (Spring Boot)  | `http://localhost:8080`    | `/swagger-ui/index.html`                |
| PHP (Laravel)       | `http://localhost:8002`    | `/api/documentation`                    |
| PostgreSQL (Java)   | `localhost:5434`           | —                                       |
| PostgreSQL (PHP)    | `localhost:5433`           | —                                       |

> As portas do host podem variar se houver conflito com outros serviços locais. Ajuste o `docker-compose.yml` conforme necessário.

## API

Ambas as APIs expõem os mesmos recursos com a mesma estrutura de rotas:

### Autenticação

| Método | Rota            | Descrição              | Auth |
|--------|-----------------|------------------------|------|
| POST   | `/api/register` | Cadastrar usuário      | —    |
| POST   | `/api/login`    | Autenticar e obter JWT | —    |
| GET    | `/api/me`       | Perfil do usuário      | ✓    |

### Tipos de Notificação

| Método | Rota                       | Descrição                  | Auth |
|--------|----------------------------|----------------------------|------|
| POST   | `/api/type/create`         | Criar tipo                 | ✓    |
| GET    | `/api/type/me`             | Listar tipos do usuário    | ✓    |
| PUT    | `/api/type/update/{id}`    | Atualizar tipo             | ✓    |
| DELETE | `/api/type/delete/{id}`    | Remover tipo               | ✓    |

### Notificações

| Método | Rota                        | Descrição                        | Auth |
|--------|-----------------------------|----------------------------------|------|
| POST   | `/api/news/create`          | Criar notificação                | ✓    |
| GET    | `/api/news/me`              | Listar notificações do usuário   | ✓    |
| GET    | `/api/news/type/{typeId}`   | Filtrar por tipo                 | ✓    |
| PUT    | `/api/news/update/{id}`     | Atualizar notificação            | ✓    |
| DELETE | `/api/news/delete/{id}`     | Remover notificação              | ✓    |

### Campos das requisições

**Register**
```json
{ "nome": "João", "sobrenome": "Silva", "email": "joao@email.com", "senha": "min8chars" }
```
> PHP usa `password` no lugar de `senha`

**Login**
```json
{ "email": "joao@email.com", "senha": "min8chars" }
```
> PHP usa `password` no lugar de `senha`

**Tipo de Notificação**
```json
{ "nomeTipo": "Alertas de Sistema" }
```

**Atualizar Tipo**
```json
{ "novoNomeTipo": "Novo Nome" }
```

**Notificação**
```json
{
  "typeId": "<uuid ou integer>",
  "titulo": "Título",
  "descricao": "Resumo",
  "corpo": "Conteúdo completo",
  "imagemDestaque": "https://exemplo.com/img.png"
}
```
> `typeId` é UUID na API Java e inteiro na API PHP. `imagemDestaque` é opcional.

### Autenticação JWT

O token é retornado como string pura no corpo da resposta. Todas as rotas protegidas exigem o header:

```
Authorization: Bearer <token>
```

## Arquitetura

### Java (Spring Boot)
- **Clean Architecture** com separação em camadas: `api`, `application`, `domain`, `infrastructure`
- Spring Security + JWT (`jjwt`)
- Spring Data JPA + PostgreSQL
- Documentação via SpringDoc OpenAPI

### PHP (Laravel)
- Arquitetura MVC com Repository Pattern
- Autenticação JWT via `tymon/jwt-auth`
- Eloquent ORM + PostgreSQL
- Documentação via L5-Swagger

## Variáveis de ambiente

As variáveis já estão pré-configuradas no `docker-compose.yml` para desenvolvimento local. Para produção, ajuste:

| Variável                     | Padrão         | Descrição                  |
|------------------------------|----------------|----------------------------|
| `SPRING_DATASOURCE_URL`      | postgres-java  | JDBC URL do banco Java     |
| `SPRING_DATASOURCE_PASSWORD` | postgres       | Senha do banco Java        |
| `DB_HOST`                    | postgres-php   | Host do banco PHP          |
| `DB_PASSWORD`                | postgres       | Senha do banco PHP         |
| `JWT_SECRET`                 | (definido)     | Chave JWT da API PHP       |
| `security.jwt.secret`        | (definido)     | Chave JWT da API Java      |
