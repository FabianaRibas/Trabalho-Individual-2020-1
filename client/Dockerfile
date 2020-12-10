FROM node:lts-alpine

# instala um servidor http simples para servir conteúdo estático
RUN npm install -g http-server

# seta diretorio
WORKDIR /client

# add /app/node_modules/.bin to $PATH
ENV PATH /client/node_modules/.bin:$PATH

# instala dependências do projeto
COPY . .

# instala dependencias do projeto
RUN npm install
RUN npm install @vue/cli@3.7.0 -g

# inicia app
CMD ["vue-cli-service", "serve"]
# CMD ["npm", "run", "serve"]

