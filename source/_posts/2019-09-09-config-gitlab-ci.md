---
title: Configuração do gitlab-ci
tags:
    - gitlab
    - ci
categories:
    - infraestrutura
---

Algumas coisas me perturbam bastante, dentre elas o fato de não ter certeza de
que vou ou não qubrar qualquer parte do projeto em que estou trabalhando. Para
evitar esse tipo de problema, nada como uma ferramenta que realiza **Integração Contínua**
e aumenta a certeza de não estar acabando com o dia de ninguém por quebrar aquele
projeto que está rodando lindamente.

Na minha busca por alguma ferramenta legal, encontrei o [gitlab-ci](https://docs.gitlab.com/ee/ci/)
que é uma ferramenta que me fez brilhar os olhos, dada sua facilidade de implementação
e o custo zero para sua adoção.

Várias equipes em que já atuei junto, utilizam o **gitlab.com** como VCS, que já traz
o **gitlab-ci** como ferramenta de CI padrão, mas nunca tinha visto ninguém rodar
com todo o poder que essa ferramenta traz.

### Requisitos para utilização do gitlab-ci

Para este texto, vou usar como referência um projeto em Python que utiliza Django
e DRF, além da exigência do PGSQL como DB.

O primeiro passo é criar sua imagem **Docker** (Se não trabalha com Docker, não sei como
te ajudar), pra isso estou usando o *Dockerfile* que segue:

```Dockerfile
FROM python:alpine

ENV PYTHONUNBUFFERED 1

LABEL Name=test_backend Version=0.0.1
EXPOSE 8000

COPY ./requirements.txt /requirements.txt
RUN pip install -r requirements.txt

RUN mkdir /app
WORKDIR /app
COPY ./app /app

RUN adduser -D user
USER user
```

Com essa imagem em mãos e tendo a certeza de que o processo de **build da mesma está em ordem**,
no menu lateral do projeto no gitlab, acesso o menu *Container Registry*.
Este menu permite criar uma ou várias imagens Docker para o projeto. Através
desta imagem será possível rodar todo o processo de pipeline.

### Configuração do .gitlab-ci.yml

**Este arquivo funciona como um orquestrador**. Nele se define todos os **jobs** que serão
utilizados na pipeline, **scripts** que vão ser rodados, scripts para rodar antes e
depois dos trabalhos definidos e etc. Mais informações sobre essas variáveis do
`.gitlab-ci.yml` podem ser acessadas [aqui](https://docs.gitlab.com/ee/ci/variables/).

Além dos jobs, scripts e variáveis de ambiente, é possível determinar
**dependências de outras imagens**. Como neste caso é necessário rodar o Postgres
junto com nosso projeto, automaticamente o `.gitlab-ci-yml` busca a última versão
da imagem solicitada e a agrega a pipeline como **service**, a.k.a serviço de dependência
externa.

Observa nosso arquivo `.gitlab-ci.yml`:

```yml
image: registry.gitlab.com/vicentimartins/backend-test:latest

services:
  - postgres:9.6

variables:
  POSTGRES_DB: "my_custom_db"
  POSTGRES_USER: "postgres"
  POSTGRES_PASSWORD: "secret"
  PGDATA: "/var/lib/postgresql/data"
  POSTGRES_INITDB_ARGS: "--encoding=UTF8 --data-checksums"

stages:
  - build
  - test

build:
  stage: build
  script:
    - pip install -r requirements.txt

test:
  stage: test
  script:
  - python manage.py test
```

Agora, se você é daquelas pessoas que **não abre mão de implementar seus testes** para
toda tarefa solicitada, quando você realizar algum *Merge Request* ou *Pull Request*
verá que o seu projeto está passando pela pipeline definida e rodando todos os testes
para minimizar erros ocorridos por novas integrações de partes do projeto atual.

Caso algum teste falhe, a pipeline retorna 0 (zero) e a integração não é permitida. Fica
a cargo do dev resolver o problema encontrado e propor novo *MR/PR*. Isso garante que
nenhum erro, facilmente detectável por teste, vai quebrar o projeto atual.

### Palavras finais

Além da aplicação mostrada neste texto, é possível adicionar outras ferramentas para
os mais diversos fins, como por exemplo, ferramentas de qualidade de código (QA). Mas
esta abordagem fica para um próximo texto.

Bem, essa foi minha mais nova tentativa de ajudar a comunidade. Peço que dêem feed
back nos comentários e que possam utilizar mais um pouco desta ferramenta maravilhosa.

Forte abraço!
