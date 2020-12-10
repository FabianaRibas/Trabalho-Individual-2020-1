FROM node:lts-alpine
# instala um servidor http simples para servir conteúdo estático
RUN npm install -g http-server
WORKDIR /client
# add /app/node_modules/.bin to $PATH
ENV PATH /client/node_modules/.bin:$PATH
COPY . .
RUN npm install
RUN npm install @vue/cli@3.7.0 -g
CMD ["vue-cli-service", "serve"]
# CMD ["npm", "run", "serve"]

