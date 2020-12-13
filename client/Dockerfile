FROM node:14
# instala um servidor http simples para servir conteúdo estático
RUN npm install -g http-server
WORKDIR /client
# add /app/node_modules/.bin to $PATH
ENV PATH /client/node_modules/.bin:$PATH
RUN npm install -g @vue/cli@3.7.0
RUN npm install
COPY . .
CMD ["npm", "run", "serve"]

