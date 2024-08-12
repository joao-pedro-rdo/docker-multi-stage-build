# docker-multi-stage-build

Repositório para documentar aprendizados sobre multi-stage build com Docker aprendidos no curso da formação DevOps Pro.

## Índice

- [Introdução](#introdução)
- [O que é Multi-Stage Build](#o-que-é-multi-stage-build)
- [Benefícios](#benefícios)
- [Exemplo de Uso](#exemplo-de-uso)
- [Como Executar](#como-executar)
- [Referências](#referências)

## Introdução

Este repositório contém exemplos e explicações sobre como utilizar a técnica de multi-stage build com Docker para otimizar a construção de imagens Docker, reduzindo o tamanho final da imagem e melhorando a segurança e a eficiência do processo de build.

## O que é Multi-Stage Build

Multi-stage build é uma técnica que permite utilizar múltiplos estágios em um único Dockerfile. Cada estágio pode ter uma função específica, como compilar o código, rodar testes e, finalmente, criar a imagem de produção. Isso permite que apenas os artefatos necessários sejam incluídos na imagem final, resultando em uma imagem mais leve e segura.

## Benefícios

- **Redução do tamanho da imagem**: Apenas os artefatos necessários são incluídos na imagem final.
- **Melhoria na segurança**: Dependências e ferramentas de build não são incluídas na imagem de produção.
- **Facilidade de manutenção**: Dockerfile mais organizado e modular.

## Exemplo de Uso

Aqui está um exemplo básico de um Dockerfile utilizando multi-stage build:

```dockerfile
# Estágio 1: Build
FROM golang:1.22-alpine as builder
WORKDIR /build
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .

# Estágio 2: Produção
FROM alpine:latest as app
WORKDIR /app
# Preciso copiar da minha imagem anterior
COPY --from=builder /build/main .
CMD ["./main"]
```

Há tambem um arquivo dockerfile-simples que é um exemplo de um Dockerfile simples que não utiliza multi-stage build, para servir de comparação:

```dockerfile
FROM golang:1.22-alpine 
WORKDIR /app
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o main .
CMD ["./main"]
```


## Como Executar

Para construir e rodar a imagem Docker utilizando o Dockerfile acima, siga os passos abaixo:
Ultilizei nomeclatura padrao do docker registry para nomear a imagem `joaoprdo/...`.
1. Construa a imagem Docker:
    ```sh
     docker build -t joaoprdo/go-multi-stage:multi-stage -f dockerfile-multistage .
    ```
    Para construir a imagem simples sem multi-stage:
    
    ```sh
     docker build -t joaoprdo/go-multi-stage:simples -f dockerfile-simples .
    ```

2. Rode o container:
    ```sh
    docker container run -p 8080:8080 joaoprdo/go-multi-stage:multi-stage
    ```
   
    Para rodar a imagem simples:
    
    ```sh
    docker container run -p 8080:8080 joaoprdo/go-multi-stage:simples
    ```
    Acesse a aplicação no navegador em `http://localhost:8080`.

3. Para comparar o tamanho das imagens, rode o comando abaixo:
    ```sh
    docker images ls
    ```
    ```sh
    REPOSITORY                  TAG               IMAGE ID       SIZE
    joaoprdo/go-multi-stage     multi-stage       f314b1cb5e2f      14.8MB
    joaoprdo/go-multi-stage     simples           89b44a6ecd1b      303MB
    ```
    Você verá que a imagem com multi-stage build é menor que a imagem sem multi-stage build.

## Referências

- [Documentação Oficial do Docker sobre Multi-Stage Builds](https://docs.docker.com/develop/develop-images/multistage-build/)
