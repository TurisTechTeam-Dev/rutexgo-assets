```mermaid
sequenceDiagram
    autonumber
    title Flujo Completo: Autenticación, Selección y Misiones (RutexGo)

    %% Declaración global de participantes para evitar errores de parseo
    participant Usuario
    participant AuthWrapper
    participant AuthUI as Login/Register
    participant Home as HomeScreen
    participant CitySel as CitySelection
    participant RouteSel as RouteSelection
    participant Nav as Navigator
    participant MapScreen
    participant TripProv as TripProvider
    participant Scanner as MissionScanner
    participant MissionUC as UseCases
    participant MissionRepo as Repository
    participant Monument as MonumentInfo
    participant Quiz
    participant Result as RouteResult

    box rgb(240, 248, 255) Autenticación
        Usuario->>AuthWrapper: Abrir App
        Note over AuthWrapper: verificar sesión
        alt No autenticado
            AuthWrapper->>AuthUI: Mostrar Login
            Usuario->>AuthUI: Credenciales
            AuthUI-->>AuthWrapper: Éxito
        end
        AuthWrapper->>Home: Ir a Home
    end

    box rgb(245, 245, 220) Selección de Destino
        Usuario->>Home: Elegir Ciudad
        Home->>Nav: pushNamed(citySelection)
        Nav->>CitySel: Abrir pantalla
        Usuario->>CitySel: Elige Ciudad
        CitySel->>Nav: pushNamed(routeSelection)
        Nav->>RouteSel: Abrir pantalla
        Usuario->>RouteSel: Elige Ruta
        RouteSel->>Nav: pushNamed(mapNavigation)
    end

    box rgb(230, 230, 250) Mapa y GPS
        Nav->>MapScreen: Abrir mapa
        MapScreen->>TripProv: Init Provider
        TripProv->>MissionUC: getPoints(routeId)
        MissionUC->>MissionRepo: Fetch POIs
        MissionRepo-->>MissionUC: Data
        MissionUC-->>TripProv: POIs List
        Note over TripProv: Iniciar GPS y cálculos
        TripProv-->>MapScreen: notifyListeners()
    end

    box rgb(240, 255, 240) Llegada y Escaneo
        Note over TripProv: Detectar proximidad
        TripProv-->>MapScreen: reached = true
        MapScreen->>Usuario: ArrivalBottomSheet
        Usuario->>MapScreen: Elegir "Escanear"
        MapScreen->>Nav: pushNamed(scanner)
        Nav->>Scanner: Abrir Scanner
        Scanner->>MissionUC: scan(qr)
        MissionUC->>MissionRepo: getMissionData
        MissionRepo-->>MissionUC: Mission Obj
        MissionUC-->>Scanner: ScanResult
    end

    box rgb(255, 250, 205) Misión y Quiz
        Scanner->>Nav: push(monumentInfo)
        Nav->>Monument: Abrir pantalla
        Usuario->>Monument: "Empezar misión"
        Monument->>Nav: push(quiz)
        Nav->>Quiz: Abrir Quiz
        Usuario->>Quiz: Responder
        Quiz-->>Monument: pointCompleted
        Monument-->>Scanner: pop()
        Scanner-->>MapScreen: Volver al mapa
    end

    box rgb(255, 245, 238) Finalización
        MapScreen->>TripProv: markCompleted()
        alt Quedan puntos
            TripProv->>TripProv: Siguiente POI
        else Ruta fin
            TripProv->>MissionUC: saveProgress()
            MissionUC->>MissionRepo: Firestore Sync
            TripProv-->>Result: Mostrar Resultados
        end
    end
```




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
graph LR
    %% ESTILOS
    classDef actor fill:#fff,stroke:#333,stroke-width:2px
    classDef usecase fill:#f9f9f9,stroke:#333,stroke-width:1px

    %% ACTORES
    Usuario((Usuario Explorador)):::actor
    Admin((Administrador)):::actor

    %% --- COLUMNA IZQUIERDA: APP MÓVIL ---
    subgraph SistemaApp ["<< System >> Sistema App Móvil"]
        direction TB
        
        subgraph Autenticacion ["Autenticación"]
            direction TB
            A1(Login Email/Google)
            A2(Detección de Rol)
            A3(Registro de Nuevo Usuario)
            A4(Recuperar Contraseña)
            A5(Logout)
            A1 -.->|include| A2
        end

        subgraph HomePerfil ["Home & Perfil"]
            direction TB
            H1(Ver Home)
            H2(Editar Username)
            H3(Cambiar Avatar)
        end

        subgraph Rutas ["Rutas"]
            direction TB
            R1(Selección de Ciudad)
            R2(Ver Disponibilidad)
            R3(Iniciar Ruta)
            R2 --> R3
        end

        subgraph Mision ["Misión: Núcleo Gamificado"]
            direction TB
            M1(Navegar con Mapa)
            M2(Detectar Llegada al POI)
            M3(Escanear QR y Validar)
            M4(Responder Quiz)
            M5(Saltar Punto Opcional)
            M6(Finalizar Ruta)
            M7(Ver Info Monumento)
            M3 -.->|extend| M7
        end
        
        %% Uniones de flujo internas de la App
        R3 --> M1
    end

    %% --- COLUMNA DERECHA: PANEL ADMIN ---
    subgraph SistemaAdmin ["<< System >> Panel Admin Web"]
        direction TB
        subgraph PanelSolo ["Panel Admin - Solo Web"]
            direction TB
            P1(CRUD Ciudades)
            P2(CRUD Rutas)
            P3(CRUD POIs y Misiones)
            P4(Mapa Interactivo de POIs)
        end
    end

    %% --- BLOQUE ACCESIBILIDAD ---
    subgraph SistemaAcc ["<< System >> Accesibilidad"]
        Acc(Transversal: AudioGuideWidget)
    end

    %% --- CONEXIONES DE ACTORES ---
    Usuario --- A1
    Usuario --- A3
    Usuario --- H1
    Usuario --- R1
    
    P1 --- Admin
    P2 --- Admin
    P3 --- Admin
    P4 --- Admin

    %% Relación de uso transversal (Líneas punteadas largas)
    Usuario -.->|use| Acc
    Admin -.->|use| Acc

    %% ORDENAMIENTO DE COLUMNAS
    SistemaApp ~~~ SistemaAdmin
    SistemaAdmin ~~~ SistemaAcc
```
