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

## 🏛️ Capa de dominio

Contiene el núcleo del negocio: sus reglas, conceptos y validaciones. Debe ser independiente de frameworks, bases de datos y otras capas.

- **Entidades (Entities):** objetos con identidad y ciclo de vida. Ejemplos: `Producto`, `Inventario`.
- **Objetos de valor (Value Objects):** se definen por sus valores, no por una identidad propia. Ejemplos: `Dinero`, `CantidadInventario`.
- **Excepciones o errores de dominio:** representan situaciones inválidas según las reglas del negocio. Ejemplo: intentar registrar una cantidad negativa.

**Carpetas posibles:** `Entidades`, `ObjetosDeValor`, `Excepciones`.

## ⚙️ Capa de aplicación

Coordina las tareas que el sistema ofrece. Cada **caso de uso** describe una acción concreta y utiliza el dominio para aplicar las reglas del negocio.

- **Comandos:** solicitan un cambio, por ejemplo `CrearProducto`.
- **Consultas:** solicitan información sin cambiar el estado, por ejemplo `ObtenerProductoPorId`.
- **Contratos:** interfaces que la aplicación necesita que otras capas implementen, por ejemplo `IRepositorioProductos`.

**Ejemplo de flujo:** `CrearProducto` → caso de uso `CrearProducto` → reglas del dominio → `IRepositorioProductos`.

**Carpetas posibles:** `CasosDeUso/Productos`, `Comandos`, `Consultas`, `Contratos`.

## 🔌 Capa de infraestructura

Contiene los detalles técnicos y las implementaciones de los contratos definidos por la aplicación. Aquí se conectan los componentes externos.

- Persistencia: `ApplicationDbContext`, configuraciones y migraciones.
- Repositorios concretos que implementan interfaces como `IRepositorioProductos`.
- Integraciones con motores de base de datos y otros servicios externos.
- Registro de dependencias para conectar contratos con sus implementaciones.

**Carpetas posibles:** `Persistencia`, `Configuraciones`, `Repositorios`, `InyeccionDependencias`.

## 🌐 Capa de API Web

Expone los casos de uso a clientes mediante HTTP. Recibe solicitudes, valida su formato, llama a la aplicación y devuelve respuestas.

- **Controladores:** definen endpoints y coordinan la entrada y salida HTTP.
- **DTOs (Data Transfer Objects):** objetos para transportar datos entre la API y sus clientes. No sustituyen a las entidades del dominio.
- **Configuración de dependencias:** el punto de entrada puede registrar las implementaciones de infraestructura que usará la aplicación.

**Carpetas posibles:** `Controllers`, `Dtos`.

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