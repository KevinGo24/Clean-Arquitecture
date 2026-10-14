# 🧭 Guía de estudio: Arquitectura limpia

Apuntes para reconocer las responsabilidades de cada capa y cómo se relacionan. La idea central es mantener las reglas del negocio independientes de detalles técnicos como la base de datos o la API.

## 🗺️ Mapa rápido

| Capa | Responsabilidad principal | Ejemplos |
| --- | --- | --- |
| 🏛️ Dominio | Reglas y conceptos del negocio | Entidades, objetos de valor |
| ⚙️ Aplicación | Acciones y casos de uso | Crear producto, consultar inventario |
| 🔌 Infraestructura | Implementaciones técnicas | Base de datos, repositorios, servicios externos |
| 🌐 API Web | Comunicación HTTP con clientes | Controladores, DTOs |

> **Regla de dependencia:** las dependencias apuntan hacia el centro. El dominio no conoce la aplicación, la base de datos ni la API.

```text
🌐 API Web ──────► ⚙️ Aplicación ──────► 🏛️ Dominio
      │                    ▲
      └──── configura ─────┤
                    🔌 Infraestructura
```

La infraestructura implementa contratos definidos por la aplicación. La API suele configurar esas implementaciones al iniciar la aplicación.

## 🗂️ Mapa de la solución `Firmeza.Web`

Estructura real del proyecto, separada por capa. Cada proyecto de la solución corresponde a una capa de la arquitectura.

```text
Firmeza.Web
├── 🌐 FirmezaApi                  (API Web)
│   ├── Dependencies
│   ├── Properties
│   ├── Controllers
│   ├── appsettings.json
│   ├── appsettings.Development.json
│   ├── FirmezaApi.http
│   ├── Program.cs
│   └── UserRoles.cs
│
├── ⚙️ FirmezaApplications         (Aplicación)
│   ├── Dependencies
│   ├── Common
│   ├── DTOs
│   ├── Erros
│   ├── Events
│   ├── Interface
│   ├── Services
│   └── DependencyInjection.cs
│
├── 🏛️ FirmezaDominion             (Dominio)
│   ├── Dependencies
│   ├── Exceptions
│   ├── Models
│   └── ValueObjects
│
└── 🔌 FirmezaInfractucture        (Infraestructura)
    ├── Dependencies
    ├── Migrations
    ├── Persistence
    ├── Repository
    └── DependencyInjection.cs
```

| Proyecto | Capa |
| --- | --- |
| `FirmezaDominion` | 🏛️ Dominio |
| `FirmezaApplications` | ⚙️ Aplicación |
| `FirmezaInfractucture` | 🔌 Infraestructura |
| `FirmezaApi` | 🌐 API Web |

> **Notas:**
> - `Dependencies` es el nodo del IDE que agrupa paquetes NuGet, proyectos y SDK referenciados; no es una carpeta real. Lo mismo ocurre con `BuildHost-net472` y `BuildHost-netcore`, que genera el IDE y no se muestran en el mapa.
> - La solución indica 5 proyectos, pero la captura solo muestra 4. Si el quinto es otro proyecto (por ejemplo, pruebas), puede agregarse al mapa.
> - Los nombres `FirmezaDominion`, `FirmezaInfractucture` y la carpeta `Erros` se muestran tal como aparecen en la solución. Si son errores de escritura, conviene renombrarlos (`FirmezaDominio`, `FirmezaInfrastructure`, `Errors`) antes de que el proyecto crezca.

## 🔗 Referencias entre capas (resumen)

Cada fila indica **qué proyectos referencia** la capa. Una capa nunca debe referenciar a otra que esté "hacia afuera" de ella.

| Capa (proyecto) | Referencia a | No debe referenciar a |
| --- | --- | --- |
| 🏛️ `Dominio` | Ninguna | Aplicación, Infraestructura, API Web |
| ⚙️ `Aplicacion` | `Dominio` | Infraestructura, API Web |
| 🔌 `Infraestructura` | `Aplicacion` (y por transitividad `Dominio`) | API Web |
| 🌐 `Api` | `Aplicacion` e `Infraestructura` | — |

> **Nota:** la API referencia a Infraestructura únicamente para registrar las implementaciones al iniciar la aplicación (inyección de dependencias). Los controladores no deben usar clases de infraestructura directamente.

### Comandos para crear las referencias

Con los nombres de la solución `Firmeza.Web` (ejecutar desde la carpeta de la solución):

```bash
# Aplicación -> Dominio
dotnet add FirmezaApplications reference FirmezaDominion

# Infraestructura -> Aplicación
dotnet add FirmezaInfractucture reference FirmezaApplications

# API -> Aplicación e Infraestructura
dotnet add FirmezaApi reference FirmezaApplications FirmezaInfractucture
```

## 📦 Dependencias (paquetes NuGet) por capa (resumen)

Los paquetes son ejemplos habituales en proyectos .NET; ajusta según las tecnologías que uses.

| Capa | Paquetes típicos | Observación |
| --- | --- | --- |
| 🏛️ Dominio | Ninguno | Debe quedar libre de frameworks y librerías externas |
| ⚙️ Aplicación | `MediatR`, `FluentValidation`, `Microsoft.Extensions.DependencyInjection.Abstractions` | Solo abstracciones y librerías que no dependen de infraestructura |
| 🔌 Infraestructura | `Microsoft.EntityFrameworkCore`, proveedor de base de datos (por ejemplo `Microsoft.EntityFrameworkCore.SqlServer` o `Npgsql.EntityFrameworkCore.PostgreSQL`) | Aquí vive todo lo que depende de la tecnología concreta |
| 🌐 API Web | `Swashbuckle.AspNetCore` (o `Microsoft.AspNetCore.OpenApi`), `Microsoft.EntityFrameworkCore.Design` | Es el proyecto de inicio, por eso suele alojar los paquetes de arranque y herramientas |

## 🏛️ Capa de dominio

Contiene el núcleo del negocio: sus reglas, conceptos y validaciones. Debe ser independiente de frameworks, bases de datos y otras capas.

- **Entidades (Entities):** objetos con identidad y ciclo de vida. Ejemplos: `Producto`, `Inventario`.
- **Objetos de valor (Value Objects):** se definen por sus valores, no por una identidad propia. Ejemplos: `Dinero`, `CantidadInventario`.
- **Excepciones o errores de dominio:** representan situaciones inválidas según las reglas del negocio. Ejemplo: intentar registrar una cantidad negativa.

**Carpetas posibles:** `Entidades`, `ObjetosDeValor`, `Excepciones`.

### 🗂️ Estructura en Firmeza: `FirmezaDominion`

```text
FirmezaDominion
├── Dependencies
├── Exceptions
├── Models
└── ValueObjects
```

| Carpeta | Qué contiene |
| --- | --- |
| `Models` | Entidades del negocio |
| `ValueObjects` | Objetos de valor |
| `Exceptions` | Excepciones de dominio |

### 🔗 Referencias

- **Referencia a proyectos:** ninguna. Es el centro de la arquitectura y no conoce a ninguna otra capa.
- **Quién la referencia:** Aplicación (directamente), Infraestructura y API (de forma transitiva).

### 📦 Dependencias

- **Paquetes NuGet:** ninguno. Solo usa el SDK base de .NET.
- **Evitar aquí:** Entity Framework Core, ASP.NET Core, atributos de serialización o de base de datos, librerías de logging o de terceros.

## ⚙️ Capa de aplicación

Coordina las tareas que el sistema ofrece. Cada **caso de uso** describe una acción concreta y utiliza el dominio para aplicar las reglas del negocio.

- **Comandos:** solicitan un cambio, por ejemplo `CrearProducto`.
- **Consultas:** solicitan información sin cambiar el estado, por ejemplo `ObtenerProductoPorId`.
- **Contratos:** interfaces que la aplicación necesita que otras capas implementen, por ejemplo `IRepositorioProductos`.

**Ejemplo de flujo:** `CrearProducto` → caso de uso `CrearProducto` → reglas del dominio → `IRepositorioProductos`.

**Carpetas posibles:** `CasosDeUso/Productos`, `Comandos`, `Consultas`, `Contratos`.

### 🗂️ Estructura en Firmeza: `FirmezaApplications`

```text
FirmezaApplications
├── Dependencies
├── Common
├── DTOs
├── Erros
├── Events
├── Interface
├── Services
└── DependencyInjection.cs
```

| Carpeta / archivo | Qué contiene |
| --- | --- |
| `Common` | Clases y utilidades compartidas dentro de la aplicación |
| `DTOs` | Objetos para transportar datos desde y hacia los casos de uso |
| `Erros` | Errores propios de la aplicación |
| `Events` | Eventos de la aplicación |
| `Interface` | Contratos que implementa la infraestructura (por ejemplo, repositorios) |
| `Services` | Casos de uso y lógica de coordinación |
| `DependencyInjection.cs` | Registro de los servicios de esta capa |

> En esta solución los DTOs viven en la capa de aplicación (`DTOs`) y no en la API, así los casos de uso y los controladores comparten los mismos objetos de transporte.

### 🔗 Referencias

- **Referencia a proyectos:** `Dominio`.
- **Quién la referencia:** Infraestructura y API.

### 📦 Dependencias

- **Paquetes NuGet típicos (opcionales según el diseño):**
  - `MediatR`: para implementar comandos y consultas (CQRS).
  - `FluentValidation`: para validar comandos y consultas.
  - `Microsoft.Extensions.DependencyInjection.Abstractions`: para exponer un método de registro de servicios de la capa.
- **Evitar aquí:** Entity Framework Core, proveedores de base de datos y cualquier paquete de ASP.NET Core. La aplicación define **interfaces** y deja que Infraestructura las implemente.

## 🔌 Capa de infraestructura

Contiene los detalles técnicos y las implementaciones de los contratos definidos por la aplicación. Aquí se conectan los componentes externos.

- Persistencia: `ApplicationDbContext`, configuraciones y migraciones.
- Repositorios concretos que implementan interfaces como `IRepositorioProductos`.
- Integraciones con motores de base de datos y otros servicios externos.
- Registro de dependencias para conectar contratos con sus implementaciones.

**Carpetas posibles:** `Persistencia`, `Configuraciones`, `Repositorios`, `InyeccionDependencias`.

### 🗂️ Estructura en Firmeza: `FirmezaInfractucture`

```text
FirmezaInfractucture
├── Dependencies
├── Migrations
├── Persistence
├── Repository
└── DependencyInjection.cs
```

| Carpeta / archivo | Qué contiene |
| --- | --- |
| `Migrations` | Migraciones de la base de datos |
| `Persistence` | `DbContext` y configuraciones de persistencia |
| `Repository` | Repositorios concretos que implementan las interfaces de `FirmezaApplications` |
| `DependencyInjection.cs` | Registro de las implementaciones de esta capa |

### 🔗 Referencias

- **Referencia a proyectos:** `Aplicacion` (para implementar sus contratos). `Dominio` llega de forma transitiva y también se usa para mapear las entidades.
- **Quién la referencia:** solo la API, para registrar las implementaciones al arrancar.

### 📦 Dependencias

- **Paquetes NuGet típicos:**
  - `Microsoft.EntityFrameworkCore`: ORM para la persistencia.
  - Proveedor de base de datos, según el motor: `Microsoft.EntityFrameworkCore.SqlServer`, `Npgsql.EntityFrameworkCore.PostgreSQL`, `Pomelo.EntityFrameworkCore.MySql`, etc.
  - `Microsoft.EntityFrameworkCore.Relational`: se incluye con el proveedor; no suele agregarse a mano.
  - `Microsoft.Extensions.DependencyInjection.Abstractions` y `Microsoft.Extensions.Configuration.Abstractions`: para el método de registro de servicios y la lectura de la cadena de conexión.
- **Aquí van** también los paquetes de cualquier servicio externo (correo, almacenamiento, colas, etc.).

## 🌐 Capa de API Web

Expone los casos de uso a clientes mediante HTTP. Recibe solicitudes, valida su formato, llama a la aplicación y devuelve respuestas.

- **Controladores:** definen endpoints y coordinan la entrada y salida HTTP.
- **DTOs (Data Transfer Objects):** objetos para transportar datos entre la API y sus clientes. No sustituyen a las entidades del dominio.
- **Configuración de dependencias:** el punto de entrada puede registrar las implementaciones de infraestructura que usará la aplicación.

**Carpetas posibles:** `Controllers`, `Dtos`.

### 🗂️ Estructura en Firmeza: `FirmezaApi`

```text
FirmezaApi
├── Dependencies
├── Properties
├── Controllers
├── appsettings.json
├── appsettings.Development.json
├── FirmezaApi.http
├── Program.cs
└── UserRoles.cs
```

| Carpeta / archivo | Qué contiene |
| --- | --- |
| `Controllers` | Endpoints HTTP |
| `Properties` | Configuración de ejecución del proyecto (perfiles de lanzamiento) |
| `appsettings.json` / `appsettings.Development.json` | Configuración general y del entorno de desarrollo |
| `FirmezaApi.http` | Solicitudes de prueba para los endpoints |
| `Program.cs` | Punto de entrada; configura servicios y llama a los `DependencyInjection.cs` de las otras capas |
| `UserRoles.cs` | Roles de usuario usados por la API |

### 🔗 Referencias

- **Referencia a proyectos:** `Aplicacion` e `Infraestructura`.
- **Quién la referencia:** nadie. Es la capa más externa y el punto de entrada.

### 📦 Dependencias

- **Paquetes NuGet típicos:**
  - `Swashbuckle.AspNetCore` o `Microsoft.AspNetCore.OpenApi`: documentación de la API.
  - `Microsoft.EntityFrameworkCore.Design`: necesario en el proyecto de inicio para crear migraciones.
  - Paquetes de autenticación (por ejemplo `Microsoft.AspNetCore.Authentication.JwtBearer`), si aplica.
- **Regla práctica:** los controladores usan casos de uso y DTOs; no deberían usar `ApplicationDbContext` ni repositorios concretos.

## 🔀 CQRS

**CQRS** significa *Command Query Responsibility Segregation* (segregación de responsabilidades entre comandos y consultas). Separa las operaciones que modifican datos de las que solo los leen:

- **Comando:** cambia el estado del sistema. Ejemplo: `CrearProducto`.
- **Consulta:** devuelve información sin modificar el estado. Ejemplo: `ListarProductos`.

CQRS puede aplicarse dentro de una sola aplicación; no implica necesariamente tener dos bases de datos.

## 🧠 Repaso rápido

1. ¿Qué capa debe contener las reglas del negocio? **Dominio.**
2. ¿Qué capa coordina una acción como crear un producto? **Aplicación.**
3. ¿Dónde se implementa el acceso a la base de datos? **Infraestructura.**
4. ¿Qué diferencia un comando de una consulta? **El comando modifica el estado; la consulta solo lee.**
5. ¿Qué proyectos referencia la API? **Aplicación e Infraestructura (esta última solo para configurar dependencias).**
6. ¿Qué capa no debe tener paquetes NuGet externos? **Dominio.**
