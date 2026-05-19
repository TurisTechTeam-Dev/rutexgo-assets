```mermaid
graph TD
    %% Roles
    Admin[Administrador]
    User[Usuario]

    %% Jerarquía: El admin es un tipo de usuario
    Admin -->|Tiene rol de| User

    %% Casos de Uso Generales (Para ambos)
    subgraph General [Casos de Uso Generales]
        direction TB
        UC1[1. Autenticarse]
        UC2[2. Gestionar perfil]
        UC3[3. Explorar rutas]
        UC4[4. Realizar misión]
        UC5[5. Consultar diario]
    end

    %% Acceso Exclusivo
    subgraph Exclusivo [Acceso Exclusivo Admin]
        UC6[6. Administrar contenido]
    end

    %% Conexiones
    User --> General
    Admin --> Exclusivo

    %% Estilos
    style Admin fill:#fff3e0,stroke:#f57c00
    style User fill:#e3f2fd,stroke:#1565c0
    style Exclusivo fill:#ffebee,stroke:#c62828
```



***Diagrama de Secuencia 1***
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

---

***Diagrama de Secuencia 2***

```mermaid
sequenceDiagram
    actor Usuario
    participant MapNavigationScreen as Pantalla de navegación
    participant TripSimulationProvider as Gestor de ruta
    participant ArrivalBottomSheet as Aviso de llegada
    participant MissionScannerScreen as Escáner QR
    participant MissionUseCases as Casos de uso de misiones
    participant MissionRepository as Repositorio de misiones
    participant MonumentInfoScreen as Detalle del monumento
    participant QuizScreen as Cuestionario
    participant RouteResultScreen as Resultado de la ruta

    TripSimulationProvider->>TripSimulationProvider: Inicializar GPS y calcular ruta
    TripSimulationProvider->>MissionUseCases: executeGetPointsForRoute(routeId)
    MissionUseCases->>MissionRepository: Obtener puntos de interés de la ruta
    MissionRepository-->>MissionUseCases: Lista de puntos de interés
    MissionUseCases-->>TripSimulationProvider: Puntos cargados

    TripSimulationProvider->>TripSimulationProvider: Detectar proximidad al punto de interés
    TripSimulationProvider-->>MapNavigationScreen: Indicar llegada al destino

    MapNavigationScreen->>ArrivalBottomSheet: Mostrar opciones de acción
    Usuario->>ArrivalBottomSheet: Elegir escanear misión

    ArrivalBottomSheet->>MissionScannerScreen: Abrir escáner QR
    MissionScannerScreen->>MissionUseCases: executeScan(qrCode)
    MissionUseCases->>MissionRepository: getPointByQr(qrCode)
    MissionRepository-->>MissionUseCases: Punto de interés

    MissionUseCases->>MissionRepository: getMissionByPointId(point.id)
    MissionRepository-->>MissionUseCases: Misión asociada
    MissionUseCases-->>MissionScannerScreen: Resultado del escaneo

    MissionScannerScreen->>MonumentInfoScreen: Mostrar información del monumento
    MonumentInfoScreen->>QuizScreen: Iniciar cuestionario

    Usuario->>QuizScreen: Responder preguntas
    QuizScreen-->>MonumentInfoScreen: Resultado de la misión
    MonumentInfoScreen-->>MissionScannerScreen: Confirmación de punto completado
    MissionScannerScreen-->>MapNavigationScreen: Retornar a la navegación

    MapNavigationScreen->>TripSimulationProvider: markCurrentPoiAsCompleted()

    alt Existen puntos pendientes
        TripSimulationProvider->>TripSimulationProvider: Recalcular siguiente punto de interés
        TripSimulationProvider-->>MapNavigationScreen: Continuar navegación
    else Ruta finalizada
        TripSimulationProvider->>MissionUseCases: getRouteName(routeId)
        MissionUseCases-->>TripSimulationProvider: Nombre de la ruta

        TripSimulationProvider->>MissionUseCases: saveBestRouteProgress(...)
        MissionUseCases->>MissionRepository: Persistir progreso de la ruta
        MissionRepository-->>MissionUseCases: Confirmación de guardado

        opt Guardado detallado del resultado
            TripSimulationProvider->>MissionUseCases: saveRouteResult(RouteResultRecord)
            MissionUseCases->>MissionRepository: Persistir resultado final
            MissionRepository-->>MissionUseCases: Confirmación de guardado
        end

        TripSimulationProvider->>RouteResultScreen: Mostrar resultado final
    end
```

---

***Diagrama de Flujo***
```mermaid
flowchart TD
    A[Inicio: Usuario abre la app] --> B{¿Sesión activa?}

    subgraph F1["Fase 1: Autenticación"]
        direction TB
        C[Mostrar autenticación]
        D{¿Acción?}
        E[Validar credenciales]
        F[Registrar usuario]
        G{¿Autenticación correcta?}
        H{¿Registro correcto?}
        I[Mostrar error]
        J[Acceder a pantalla principal]

        C --> D
        D -- Iniciar sesión --> E --> G
        D -- Registrarse --> F --> H
        G -- No --> I --> C
        H -- No --> I
        G -- Sí --> J
        H -- Sí --> J
    end

    B -- No --> C
    B -- Sí --> J

    subgraph F2["Fase 2: Selección de ciudad y ruta"]
        direction TB
        K[Mostrar opciones principales]
        L[Abrir selector de ciudades]
        M[Obtener ciudades disponibles]
        N[Usuario selecciona ciudad]
        O[Obtener rutas de la ciudad]
        P[Mostrar rutas disponibles]
        Q[Usuario selecciona ruta]
        R[Abrir pantalla de navegación]

        K --> L --> M --> N --> O --> P --> Q --> R
    end

    J --> K

    subgraph F3["Fase 3: Navegación y misión"]
        direction TB
        S[Inicializar gestor de ruta]
        T[Obtener POIs de la ruta]
        U[Iniciar GPS y cálculo de trayecto]
        V{¿Se alcanza un POI?}
        W[Mostrar aviso de llegada]
        X{¿Acción en el punto?}
        Y[Abrir escáner QR]
        Z[Marcar punto omitido/completado]
        AA[Leer QR]
        AB[Validar punto y misión]
        AC{¿QR válido?}
        AD[Mostrar detalle del monumento]
        AE[Iniciar cuestionario]
        AF[Responder preguntas]
        AG[Calcular resultado]
        AH[Registrar punto completado]
        AI[Actualizar progreso]

        S --> T --> U --> V
        V -- No --> U
        V -- Sí --> W --> X
        X -- Escanear QR --> Y --> AA --> AB --> AC
        X -- Omitir punto --> Z --> AI
        AC -- No --> Y
        AC -- Sí --> AD --> AE --> AF --> AG --> AH --> AI
    end

    R --> S

    subgraph F4["Fase 4: Cierre de ruta y resultados"]
        direction TB
        AJ{¿Quedan puntos pendientes?}
        AK[Recalcular siguiente destino]
        AL[Finalizar ruta]
        AM[Calcular puntuación final]
        AN[Guardar mejor progreso]
        AO{¿Guardar resultado detallado?}
        AP[Guardar resultado detallado]
        AQ[Mostrar resultados finales]
        AR{¿Iniciar nueva ruta?}
        AS[Volver a selector de ciudades]
        AT[Fin]

        AJ -- Sí --> AK
        AK --> U
        AJ -- No --> AL --> AM --> AN --> AO
        AO -- Sí --> AP --> AQ
        AO -- No --> AQ
        AQ --> AR
        AR -- Sí --> AS
        AR -- No --> AT
    end

    AI --> AJ
    AS --> L

    style F1 fill:#E8F5E9,stroke:#4CAF50,stroke-width:2px
    style F2 fill:#E3F2FD,stroke:#2196F3,stroke-width:2px
    style F3 fill:#F3E5F5,stroke:#9C27B0,stroke-width:2px
    style F4 fill:#FFF3E0,stroke:#FF9800,stroke-width:2px
```

---

## Diagrama de Casos de Uso

```mermaid
flowchart LR
    %% Definición de Actores
    User[Usuario]
    Admin[Administrador]

    %% Estructura de Casos de Uso
    subgraph Sistema [Sistema: RuteX Go]
        
        %% 1. Autenticarse
        U1[1. Autenticarse]
        U1 --> S1[Iniciar sesión]
        U1 --> S2[Registrarse]
        U1 --> S3[Iniciar sesión con Google]
        U1 --> S4[Recuperar contraseña]

        %% 2. Gestionar perfil
        U2[2. Gestionar perfil]
        U2 --> S5[Ver información personal]
        U2 --> S6[Actualizar datos de perfil]

        %% 3. Explorar rutas
        U3[3. Explorar rutas]
        U3 --> S7[Seleccionar ciudad]
        U3 --> S8[Seleccionar ruta]
        U3 --> S9[Consultar detalles de ruta]

        %% 4. Realizar misión
        U4[4. Realizar misión]
        U4 --> S10[Escanear punto de interés]
        U4 --> S11[Completar reto de ruta]
        U4 --> S12[Consultar progreso]

        %% 5. Consultar diario
        U5[5. Consultar diario]
        U5 --> S13[Seleccionar rutas completadas]
        U5 --> S14[Añadir recuerdos]
        U5 --> S15[Generar diario]

        %% 6. Administrar contenido
        U6[6. Administrar contenido]
        U6 --> S16[Gestionar ciudades]
        U6 --> S17[Gestionar rutas]
        U6 --> S18[Gestionar puntos de interés]
        U6 --> S19[Gestionar misiones]
    end

    %% Conexiones con Actores
    User --> U1
    User --> U2
    User --> U3
    User --> U4
    User --> U5

    Admin --> U1
    Admin --> U3
    Admin --> U6
```
