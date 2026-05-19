```mermaid
sequenceDiagram
    actor Usuario
    participant AuthWrapper
    participant LoginScreen as Pantalla de inicio de sesión
    participant RegisterScreen as Pantalla de registro
    participant AuthUseCases as Casos de uso de autenticación
    participant HomeScreen as Pantalla principal
    participant CitySelectionScreen as Selector de ciudades
    participant RoutesUseCases as Casos de uso de rutas
    participant RouteSelectionScreen as Selector de rutas
    participant MapNavigationScreen as Pantalla de navegación

    Usuario->>AuthWrapper: Abrir la aplicación
    AuthWrapper->>AuthWrapper: Verificar estado de autenticación

    alt Usuario no autenticado
        AuthWrapper->>LoginScreen: Mostrar pantalla de inicio de sesión
        Usuario->>LoginScreen: Introducir credenciales o acceder al registro

        alt Inicio de sesión
            LoginScreen->>AuthUseCases: login(email, password)
            AuthUseCases-->>LoginScreen: Usuario autenticado
            LoginScreen->>AuthUseCases: checkAdminStatus(uid)
            AuthUseCases-->>LoginScreen: Resultado de validación
            LoginScreen->>HomeScreen: Navegar a la pantalla principal
        else Registro de usuario
            LoginScreen->>RegisterScreen: Abrir pantalla de registro
            Usuario->>RegisterScreen: Completar formulario de registro
            RegisterScreen->>AuthUseCases: register(name, username, email, password)
            AuthUseCases-->>RegisterScreen: Registro completado
            RegisterScreen->>HomeScreen: Navegar a la pantalla principal
        end
    else Usuario autenticado
        AuthWrapper->>HomeScreen: Acceder directamente a la pantalla principal
    end

    Usuario->>HomeScreen: Solicitar exploración de ciudades
    HomeScreen->>CitySelectionScreen: Abrir selector de ciudades
    CitySelectionScreen->>RoutesUseCases: getCities()
    RoutesUseCases-->>CitySelectionScreen: Lista de ciudades disponibles

    Usuario->>CitySelectionScreen: Seleccionar ciudad
    CitySelectionScreen->>RouteSelectionScreen: Abrir selector de rutas
    RouteSelectionScreen->>RoutesUseCases: getRoutesByCity() / getRoutesByCityKeys()
    RoutesUseCases-->>RouteSelectionScreen: Rutas asociadas a la ciudad

    Usuario->>RouteSelectionScreen: Seleccionar ruta
    RouteSelectionScreen->>MapNavigationScreen: Abrir pantalla de navegación
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
flowchart TD
    A([Inicio de la app]) --> B[Inicializar Firebase]
    B --> C[MaterialApp]
    C --> D[AuthWrapper]

    D --> E{¿Hay usuario autenticado?}

    E -- No --> F[SplashScreen]
    F --> G[LoginScreen]
    G --> H{¿Quiere registrarse?}

    H -- Sí --> I[RegisterScreen]
    I --> J[Crear cuenta]
    J --> K[Volver a LoginScreen]

    H -- No --> L[Iniciar sesión]
    L --> M{¿Autenticación correcta?}
    M -- No --> G
    M -- Sí --> N{¿Es admin y entra desde web?}

    E -- Sí --> N

    N -- Sí --> O[AdminPanelScreen]
    O --> P[Gestionar ciudades]
    P --> Q[Gestionar rutas]
    Q --> R[Gestionar puntos de interés]
    R --> S[Gestionar misiones]
    S --> T[Cerrar sesión]
    T --> F

    N -- No --> U[HomeScreen]

    U --> V[ProfileScreen]
    V --> U

    U --> W[CitySelectionScreen]
    W --> X[RouteSelectionScreen]
    X --> Y[MapNavigationScreen]
    Y --> Z[MissionScannerScreen]
    Z --> AA[MonumentInfoScreen]
    AA --> AB[QuizScreen]
    AB --> AC[RouteResultScreen]
    AC --> U

    U --> AD[ExplorerDiaryScreen]
    AD --> U

    U --> AE[Cerrar sesión]
    AE --> F
```

---

```mermaid
flowchart LR
  %% Agrupaciones
  subgraph Sistema_RuteX_Go["Sistema RuteX Go"]
    style Sistema_RuteX_Go fill:#fff8e6,stroke:#e6c07a,stroke-width:1px
    app["Aplicación Móvil<br/>Flutter"]
  end

  subgraph Servicios_Externos["Servicios Externos (Firebase & APIs)"]
    style Servicios_Externos fill:#eef6ff,stroke:#9fbfff,stroke-width:1px
    auth["Firebase Auth<br/>Email / Google"]
    firestore["Cloud Firestore<br/>Base de Datos NoSQL"]
    storage["Cloud Storage<br/>Multimedia / Fotos"]
    maps["APIs de Mapas<br/>Google Maps / OSM"]
  end

  %% Actores
  Turista((Turista))
  Administrador((Administrador))

  %% Relaciones actor -> sistema
  Turista -- "Interacción UI" --> app
  Administrador -- "Gestión de Contenido" --> app

  %% Relaciones app -> servicios externos (con etiquetas)
  app -- "Validación Acceso" --> auth
  app -- "Consulta Rutas / Quizzes" --> firestore
  app -- "CRUD Ciudades / Rutas / POIs" --> firestore
  app -- "Subida de Imágenes" --> storage
  app -- "Carga Mapas Turísticos" --> maps
  app -- "Localización de Puntos" --> maps

  %% Alineación sugerida
  classDef actor fill:#ffffff,stroke:#cfcfcf;
  class Turista,Administrador actor;
```

---

```mermaid
flowchart TB
  %% Actor superior
  actor((Turista / Admin))

  %% Dispositivo móvil (contenedor)
  subgraph Dispositivo_Móvil["Dispositivo Móvil (App Flutter)"]
    style Dispositivo_Móvil fill:#fffce6,stroke:#e6d89a,stroke-width:1px
    UI["Capa de Interfaz<br/>Widgets / UI UX"]
    Logic["Lógica de Negocio<br/>Providers / Casos de Uso"]
    Data["Capa de Datos<br/>Repositorios / SDK Firebase"]
  end

  %% APIs de mapas (a la derecha)
  Maps["APIs de Mapas<br/>Google Maps / OSM"]

  %% Servicios Firebase (contenedor inferior)
  subgraph Servicios_Firebase["Servicios Firebase (BaaS)"]
    style Servicios_Firebase fill:#f7fff3,stroke:#cfe8c6,stroke-width:1px
    Auth["Firebase Auth<br/>Autenticación"]
    Firestore[(Cloud Firestore<br/>Base de Datos NoSQL)]
    Storage["Cloud Storage<br/>Repositorio Imágenes"]
  end

  %% Conexiones principales
  actor -->|Usa la interfaz| UI
  UI -->|Llama a| Logic
  Logic -->|Solicita datos a| Data
  UI -->|Visualiza mapas| Maps

  %% Conexiones desde capa de datos a servicios
  Data -->|Protocolo HTTPS| Auth
  Data -->|Sincronización RealTime| Firestore
  Data -->|Carga de archivos| Storage

  %% Estilos de nodos
  classDef actorStyle fill:#f3f0ff,stroke:#c8bfff,stroke-width:1px;
  classDef boxStyle fill:#f2efff,stroke:#d6c9ff,stroke-width:1px;
  class actor actorStyle;
  class UI,Logic,Data,Auth,Storage boxStyle;
```

---

```mermaid
graph TD
    %% Inicio y Autenticación
    INICIO([INICIO: Iniciar aplicación]) --> InitFirebase[Inicializar Firebase]
    InitFirebase --> CheckSession[Verificar sesión de usuario]
    CheckSession --> IsAuth{¿Hay usuario autenticado?}

    %% Flujo de No Autenticado
    IsAuth -- NO --> SplashLogin[Mostrar pantalla Splash/Login]
    SplashLogin --> WantRegister{¿Usuario quiere registrarse?}
    
    WantRegister -- SÍ --> PantallaRegistro[Pantalla de Registro<br>Ingresar datos: nombre, usuario, email, contraseña]
    PantallaRegistro --> SaveFirebase[Guardar en Firebase]
    SaveFirebase --> GoToLogin[Ir a Login]
    GoToLogin --> LoginOptions
    
    WantRegister -- NO --> LoginOptions[Mostrar opciones de login:<br>- Iniciar sesión con email/contraseña<br>- Iniciar sesión con Google<br>- Recuperar contraseña]
    LoginOptions --> AuthSuccess{¿Autenticación exitosa?}
    AuthSuccess -- NO --> ShowError[Mostrar error]
    
    %% Flujo de Autenticado / Login Exitoso
    IsAuth -- SÍ --> IsAdmin{¿Es usuario web Y<br>es administrador?}
    AuthSuccess -- SÍ --> IsAdmin

    %% Flujo Administrador
    IsAdmin -- SÍ --> AdminPanel[Pantalla de Panel de Administración]
    AdminPanel --> WhatManage{¿Qué desea gestionar?}
    
    WhatManage --> Ciudades[Ciudades<br>CRUD]
    WhatManage --> Rutas[Rutas<br>CRUD]
    WhatManage --> POI[Puntos de interés<br>CRUD]
    WhatManage --> Misiones[Misiones<br>CRUD]
    
    Ciudades --> DoOp[Realizar operación]
    Rutas --> DoOp
    POI --> DoOp
    Misiones --> DoOp
    
    DoOp --> SaveFirestore[Guardar cambios en Firestore/Storage]
    SaveFirestore --> LoopAdmin[Volver a seleccionar opción o cerrar sesión]
    LoopAdmin --> AdminPanel

    %% Flujo Usuario Estándar (Pantalla Home)
    IsAdmin -- NO --> Home[Pantalla Home]
    
    %% Opciones de la Home
    Home --> Op1[Opción 1: Ver perfil]
    Home --> Op2[Opción 2: Explorar rutas]
    Home --> Op3[Opción 3: Realizar misión]
    Home --> Op4[Opción 4: Ver diario del explorador]
    Home --> Op5[Opción 5: Cerrar sesión]

    %% Detalles Opción 1
    Op1 --> Op1_Detalles[Cargar datos del usuario<br>Mostrar información personal<br>Opción para actualizar nombre/avatar]
    Op1_Detalles --> Home

    %% Detalles Opción 2
    Op2 --> Op2_Detalles[Seleccionar ciudad<br>Ver rutas disponibles de esa ciudad<br>Seleccionar una ruta<br>Ver detalles y disponibilidad de la ruta<br>Opción para iniciar misión]
    Op2_Detalles --> Home

    %% Detalles Opción 3
    Op3 --> Op3_Detalles[Ir a escáner QR<br>Escanear código QR del punto de interés<br>Validar que es el punto correcto<br>Mostrar información del monumento<br>Completar misión/reto<br>Hacer quiz si aplica<br>Guardar progreso en Firestore]
    Op3_Detalles --> Home

    %% Detalles Opción 4
    Op4 --> Op4_Detalles[Cargar rutas completadas<br>Seleccionar rutas para incluir en diario<br>Añadir fotos a cada ruta<br>Vista previa del diario<br>Generar y descargar PDF]
    Op4_Detalles --> Home

    %% Detalles Opción 5
    Op5 --> Logout[Cerrar sesión]
    Logout --> ReturnSplash[Volver a Splash/Login]
```
