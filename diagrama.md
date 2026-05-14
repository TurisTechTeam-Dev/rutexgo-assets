```mermaid
sequenceDiagram
    autonumber
    title Flujo Completo: Autenticación, Selección y Misiones (RutexGo)

    actor U as Usuario
    participant AW as AuthWrapper
    participant UI as Login/Register
    participant H as HomeScreen
    participant CS as CitySelection
    participant RS as RouteSelection
    participant N as Navigator
    participant M as MapScreen
    participant T as TripProvider
    participant S as Scanner
    participant UC as MissionUseCases
    participant R as MissionRepository
    participant Mon as MonumentInfo
    participant Q as Quiz
    participant Res as RouteResult

    box rgb(240, 248, 255) Autenticación
        U->>AW: Abrir App
        Note over AW: verificar sesión (authStateChanges)
        alt No autenticado
            AW->>UI: Mostrar Login/Registro
            U->>UI: Ingresar credenciales
            UI-->>AW: Login exitoso
        end
        AW->>H: Navegar a HomeScreen
    end

    box rgb(245, 245, 220) Selección de Destino
        U->>H: Elegir Ciudad
        H->>N: pushNamed(citySelection)
        N->>CS: Abrir CitySelectionScreen
        U->>CS: Seleccionar Ciudad
        CS->>N: pushNamed(routeSelection, cityKeys)
        N->>RS: Abrir RouteSelectionScreen
        U->>RS: Seleccionar Ruta
        RS->>N: pushNamed(mapNavigation, routeId)
    end

    box rgb(230, 230, 250) Inicialización del Mapa
        N->>M: Abrir MapNavigationScreen
        M->>T: Inicializar Provider
        T->>UC: executeGetPointsForRoute(routeId)
        UC->>R: Obtener puntos de ruta
        R-->>UC: Lista POIs
        UC-->>T: Puntos de Interés
        Note over T: Iniciar GPS y calcular ruta
        T-->>M: notifyListeners()
    end

    box rgb(240, 255, 240) Interacción en POI
        Note over T: Detectar proximidad al POI
        T-->>M: hasReachedDestination = true
        M->>U: Mostrar ArrivalBottomSheet
        U->>M: Elegir "Escanear Misión"
        M->>N: pushNamed(missionQrScanner)
        N->>S: Abrir MissionScannerScreen
    end

    box rgb(255, 250, 205) Resolución de Misión
        S->>UC: executeScan(qrCode)
        UC->>R: getPointByQr + getMission
        R-->>UC: Datos de Misión
        UC-->>S: MissionScanResult
        S->>N: pushNamed(monumentInfo)
        N->>Mon: Abrir MonumentInfoScreen
        U->>Mon: Pulsar "Empezar misión"
        Mon->>N: pushNamed(quiz)
        N->>Q: Abrir QuizScreen
        U->>Q: Responder preguntas
        Q-->>Mon: pointCompleted
        Mon-->>S: return pointCompleted
        S-->>M: Volver al mapa
    end

    box rgb(255, 245, 238) Cierre y Guardado
        M->>T: markCurrentPoiAsCompleted()
        alt Quedan puntos pendientes
            T->>T: Recalcular siguiente POI
            T-->>M: Continuar ruta
        else Ruta completada
            T->>UC: saveBestRouteProgress()
            UC->>R: Guardar progreso en Firestore
            T->>UC: saveRouteResult()
            UC->>R: Guardar récord final
            T-->>Res: Navegar a RouteResultScreen
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
