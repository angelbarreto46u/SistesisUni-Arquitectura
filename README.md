# 2. Guía de Renderizado y Explotación con Mermaid Live Editor

Uso de Mermaid.js para modelar arquitectura mediante código en repositorios y editores web[cite: 1].

### 2.1 Paso a Paso para la Creación y Exportación
1. Ingrese al entorno en línea a través de [Mermaid Live Editor](https://mermaid.live)[cite: 1].
2. Copie el código Mermaid del diagrama y péguelo en el panel izquierdo (Code Editor)[cite: 1].
3. Verifique la vista previa interactiva en el panel derecho (Preview)[cite: 1].
4. En el menú inferior **Actions**, seleccione **PNG** para documentación digital o **SVG** para formato vectorial imprimible[cite: 1].
5. Puede copiar el enlace codificado mediante el botón **Copy Link to Clipboard**[cite: 1].

### 2.2 Inclusión en Repositorios GitHub (README.md)
GitHub renderiza nativamente bloques Mermaid utilizando el bloque de código correspondiente[cite: 1]:

```mermaid
graph TD
    A[Usuario] --> B(SISTESIS-UNI)
```

---

# 3. Especificación de Diagramas Mermaid.js

### 3.1 Diagrama de Secuencia API-First (Autenticación JWT y Carga Asíncrona)
Flujo cronológico de autenticación, validación de token y persistencia del documento[cite: 1].

```mermaid
sequenceDiagram
    autonumber
    participant Estudiante as Estudiante UNI
    participant Client as Single Page Application (Angular)
    participant AuthAPI as API Gateway / Auth Controller
    participant ThesisAPI as Thesis Controller / Command Service
    participant Storage as Cloud/Blob Storage
    participant DB as PostgreSQL Database

    Estudiante->>Client: 1. Ingresa credenciales (UNI ID & Password)
    Client->>AuthAPI: 2. POST /api/v1/auth/login
    AuthAPI->>DB: 3. Validar credenciales e identidad
    DB-->>AuthAPI: 4. Usuario valido (Role: Student)
    AuthAPI-->>Client: 5. Retorna JWT Access Token + Refresh Token
    
    Estudiante->>Client: 6. Carga Protocolo de Tesis (.pdf) y metadatos
    Client->>ThesisAPI: 7. POST /api/v1/theses/upload (Header Bearer JWT)
    ThesisAPI->>ThesisAPI: 8. Validar Token JWT y Claims de Autorizacion
    ThesisAPI->>Storage: 9. Stream de archivo PDF a almacenamiento
    Storage-->>ThesisAPI: 10. URI del archivo almacenado
    ThesisAPI->>DB: 11. Registrar metadatos de Tesis (Estado: Pendiente)
    DB-->>ThesisAPI: 12. Confirmacion de persistencia (ID Tesis)
    ThesisAPI-->>Client: 13. 202 Accepted / 201 Created (Thesis JSON Payload)
    Client-->>Estudiante: 14. Muestra confirmacion de entrega registrada
```

---

### 3.2 Diagrama de Contexto C4 (Nivel 1)
Fronteras del sistema SISTESIS-UNI, usuarios principales y sistemas externos[cite: 1].

```mermaid
graph TD
    subgraph ContextoBoundary ["SISTESIS-UNI - Sistema de Gestion de Tesis Monograficas"]
        SISTESIS["SISTESIS-UNI System"]
    end
    Estudiante["Estudiante UNI<br/>Usuario que somete protocolos y propuestas de tesis"]
    Docente["Docente / Tutor<br/>Revisor de propuestas y evaluador de avances"]
    Admin["Administrador TIC<br/>Gestiona usuarios, roles y parametros institucionales"]
    SIAT["SIAT UNI<br/>Sistema de Control Academico Central"]
    NotificationSystem["Servicio de Notificaciones<br/>SMTP / Servidor de Correo UNI"]

    Estudiante -->|Envia protocolos y consulta estados| SISTESIS
    Docente -->|Revisa, observa y aprueba tesis| SISTESIS
    Admin -->|Configura periodos y asigna tutores| SISTESIS
    SISTESIS -->|Sincroniza matricula y datos de estudiantes| SIAT
    SISTESIS -->|Envia avisos de revision y estado| NotificationSystem

    style SISTESIS fill:#f1f5f9,stroke:#475569,stroke-width:2px,color:#0f172a
    style ContextoBoundary stroke:#cbd5e1,stroke-dasharray: 5 5
```

---

### 3.3 Diagrama de Contenedores C4 (Nivel 2)
Topología de ejecución, aplicaciones cliente, API backend, bases de datos y almacenamiento[cite: 1].

```mermaid
graph TB
    subgraph SISTESIS ["SISTESIS-UNI Boundary"]
        SPA["Single Page Application<br/>Angular / TypeScript<br/>Interfaz Web para usuarios"]
        MobileApp["Aplicacion Movil<br/>Flutter<br/>Consultas rapidas e intimaciones"]
        API["API RESTful backend<br/>ASP.NET Core 8 / C#<br/>Logica de Negocio y Control de Acceso"]
        DB[("PostgreSQL Database<br/>Almacenamiento Relacional de Metadatos")]
        BlobStorage[("MinIO / S3 Object Storage<br/>Almacenamiento de Documentos PDF")]
    end
    SIAT_Ext["SIAT UNI API<br/>Sistema de Control Academico"]
    SMTP_Ext["Servidor SMTP UNI<br/>Notificaciones"]

    Estudiante -->|HTTPS / WSS| SPA
    Docente -->|HTTPS / WSS| SPA
    Estudiante -->|HTTPS| MobileApp
    SPA -->|JSON / HTTPS API| API
    MobileApp -->|JSON / HTTPS API| API
    API -->|Npgsql / EF Core| DB
    API -->|S3 SDK| BlobStorage
    API -->|REST / HTTPS| SIAT_Ext
    API -->|SMTP Protocol| SMTP_Ext

    style API fill:#f8fafc,stroke:#334155,stroke-width:2px,color:#0f172a
    style DB fill:#f8fafc,stroke:#334155,stroke-width:1px,color:#0f172a
    style BlobStorage fill:#f8fafc,stroke:#334155,stroke-width:1px,color:#0f172a
    style SPA fill:#f8fafc,stroke:#334155,stroke-width:1px,color:#0f172a
    style MobileApp fill:#f8fafc,stroke:#334155,stroke-width:1px,color:#0f172a
```

---

# 5. Guía de Ejercicios Propuestos y Resueltos

### Ejercicio 1: Modelado de Manejo de Errores en Secuencia
Manejo cuando un estudiante sube un archivo PDF con un tamaño superior al límite permitido (15 MB)[cite: 1].

```mermaid
sequenceDiagram
    autonumber
    participant Estudiante as Estudiante UNI
    participant Client as Web App
    participant API as Thesis API
    
    Estudiante->>Client: Adjunta archivo PDF (> 15MB)
    Client->>API: POST /api/v1/theses/upload (Multipart Form Data)
    API->>API: Validar File Size Limit (Middleware)
    API-->>Client: 413 Payload Too Large (JSON Error Detail)
    Client-->>Estudiante: Muestra mensaje "El archivo excede el limite de 15MB"
```

---

### Ejercicio 2: Inclusión de Microservicio de Anti-Plagio
Extensión de contenedores C4 incluyendo RabbitMQ y el motor `PlagiarismCheckerEngine` (Python/FastAPI) con actualización asíncrona en PostgreSQL[cite: 1].

```mermaid
graph TB
    subgraph SISTESIS ["SISTESIS-UNI Boundary"]
        SPA["Single Page Application<br/>Angular / TypeScript"]
        MobileApp["Aplicacion Movil<br/>Flutter"]
        API["API RESTful backend<br/>ASP.NET Core 8 / C#"]
        DB[("PostgreSQL Database<br/>Metadatos y Estado de Plagio")]
        BlobStorage[("MinIO / S3 Object Storage<br/>Almacenamiento de Documentos PDF")]
        RabbitMQ[("RabbitMQ Message Broker<br/>Cola thesis.submitted")]
        PlagiarismEngine["PlagiarismCheckerEngine<br/>Python / FastAPI"]
    end

    SIAT_Ext["SIAT UNI API"]
    SMTP_Ext["Servidor SMTP UNI"]

    Estudiante -->|HTTPS| SPA
    Docente -->|HTTPS| SPA
    Estudiante -->|HTTPS| MobileApp
    SPA -->|JSON / HTTPS API| API
    MobileApp -->|JSON / HTTPS API| API
    API -->|Npgsql / EF Core| DB
    API -->|S3 SDK| BlobStorage
    API -->|Publica evento ThesisUploadedEvent| RabbitMQ

    RabbitMQ -->|Consume evento con URI del PDF| PlagiarismEngine
    PlagiarismEngine -->|Descarga documento para analisis| BlobStorage
    PlagiarismEngine -->|Actualiza Score y Estado| DB
    PlagiarismEngine -->|Alerta si similitud es alta| SMTP_Ext
    API -->|REST / HTTPS| SIAT_Ext
    API -->|SMTP Protocol| SMTP_Ext

    style PlagiarismEngine fill:#ecfdf5,stroke:#059669,stroke-width:2px,color:#064e3b
    style RabbitMQ fill:#fef3c7,stroke:#d97706,stroke-width:2px,color:#78350f
    style API fill:#f8fafc,stroke:#334155,stroke-width:2px,color:#0f172a
    style DB fill:#f8fafc,stroke:#334155,stroke-width:1px,color:#0f172a
    style BlobStorage fill:#f8fafc,stroke:#334155,stroke-width:1px,color:#0f172a
```