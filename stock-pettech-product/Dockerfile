# Dockerfile com configuração de desenvolvimento local

# informar a imagem base (oficial) do Node.js (alpine = versão reduzida)
FROM node:20-alpine

# criar o diretório de trabalho dentro do contêiner (pasta principal)
WORKDIR /usr/app

# copiar os arquivos do projeto para dentro do container (./ pega o WORKDIR)
COPY package.json ./

# executar a instalação das dependencias
RUN npm install

# copiar o codigo fonte (. . copia tudo da pasta do código para a pasta do container)
COPY . .

# instalar as dependencias do código (quantas forem necessárias)
RUN npm install -g pnpm

# Compilar o código TypeScript (gerar arquivo de produção)
RUN pnpm build

# Expor a porta utilizada pelo seu aplicativo Nest.js (padrão = porta 3000)
EXPOSE 3010

# Iniciar o aplicativo quando subir o contêiner [executar, onde é executado]
CMD ["node", "dist/main"]