flowchart TD
    %% Definición de la App Móvil
    subgraph Mobile ["App Móvil RuteX Go (Flutter)"]
        direction TB
        
        subgraph Layers ["Arquitectura por Capas"]
            direction LR
            P[Presentation] --> D[Domain]
            DT[Data] --> D
        end

        subgraph Modules ["Módulos (Features)"]
            direction TB
            M1[auth: Login / Register]
            M2[mission: Quiz / QR / Navigation]
            M3[routes: City / Selection]
            M4[profile: Home / Stats]
            M5[admin_panel]
            M6[splash]
        end
        
        Services[Servicios: GPS / Cámara / Maps]
    end

    %% Definición del Backend
    subgraph Backend ["Infraestructura Firebase"]
        direction TB
        Auth[Firebase Auth]
        FS[Firestore DB]
        ST[Cloud Storage]
        AN[Analytics]
    end

    %% Relaciones de flujo
    Modules --> Layers
    Layers --> Services
    Services --> Auth
    Services --> FS
    Services --> ST
    Layers -.-> AN
