# Aplicação PetShop - API Integrando Banco de Dados PostgreSQL #

## Configuração base do projeto - Módulo de Produtos ##

- @types/node biblioteca com os tipos do NodeJS,
- tsup biblioteca para fazer o transpiler o nosso código com zero configurações,
- tsx biblioteca para executar nosso código typescript com zero configuração typescript,
- fastify framework servidor web rápido e simples, inspirado no express e no Hapi.

```markdown
npm init -y
npm i -D @types/node tsup tsx typescript
npm i fastify
```

- zod biblioteca para validação de schema
- dotenv biblioteca para adicionarmos variáveis de ambiente no processo do Node.JS

`npm i dotenv zod`

- eslint ferramenta de análise de código estática para padronizar o código JavaScript

```markdown
npm i -D @typescript-eslint/eslint-plugin@6.21.0 @typescript-eslint/parser@6.21.0 eslint@8.57.0 eslint-config-prettier@9.1.0 eslint-config-standard@17.1.0 eslint-plugin-import@2.29.1 eslint-plugin-n@16.6.2 eslint-plugin-prettier@5.1.3 eslint-plugin-promise@6.1.1 prettier@3.2.5
```

- caso não tenha uma cola com a configuração do eslint

`npx eslint init`

- utilizar a versão 20 do Node

`nvm use 20`

- iniciar configurações do typescript

`npx tsc --init`

- ajustar configurações no tsconfig.json

```typescript
"target": "es2020",
"baseUrl": "./",
"paths": { "@/*": ["./src/*"] },
```

- agora já pode utilizar o alias no arquivo server.ts

`import { app } from '@/app';`

## Estrutura das pastas ##

```markdown
pettech/
├── build/
├── node_modules
├── src/
│   ├── entities/
│   │   └── models/
│   ├── env/
│   ├── http/
│   │   ├── controllers/
│   │   │   ├── address/
│   │   │   ├── category/
│   │   │   ├── person/
│   │   │   ├── product/
│   │   │   └── user
│   │   └── middlewares/
│   ├── lib/
│   │   ├── pg/
│   │   └── typeorm
│   │       └── migrations/
│   ├── repositories/
│   │   ├── pg/
│   │   └── typeorm/
│   ├── use-cases/
│   │   ├── errors/
│   │   └── factory/
│   ├── utils/
│   ├── app.ts
│   └── server.ts
├── .env
├── .env.example
├── .eslint.json
├── .gitignore
├── .npmrc
├── package.json
├── tsconfig.json
└── README.md
```

## Conexão PostgreSQL utilizando Docker e DBeaver ##

- certifique-se de que você tem a imagem do PostgreSQL disponível localmente

`docker images`

- verifique os containers existentes

`docker ps -a`

- verificar se existe outro serviço utilizando a mesma porta para não dar conflito

`sudo ss -tuln | grep 5432`

- caso não exista, criar um container postgres com a configuração abaixo

`docker run --name mypostgres -e POSTGRES_USER=user -e POSTGRES_PASSWORD=123456 -d -p 5433:5432 postgres:latest`

- caso tenha conflito de containers, remova o atual e crie um novo container

`docker rm -f <nome-do-container>` ou `docker stop mypostgres` e `docker rm mypostgres`

- conferir se o container subiu corretamente

`docker logs mypostgres`

- iniciar o continer que estiver parado

`docker start mypostgres`

- para abrir um terminal interativo dentro do container`

`docker exec -it mypostgres bash`

- para listar todos os bancos de dados no PostgreSQL

`psql -U user -c "\l"`

- conectar ao PostgreSQL de fora do container com o DBeaver

`https://dbeaver.com/docs/dbeaver/`

```markdown
- Host: localhost (ou 127.0.0.1)
- Porta: 5433 (ou a porta mapeada)
- Banco de Dados: postgres (ou o banco que deseja acessar)
- Usuário: user (ou o nome de usuário configurado)
- Senha: 123456 (ou a senha configurada)
```

- DBeaver com o botão direito criar o banco de dados "pettech"
- abrir o banco "pettech" e no editor SQL abrir script SQL
- utilizar o script abaixo pra criar e configurar as tabelas

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

- para listar as tabelas criadas em um banco de dados PostgreSQL
  - se conectar ao banco de dados usando o cliente psql

  `psql -U user -d <nome_do_banco>`

  - se já estiver dentro do contêiner Docker
  
  `docker exec -it mypostgres psql -U user -d <nome_do_banco>`

  - para listar as tabelas dentro do psql => `\dt`
  - para sair do psql => `\q`

## Repository Pattern ##

Repository Pattern é um padrão de design que abstrai a lógica de acesso a dados de um aplicativo, permitindo que ele interaja com diferentes tipos de bancos de dados sem alterar a lógica de negócios. Ele cria uma camada intermediária entre a lógica de negócios e o acesso a dados, tornando o código mais organizado, reutilizável e fácil de manter.

## Use Cases ##

Um Caso de Uso é uma representação completa de todas as potenciais ações que um usuário pode realizar em um sistema. Cada ação é considerada uma requisição, e o Caso de Uso especifica como o sistema responde a essa requisição.

## Solid ##

SOLID é um conjunto de cinco princípios de design orientado a objetos que ajudam a produzir software que é fácil de manter, de entender e de expandir ao longo do tempo.

- Principio da Responsabilidade Única => uma classe deve ter apenas uma responsabilidade, ex: a classe cadastro de funcionario, não deve calcular salario, claculo deve estar em outra classe.

- Principio da inversão de dependencia = > devemos depender de abstrações (interfaces) ao invés de implementações (classes concretas) a fim de ter um menor acoplamento entre as camadas do sistema.

## Configuração do script start da aplicação ##

```typescript
"scripts": {
  "start:dev": "tsx watch src/server.ts",
  "start": "node build/server.js",
  "build": "tsup src --out-dir build"
}
```

- no terminal testar a aplicação:

`npm run start:dev` e `npm run build`

- testar criar uma pessoa utilizando Postman com return 201 Created

`POST http://localhost:3000/person`

```typescript
body:
  {
      "cpf": "123456789",
      "name": "Gustavo",
      "birth": "1991-09-15",
      "email": "teste@gmail.com"
  }
```

## Integrando o banco de dados ##

- DRIVER DB => driver de banco de dados é um software que permite a uma aplicação interagir com um banco de dados, facilitando operações como conexão, consulta, atualização e manipulação de dados (api).

- ORM (Object-Relational Mapping) é uma técnica de programação que converte dados entre sistemas incompatíveis usando programação orientada a objetos, permitindo que os desenvolvedores interajam com o banco de dados usando objetos em vez de SQL.

- inicialmente vamos utilizar o driver nativo pg e singleton class

`npm i pg && npm i -D @types/pg`

- Instalar no VSCode as extensões abaixo para se conectar no banco de dados sem utilizar o DBeaver
  - Database Client
  - Database Client JDBC

- Agora vamos intalar o ORM para import das dependencias, e caso opte por utilizar somente o ORM, tambem precisa instalar um drive de conexão com o banco de dados

`npm i typeorm reflect-metadata && npm i --save-dev @types/node && npm i @swc/core`

- será necessário habilitar ou configurar no tsconfig.json as linhas

```typescript
"experimentalDecorators": true,
"emitDecoratorMetadata": true,
"types": ["node"],
```

## Factory Pattern ##

O Factory Pattern é um padrão de design que usa métodos de fábrica para lidar com a criação de objetos, permitindo que uma classe delegue a instânciação de objetos para subclasses.

## Migrations ##

- Migrations são uma maneira de gerenciar alterações e evoluções do esquema de um banco de dados ao longo do tempo, permitindo que você versione e controle as alterações de forma sistemática e organizada.

`npx typeorm migration:create ./src/lib/migrations/ProductAutoGenerateUUID`

- depois que configurar a migration e o typeorm vamos buildar a migration

`npm run build && npx typeorm migration:run -d ./build/lib/typeorm.js`

- caso queira reverter a migration acima
`npx typeorm migration:revert -d ./build/lib/typeorm.js`

## Autenticação com JWT ##

- JWT (JSON Web Token) é um padrão para autenticação de usuários em uma API, onde um token compacto e autocontido é gerado durante o login (sign-in), contendo um payload com informações do usuário, que pode ser verificado e confiável porque é assinado digitalmente

`npm i @fastify/jwt bcryptjs && npm i -D @types/bcryptjs`

- Configurar Autenticação no Postman

```typescript
POST http://localhost:3000/user/signin

Send
Body: {
  "username": "Gustavo",
  "password": "123456"
}

Return
Body: {
  "token": "exemplo.yJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.exemplo"
}
```

- Configurar no Auth => Auth Type => inserir token no Bearer Token

## Middlewares ##

- Middleware em uma API web é uma função ou um conjunto de funções que são executadas antes da rota final do pedido, usadas para manipular o pedido e a resposta, ou para executar tarefas específicas como autenticação, logging, etc.

## MongoDB ##

- O MongoDB é um banco de dados não relacional e orientado a documentos ele traz uma abordagem flexível e escalável e ele não vai salvar os dados da maneira tradicional como um banco SQL ele vai salvar objetos BJSON(Bynary JSON) a estrutura e a mesma do JSON que ja conhecemos.

- Recomendado quando precisamos salvar e consultar grandes volumes de dados não estruturados.

## Configuração base do projeto - Módulo de Estoque ##

## Conexão MongoDB utilizando Docker ##

`docker run --name myMongoDB -p 27017:27017 -d mongo:latest`

- Vamos trabalhar com o Framework Nest.JS, mais utilizado com Typescript, não é comum mas tambem pode ser utilizado com Javascript e trabalha muito bem com Express ou com Fastfy

- Vamos garantir que estamos utilizando o Node 20

`node -v && nvm use 20`

- instalar a interface do Nest.JS de forma global

`npm i -g @nestjs/cli`

- e configurar o Nest.JS na raiz do projeto

`nest new stock-pettech-product`

- vamos instalar as bibliotecas e utilizar o driver ORM Mongoose, porém o Mongo já possui seu próprio driver que poderia ser utilizado.

`npm i @nestjs/mongoose mongoose`

- tudo pronto para começar, vamos criar o modulo stock

`nest generate module stock`

- vamos instalar o zod no projeto para utilizaar o zod validation:

`npm i zod`

- vamos instalar a autenticação para guarda das rotas

`npm i @nestjs/jwt`

- e vamos instalar um módulo pra trabalhar as variáveis de ambiente:

`npm i @nestjs/config`

## Integração do modulo de estoque com o modulo de gestão de produtos ##

- abrir o terminal na pasta pettech e instalar a biblioteca node-fetch para fazer requisições http, assim quando o modulo de gestão criar um novo produto, automaticamente o produto será criado no estoque.

`npm i node-fetch`

## CI/CD com GitHub Actions ##

- criar o Dockerfile no modulo stock (explicado no próprio código)
- criar database cloud [mongoatlas](https://www.mongodb.com/pt-br/atlas)
- anotar os users, keys e urls criados
- database `pettech` user `mhbordalo` key `lred7URPzm2ieeDV`
- database user `pettechstock` Key `rhXnqnrGApZ5T1ir`
- no database clicar em connect/drivers e copiar a url base de conexão

`mongodb+srv://mhbordalo:<db_password>@pettech.iycz4.mongodb.net/?retryWrites=true&w=majority&appName=pettech`

- pode configurar IP aberto em security/networks access <0.0.0.0/0> (não recomendado)
- subir a imagem local da aplicação com a url do database cloud driver

`docker build -t pettech:latest --build-arg="MONGO_URI=mongodb+srv://mhbordalo:lred7URPzm2ieeDV@pettech.iycz4.mongodb.net/?retryWrites=true&w=majority&appName=pettech" --build-arg="JWT_SECRET=pettech" .`

- Compass é um software de conexão Mongo que precisa ser instalado

- menu database/conect/compass copiar a connection string

`mongodb+srv://<db_username>:<db_password>@pettech.iycz4.mongodb.net/`

- abrir o Compass em new connection, colar a url substituindo infos
`mongodb+srv://pettechstock:rhXnqnrGApZ5T1ir@pettech.iycz4.mongodb.net/`

- e subir a nova imagem database

`docker build -t pettech:latest --build-arg="MONGO_URI=mmongodb+srv://pettechstock:rhXnqnrGApZ5T1ir@pettech.iycz4.mongodb.net/pettech" --build-arg="JWT_SECRET=pettech" .`

- testar o container gerado
`docker run --name pettech-stock -p 3010:3010 -d -t pettech:latest`

- acessar o DockerHub e criar o repositório `pettech-stock` privado
- acessar o GitHub para escrever a action no GitHub Actions
