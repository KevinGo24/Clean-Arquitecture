# Clean-Arquitecture
Conceptos
# Capa de Dominio 

La capa de dominio es el nucleo fundamental de toda aplicacion siguiendo los conceptos de arquitectura limpia, y no depende de nadie

- Entidades (Entities):
    - prodcutos
    - inventario
    - ejemplo: productos e inventarios
- Objetos de valores (ObjectValues):
    - Dinero
    - Cantida_Inventario
    - ejmplo: dinero e cantidad de inventario
- Exepciones (Exeptions):
    - NotificacionExeptions
    - ejemplo: mensaje de validaciones
## Carpeta de Dominio
- Enttidades
- Objetos de valores
- Excepciones
# Capa de Aplicacion

es el nivel más cercano al usuario final en las arquitecturas de red, actuando como la interfaz directa entre los programas de software y la red subyacente

- casos de usos:
    - productos
    - comandos:
        - Crear productos:
            - ComandoCrearProductos
            - CasoUsosCrearProductos
- Contraos:
    - IRepositorioProductos
## Carpetas de Dominio
- Casos de usos:
    - productos
    - comandos
# Capa de Infratuctura

 Es el nivel base que provee los recursos físicos, virtuales o técnicos necesarios para que un sistema, una aplicación o una red puedan funciona

 - Aqui es donde va los driver de motores de base de datos

 ## Carpetas de infratuctura
 - Persistencia:
    * AplicationsDbContext
    * Carp_Configuraciones
    * repositorios
- Inyecion de dependecia

# Capa de Api-Web

 es una sección o conjunto de componentes en la arquitectura de software que actúa como intermediaria y punto de comunicación entre la interfaz de usuario (el cliente) y los servicios de la parte interna (backend, bases de datos y servidores)
 
- Dependendica Desings 
# Referencias 

- Dominio refiere a la capa de Aplicacion
- Infratutura refiere a la caoa de aplicacion
- Api refiere a infratucura y aplicaciom

# CQRES

Un patrón de arquitectura de software que separa las operaciones de lectura (consultas) de las operaciones de escritura (comandos)

## ingles

 CQRS (Command Query Responsibility Segregation)