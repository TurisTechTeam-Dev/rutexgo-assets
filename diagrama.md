```mermaid
flowchart TD
    %% Capas principales de Clean Architecture
    subgraph Presentation["Presentation (UI, Widgets, Screens)"]
        direction TB
        P1[auth]
        P2[mission]
        P3[routes]
        P4[profile]
        P5[admin_panel]
        P6[splash]
    end

    subgraph Domain["Domain (Casos de uso, entidades, lógica de negocio)"]
        direction TB
        D1[Use Cases]
        D2[Entities]
        D3[Repositories (Abstracciones)]
    end

    subgraph Data["Data (Repositorios, fuentes de datos, servicios externos)"]
        direction TB
        DA1[Repositorios (Implementaciones)]
        DA2[Data Sources]
        DA3[Models]
    end

    subgraph External["External (Firebase, APIs, Servicios)"]
        direction TB
        E1[Firebase Auth]
        E2[Firestore DB]
        E3[Cloud Storage]
        E4[Analytics]
        E5[GPS / Cámara / Maps]
    end

    %% Dependencias
    P1 --> D1
    P2 --> D1
    P3 --> D1
    P4 --> D1
    P5 --> D1
    P6 --> D1

    D1 --> D2
    D1 --> D3

    D3 --> DA1
    DA1 --> DA2
    DA1 --> DA3

    DA2 --> E1
    DA2 --> E2
    DA2 --> E3
    DA2 --> E4
    DA2 --> E5
```
