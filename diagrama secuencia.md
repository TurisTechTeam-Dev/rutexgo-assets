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
    A[Inicio: Usuario abre la aplicación] --> B{¿Existe sesión activa?}

    B -- No --> C[Mostrar pantalla de autenticación]
    C --> D{¿Acción del usuario?}
    D -- Iniciar sesión --> E[Validar credenciales]
    D -- Registrarse --> F[Registrar nuevo usuario]

    E --> G{¿Autenticación correcta?}
    G -- No --> H[Mostrar error de autenticación]
    H --> C
    G -- Sí --> I[Acceder a pantalla principal]

    F --> J{¿Registro correcto?}
    J -- No --> K[Mostrar error de registro]
    K --> C
    J -- Sí --> I

    B -- Sí --> I

    I --> L[Mostrar opciones principales]
    L --> M[Acceder al selector de ciudades]
    M --> N[Obtener ciudades disponibles]
    N --> O[Mostrar listado de ciudades]
    O --> P[Usuario selecciona ciudad]

    P --> Q[Obtener rutas asociadas a la ciudad]
    Q --> R[Mostrar rutas disponibles]
    R --> S[Usuario selecciona ruta]
    S --> T[Abrir pantalla de navegación]

    T --> U[Inicializar gestor de ruta]
    U --> V[Obtener puntos de interés de la ruta]
    V --> W[Iniciar seguimiento GPS y cálculo de trayecto]

    W --> X{¿Se alcanza un punto de interés?}
    X -- No --> W
    X -- Sí --> Y[Mostrar aviso de llegada al punto]

    Y --> Z{¿Acción en el punto?}
    Z -- Escanear QR --> AA[Abrir escáner QR]
    Z -- Omitir punto --> AB[Marcar punto como omitido/completado sin misión]

    AA --> AC[Leer código QR]
    AC --> AD[Validar punto y misión asociada]
    AD --> AE{¿QR válido y punto esperado?}
    AE -- No --> AF[Mostrar mensaje de QR no válido]
    AF --> AA
    AE -- Sí --> AG[Mostrar detalle del monumento]

    AG --> AH[Iniciar cuestionario de misión]
    AH --> AI[Usuario responde preguntas]
    AI --> AJ[Calcular resultado de la misión]
    AJ --> AK[Registrar punto como completado]

    AB --> AL[Actualizar progreso de ruta]
    AK --> AL

    AL --> AM{¿Quedan puntos pendientes?}
    AM -- Sí --> AN[Recalcular siguiente destino]
    AN --> W

    AM -- No --> AO[Finalizar ruta]
    AO --> AP[Calcular puntuación final y métricas]
    AP --> AQ[Guardar mejor progreso de ruta]
    AQ --> AR{¿Corresponde guardar resultado detallado?}
    AR -- Sí --> AS[Guardar resultado detallado de la ruta]
    AR -- No --> AT[Continuar sin guardado detallado]

    AS --> AU[Mostrar pantalla de resultados finales]
    AT --> AU

    AU --> AV{¿Usuario desea iniciar nueva ruta?}
    AV -- Sí --> M
    AV -- No --> AW[Fin del proceso]
```
