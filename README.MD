# TadsPlayer

Plataforma modular de armazenamento e organização musical baseada em arquitetura de microsserviços, desenvolvida como alternativa leve e independente aos serviços de streaming *mainstream*.

## 1. Domínio e Propósito
* **Domínio:** Tocador de música e gestão de acervos digitais.
* **Propósito:** Permitir o cadastro de metadados musicais (artistas, álbuns e faixas), a organização de músicas em playlists personalizadas e o acompanhamento do histórico de reprodução e audiência de cada faixa.

## 2. Arquitetura do Sistema

O sistema é dividido em **3 microsserviços independentes** desenvolvidos em **Java + Spring Boot**, adotando o padrão *Database per Service* (bancos isolados) e comunicação híbrida (síncrona via HTTP `RestClient` e assíncrona via `RabbitMQ`).

```mermaid
graph LR
    Client([Cliente / Postman])

    subgraph "Microsserviços (Spring Boot)"
        Cat[Catalogue Service<br/>Porta 8081]
        Lib[Library Service<br/>Porta 8082]
        Play[Playback Service<br/>Porta 8083]
    end

    subgraph "Mensageria"
        RMQ[[RabbitMQ<br/>Fila: track.played.queue]]
    end

    subgraph "Bancos de Dados Isolados"
        DB1[(Catalogue DB)]
        DB2[(Library DB)]
        DB3[(Playback DB)]
    end

    Client -->|HTTP / REST| Cat
    Client -->|HTTP / REST| Lib
    Client -->|HTTP / REST| Play

    Lib -->|Síncrono: RestClient<br/>GET /tracks/id| Cat
    Play -->|Síncrono: RestClient<br/>GET /tracks/id| Cat

    Play -.->|Assíncrono: Publica evento<br/>track.played| RMQ
    RMQ -.->|Assíncrono: Consome evento<br/>Incrementa playCount| Cat

    Cat --- DB1
    Lib --- DB2
    Play --- DB3