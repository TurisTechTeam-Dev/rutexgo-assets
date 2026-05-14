```mermaid
flowchart TD
    %% Módulos principales (Features)
    subgraph Features["Módulos (Features)"]
        direction LR
        F1[auth: Login / Register]
        F2[mission: Quiz / QR / Navigation]
        F3[routes: City / Selection]
        F4[profile: Home / Stats]
        F5[admin_panel]
        F6[splash]
        F7[explorer_diary: PDF]
    end

    %% Capas de Clean Architecture
    subgraph Presentation["Presentation"]
        direction TB
        P1[Widgets / Screens]
    end

    subgraph Domain["Domain (Lógica de Negocio)"]
        direction TB
        D1[Use Cases]
        D2[Entities]
        D3[Repository Abstractions]
    end

    subgraph Data["Data"]
        direction TB
        DA1[Repository Implementations]
        DA2[Data Sources]
        DA3[Models]
    end

    %% Servicios externos
    subgraph External["External Services"]
        direction LR
        E1[Firebase Auth]
        E2[Firestore DB]
        E3[Cloud Storage]
        E4[GPS]
        E5[Cámara]
        E6[Maps]
        E7[PDF Generator]
    end

    %% Relaciones entre módulos y capas
    F1 --> P1
    F2 --> P1
    F3 --> P1
    F4 --> P1
    F5 --> P1
    F6 --> P1
    F7 --> P1

    P1 --> D1
    D1 --> D2
    D1 --> D3
    D3 --> DA1
    DA1 --> DA2
    DA1 --> DA3

    %% Relación de Data con servicios externos
    DA2 --> E1
    DA2 --> E2
    DA2 --> E3
    DA2 --> E4
    DA2 --> E5
    DA2 --> E6
    DA2 --> E7
```



```mermaid
erDiagram
    usuarios ||--o| config_rangos : "referencia campo 'rango'"
    usuarios ||--o{ resultado : "referencia campo 'id_usuario'"
    ciudades ||--o{ rutas : "referencia campo 'id_ciudad'"
    rutas ||--o{ puntos_interes : "referencia array 'id_puntos_interes'"
    puntos_interes ||--o| misiones : "referencia campo 'punto_interes_id'"
    rutas ||--o{ resultado : "referencia campo 'id_ruta'"

    usuarios {
        string uid
        string usuario
        string nombre
        string email
        string rango
        int64 puntos
        array rutas_completadas
        timestamp fecha_creacion
        timestamp ultimo_acceso
        boolean isAdmin
        string avatar
    }

    ciudades {
        string nombre
        string provincia
        string imagen
        boolean isActive
    }

    rutas {
        string nombre
        string descripcion
        string id_ciudad
        array id_puntos_interes
        string dificultad
        string duracion
        int puntos_totales
        string imagen
        boolean isActive
    }

    puntos_interes {
        string nombre
        string descripcion
        geopoint localizacion
        string qr_code
        number radio_activacion
        string imagen
    }

    misiones {
        string titulo
        string punto_interes_id
        number puntos_premio
        array_map preguntas
    }

    resultado {
        string id_usuario
        string id_ruta
        string nombre_ruta
        number puntuacion_intento
        number mejor_puntuacion_anterior
        number mejor_puntuacion_guardada
        string tiempo_intento
        int misiones_completadas
        int puntos_interes_visitados
        array Puntos_interes_visitados_nombres
        int total_puntos_interes
        int puntos_totales_posibles
        int respuestas_correctas
        int total_respuestas
        array_map respuestas
        array puntos_interes_saltados
        timestamp fecha_creacion
    }

    config_rangos {
        array_map rangos
    }
```

```mermaid
graph LR
    %% Actores
    U1((Turista))
    U2((Administrador))

    %% Sistema Central
    subgraph "Sistema RuteX Go"
        App[Aplicación Móvil<br/>Flutter]
    end

    %% Servicios Externos
    subgraph "Servicios Externos (Firebase & APIs)"
        Auth[Firebase Auth<br/>Email / Google]
        Firestore[(Cloud Firestore<br/>Base de Datos NoSQL)]
        Storage[Cloud Storage<br/>Multimedia / Fotos]
        Maps[APIs de Mapas<br/>Google Maps / OSM]
    end

    %% Flujo de interacciones del Turista
    U1 -- "Interacción UI" --> App
    App -- "Validación Acceso" --> Auth
    App -- "Consulta Rutas / Quizzes" --> Firestore
    App -- "Carga Mapas Turísticos" --> Maps

    %% Flujo de interacciones del Admin
    U2 -- "Gestión de Contenido" --> App
    App -- "CRUD Ciudades / Rutas / POIs" --> Firestore
    App -- "Subida de Imágenes" --> Storage
    App -- "Localización de Puntos" --> Maps
```

```mermaid
graph TB
    %% Actores
    User((Turista / Admin))

    %% Contenedor Aplicación Móvil
    subgraph "Dispositivo Móvil (App Flutter)"
        UI[Capa de Interfaz<br/>Widgets / UI UX]
        Logic[Lógica de Negocio<br/>Providers / Casos de Uso]
        Data[Capa de Datos<br/>Repositorios / SDK Firebase]
    end

    %% Contenedor Backend
    subgraph "Servicios Firebase (BaaS)"
        Auth[Firebase Auth<br/>Autenticación]
        Firestore[(Cloud Firestore<br/>Base de Datos NoSQL)]
        Storage[Cloud Storage<br/>Repositorio Imágenes]
    end

    %% Servicios Externos
    Maps[APIs de Mapas<br/>Google Maps / OSM]

    %% Flujos de comunicación
    User -- "Usa la interfaz" --> UI
    UI -- "Llama a" --> Logic
    Logic -- "Solicita datos a" --> Data

    Data -- "Protocolo HTTPS" --> Auth
    Data -- "Sincronización RealTime" --> Firestore
    Data -- "Carga de archivos" --> Storage
    UI -- "Visualiza mapas" --> Maps
```
```mermaid
graph TD
    %% Título del Diagrama
    title[DIAGRAMA DE CASOS DE USO DEL SISTEMA]
    style title fill:none,stroke:none,font-weight:bold,font-size:20px

    subgraph AreaGlobal [" "]
        direction LR
        style AreaGlobal fill:none,stroke:none

        %% COLUMNA IZQUIERDA: USUARIO
        Usuario((Usuario Explorador))

        %% COLUMNA CENTRAL: SISTEMA APP
        subgraph SistemaApp ["<< System >> Sistema App Móvil"]
            direction TB
            
            subgraph Autenticacion ["Autenticación"]
                direction TB
                UC_L(Login Email/Google) --- UC_Det(Detección de Rol)
                UC_R(Registro de Nuevo Usuario)
                UC_RC(Recuperar Contraseña)
                UC_LO(Logout)
                UC_L -.->|include| UC_Det
            end

            subgraph HomePerfil ["Home & Perfil"]
                direction TB
                UC_VH(Ver Home)
                UC_EU(Editar Username)
                UC_CA(Cambiar Avatar)
            end

            subgraph Rutas ["Rutas"]
                direction TB
                UC_SC(Selección de Ciudad)
                UC_VD(Ver Disponibilidad)
                UC_IR(Iniciar Ruta)
                UC_VD --> UC_IR
            end

            subgraph Diario ["Diario del Explorador"]
                direction TB
                UC_VR(Visualizar Rutas)
                UC_AF(Añadir Fotos Personales)
                UC_EP(Exportar PDF)
            end

            subgraph Mision ["Misión: Núcleo Gamificado"]
                direction TB
                UC_NM(Navegar con Mapa)
                UC_DL(Detectar Llegada)
                UC_EQ(Escanear QR)
                UC_RQ(Responder Quiz)
                UC_SP(Saltar Punto)
                UC_FR(Finalizar Ruta)
                UC_VI(Ver Info Monumento)
                UC_EQ -.->|extend| UC_VI
            end
            
            UC_IR --> UC_NM
        end

        %% COLUMNA DERECHA: PANEL ADMIN + ADMIN
        subgraph ColumnaAdmin [" "]
            direction TB
            style ColumnaAdmin fill:none,stroke:none
            
            subgraph PanelWeb ["<< System >> Panel Admin Web"]
                direction TB
                subgraph PanelSolo ["Panel Admin - Solo Web"]
                    direction TB
                    UC_CC(CRUD Ciudades)
                    UC_CR(CRUD Rutas)
                    UC_CP(CRUD POIs y Misiones)
                    UC_MI(Mapa Interactivo)
                end
            end
            
            Admin((Administrador))
        end
    end

    %% BLOQUE INFERIOR: ACCESIBILIDAD
    subgraph Accesibilidad ["<< System >> Accesibilidad"]
        UC_Audio(Transversal: AudioGuideWidget)
    end

    %% CONEXIONES
    Usuario --- Autenticacion
    Usuario --- HomePerfil
    Usuario --- Rutas
    Usuario --- Diario
    
    PanelSolo --- Admin
    
    Usuario -.->|use| UC_Audio
    Admin -.->|use| UC_Audio

    %% ESTILOS PARA LOOK PROFESIONAL
    classDef default font-family:Arial, font-size:12px
    style SistemaApp fill:#ffffff,stroke:#333,stroke-width:2px
    style PanelWeb fill:#ffffff,stroke:#333,stroke-width:2px
    style Accesibilidad fill:#ffffff,stroke:#333,stroke-width:2px
```
