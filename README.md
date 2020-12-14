# Trabalho Individual - GCES - 2020/1

A Gestão de Configuração de Software é parte fundamental no curso de GCES, e dominar os conhecimentos de configuração de ambiente, containerização, virtualização, integração e deploy contínuo tem se tornado cada vez mais necessário para ingressar no mercado de trabalho.

Para exercitar estes conhecimentos, você deverá aplicar os conceitos estudados ao longo da disciplina no produto de software contido neste repositório.

O sistema se trata de uma aplicação Web, cuja funcionalidade consiste na pesquisa e exibição de perfis de usuários do GitHub, que é composta de:

- Front End escrito em Javascript, utilizando os frameworks Vue.JS e Quasar;
- Back End escrito em Ruby on Rails, utilizado em modo API;
- Banco de Dados PostgreSQL;

Para executar a aplicação na sua máquina, basta seguir o passo-a-passo descrito no arquivo [Descrição e Instruções](Descricao-e-Instrucoes.md).

## Critérios de avaliação

### 1. Containerização

A aplicação deverá ter seu ambiente completamente containerizado. Desta forma, cada subsistema (Front End, Back End e Banco de Dados) deverá ser isolado em um container individual.

Deverá ser utilizado um orquestrador para gerenciar comunicação entre os containers, o uso de credenciais, networks, volumes, entre outras configurações necessárias para a correta execução da aplicação.

Para realizar esta parte do trabalho, recomenda-se a utilização das ferramentas:

- Docker versão 17.04.0+
- Docker Compose com sintaxe na versão 3.2+

### 2. Integração contínua

Você deverá criar um 'Fork' deste repositório, onde será desenvolvida sua solução. Nele, cada commit submetido deverá passar por um sistema de integração contínua, realizando os seguintes estágios:

- Build: Construção completa do ambiente;
- Testes: Os testes automatizados da aplicação devem ser executados;
- Coleta de métricas: Deverá ser realizada a integração com algum serviço externo de coleta de métricas de qualidade;

O sistema de integração contínua deve exibir as informações de cada pipeline, e impedir que trechos de código que não passem corretamente por todo o processo sejam adicionados à 'branch default' do repositório.

Para esta parte do trabalho, poderá ser utilizada qualquer tecnologia ou ferramenta que o aluno desejar, como GitlabCI, TravisCI, CircleCI, Jenkins, CodeClimate, entre outras.

### 3. Deploy contínuo (Extra)

Caso cumpra todos os requisitos descritos acima, será atribuída uma pontuação extra para o aluno que configure sua pipeline de modo a publicar a aplicação automaticamente, sempre que um novo trecho de código seja integrado à branch default.

## Resolução do Trabalho Individual - GCES - 2020/1

### 1. Solução Containerização

A aplicação teve seu ambiente containerizado. Cada subsistema (Back End, Front End e Banco de Dados) possui seu arquivo de Dockerfile.

Arquivo: **./api/Dockerfile**

```bat
FROM ruby:2.5.7
ENV BUNDLER_VERSION=2.1.4
CMD echo "I'm running a ruby 2.5.7 image"

RUN apt-get update -qq && apt-get install -y nodejs postgresql-client
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev

WORKDIR /api

COPY ./api/Gemfile .
COPY ./api/Gemfile.lock .

RUN gem install bundler
RUN bundle install

COPY ./api/ /api

COPY ./api/entrypoint.sh /usr/bin/entrypoint.sh

RUN chmod +x /usr/bin/entrypoint.sh

# Start the main process.
ENTRYPOINT ["entrypoint.sh"]
```

Arquivo: **./client/Dockerfile**

```bat
FROM node:14
# instala um servidor http simples para servir conteúdo estático
RUN npm install -g http-server
WORKDIR /client
COPY ./client/package.json .
COPY ./client/yarn.lock .
# add /app/node_modules/.bin to $PATH
ENV PATH /client/node_modules/.bin:$PATH

RUN yarn global add @vue/cli@4.4.6
RUN yarn install
COPY . .
CMD ["npm", "run", "serve"]
```

Foi utilizado um orquestrador para gerenciar comunicação entre os containers

Arquivo: **docker-compose.yml**

```bat
version: "3.8"
services:
  db:
    container_name: db
    image: postgres:13.1-alpine
    restart: always
    ports:
      - 5432:5432
    networks:
      - aplication_production
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password

  client:
    container_name: client
    restart: always
    networks:
      - aplication_production
    build:
      context: .
      dockerfile: ./client/Dockerfile
      cache_from:
        - node:14
    ports:
      - 8080:8080
    volumes:
      - ./client:/client
      - node_modules:/client/node_modules

  api:
    container_name: api
    networks:
      - aplication_production
    build:
      context: .
      dockerfile: ./api/Dockerfile
      cache_from:
        - ruby:2.5.7
    command: bundle exec rails s -p 3000 -b '0.0.0.0'
    env_file: .env
    ports:
      - 3000:3000
    volumes:
      - ./api/:/api
    links:
      - db
    depends_on:
      - db

networks:
  aplication_production:
    driver: bridge

volumes:
  node_modules:
```
Para executar o container, basta digitar no terminal o seguinte comando:
```bat
$ docker-compose up --build
```

### 2. Solução Integração contínua

Esta solução foi desenvolvida neste fork do projeto original. Nele, cada push, PR ou deploy realizado é submetido a um sistema de integração contínua, configurado no **Github Actions**, realizando os seguintes estágios/jobs:

- Build: Construção completa do ambiente;

```bat
name: CI

on:
  push:
  pull_request:
    branches: [ master ]
  deployment:
    branches: [ master ]
    
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: build api containers
        run: docker-compose up --build -d
```

- Testes: Os testes automatizados da api e client são executados. Para executar os teste separadamente do CI basta digitar os camando abaixo, no terminal no diretorio raiz:

Para o client:
```bash
$ docker-compose run client yarn run test:unit
```
Para a api:
```bash
docker-compose run api bundle exec rake test
```

- Coleta de métricas: É realizada a integração com **Sonar Cloud** para coleta de métricas de qualidade de ambos os diretórios;

```bat
api_test:
    needs: [ build ]
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
    - uses: actions/checkout@v2
      
    - name: Build api container
      run:  |
        docker-compose up --detach --build db api
        
    - name: Run api tests
      run: |
        docker-compose run api bundle exec rake test
    - name: API Sonar coverage scan
      uses: SonarSource/sonarcloud-github-action@master
      with:
        projectBaseDir: api
      env:
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN_API }}
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
      
  client_test:
    needs: [ build ]
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
    - uses: actions/checkout@v2

    - name: Build client container
      run: |
        docker-compose up --detach --build client
    - name: Run client tests
      run: |
        docker-compose run client yarn run test:unit
    - name: Client Sonar coverage scan
      uses: SonarSource/sonarcloud-github-action@v1.4
      with:
        projectBaseDir: client
      env:
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN_CLIENT }}
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

```

### 3. Solução Deploy contínuo (Extra)

Não foi possível completar tarefa a tempo de entrega.
