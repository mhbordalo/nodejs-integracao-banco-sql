# API PetShop integrando ao PostgreSQL #

API para integração ao banco de dados PostgreSQL de um PetShop,
desenvolvida como requisito de aprendizado referente PosTech FIAP.

## Configuração da Aplicação ##

- utilizar a versão 20 do NodeJS:

`nvm use 20`

- no terminal para testar a aplicação:

 `npm run build && npm run start:dev`

## Criar o BD Postgres utilizando Docker ##

- verificar a imagem do Postgres local:

`docker images`

- listar os containers existentes:

`docker ps -a`

- na raiz da aplicação subir o container docker postgres:

`docker run --name mypostgres -e POSTGRES_USER=user -e POSTGRES_PASSWORD=123456 -d -p 5433:5432 postgres:latest`

- iniciar o container:

`docker start mypostgres`

- iniciar o DBeaver e configurar a conexão:
  - Host: localhost (ou 127.0.0.1)
  - Porta: 5433 (ou a porta mapeada)
  - Banco de Dados: postgres
  - Usuário: user (ou o nome configurado)
  - Senha: 123456 (ou a senha configurada)

- criar o banco de dados `pettech`

## Query SQL para criar as tabelas ##

- executar o script no DBeaver para criar e configurar as tabelas:

```sql
CREATE TABLE product (
  id UUID PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  description TEXT,
  image_url VARCHAR(255),
  price DOUBLE PRECISION
);

CREATE TABLE category (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  creation_date TIMESTAMP WITHOUT TIME ZONE
);

CREATE TABLE product_category (
  product_id UUID NOT NULL,
  category_id SERIAL NOT NULL,
  PRIMARY KEY (product_id, category_id),
  FOREIGN KEY (product_id) REFERENCES product (id) ON DELETE CASCADE,
  FOREIGN KEY (category_id) REFERENCES category (id) ON DELETE CASCADE
);

CREATE TABLE address (
  id SERIAL PRIMARY KEY,
  street VARCHAR(255) NOT NULL,
  city VARCHAR(255) NOT NULL,
  state VARCHAR(2) NOT NULL,
  zip_code VARCHAR(10) NOT NULL
);

CREATE TABLE person (
  id BIGSERIAL PRIMARY KEY,
  cpf VARCHAR(11) not null,
  name VARCHAR(100) not null,
  birth DATE not null,
  email VARCHAR(255) not null
);

CREATE TABLE "user" (
  id SERIAL PRIMARY KEY,
  username VARCHAR(255) NOT NULL,
  password VARCHAR(255) NOT NULL
);

ALTER TABLE address
ADD COLUMN person_id BIGINT NOT NULL;

ALTER TABLE address
ADD CONSTRAINT fk_address_person
FOREIGN KEY (person_id)
REFERENCES person(id);

ALTER TABLE person
ADD COLUMN user_id INTEGER UNIQUE,
ADD CONSTRAINT fk_user_id FOREIGN KEY (user_id) REFERENCES "user"(id);
```

- entrar no banco:
  
`docker exec -it mypostgres psql -U user -d pettech`

- listar as tabelas criadas:

`\dt`

- no terminal executar e configurar a migration

`npx typeorm migration:create ./src/lib/typeorm/migrations/ProductAutoGenerateUUID`

- e buildar a migration

`npm run build && npx typeorm migration:run -d ./build/lib/typeorm/typeorm.js`

## End-points ##

- Testar os endpoints utilizando Postman:

### user ###

GET  /user/:id

POST /user

```typescript
{
  "username": "Gustavo",
  "password": "123456"
}
```

POST /user/signin

```typescript
{
  "username": "Gustavo",
  "password": "123456"
}
```

- retorna token para utilizar na configuração de acesso:
  - Auth => Auth Type => Bearer Token

### produtc ###

GET  /product/:id

GET  /product

```typescript
Params: {
  "page": 1
  "limit": 10
}
```

POST /product

```typescript
{
  "name": "Bolinha Azul",
  "description": "Bolinha Azul",
  "image_url": "https://minhaurl.com.br/imagem",
  "price": 25.50
}
```

PUT  /product/:id

```typescript
{
  "name": "Bolinha Vermelha Edit/Update",
  "description": "Bolinha Vermelha",
  "image_url": "https://minhaurl.com.br/imagem",
  "price": 30.00,
  "categories": [
    {
      "name": "Brinquedo"
    }
  ]
}
```

DELETE /product/:id

### person ###

POST /person

```typescript
{
  "cpf": "123456789",
  "name": "Gustavo",
  "birth": "2000-09-15",
  "email": "teste1@gmail.com",
  "user_id": 1
}
```

### address ###

GET  /address/person

```typescript
Params: {
  "page": 1
  "limit": 10
}
```

POST /address

```typescript
{
  "street": "Rua dos Mirandas, 123",
  "city": "São Paulo",
  "state": "SP",
  "zip_code": "15457070",
  "person_id": 1
}
```

### category ###

POST /category

```typescript
{
  "name": "Brinquedo"
}
```

POST /product

```typescript
{
    "name": "Bolinha Azul",
    "description": "Bolinha Azul",
    "image_url": "https://minhaurl.com.br/imagem",
    "price": 25.50,
    "categories": [
        {
            "id": 1,
            "name": "Brinquedo"
        }
    ]
}
```
