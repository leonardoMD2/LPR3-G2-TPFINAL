# Trabajo Practico LPR3 G2 - Biblioteca de juegos Retro

Martinez Leonardo

Este trabajo practico consiste en construir una API REST con FastAPI y Supabase (PostgreSQL) para gestionar una biblioteca de juegos retro, aplicando inyeccion de dependencias, managers orientados a objetos, interfaces como contratos y validacion con Pydantic. Se resuelve de forma individual.

## Objetivos y alcance

Al finalizar el trabajo, el estudiante debe demostrar que puede disenar una base de datos relacional normalizada y exponerla mediante una API ordenada por capas, donde cada pieza tiene una unica responsabilidad.

Objetivos de aprendizaje:

- Diseniar un esquema relacional con claves primarias, claves foraneas y tablas intermedias para relaciones muchos a muchos.
- Conectar FastAPI con Supabase (PostgreSQL) sin acoplar los endpoints a la base de datos.
- Aplicar inyeccion de dependencias para entregar a cada endpoint su manager, y al manager su conexion.
- Representar la logica de negocio en clases (managers) que cumplan contratos definidos por interfaces.
- Validar toda entrada y salida de datos con modelos Pydantic.
- Resolver consultas que combinan varias tablas y exponerlas con filtros de consulta y rutas dinamicas.
- Explicar y justificar las decisiones de diseno en defensa oral.

## Dominio: biblioteca de juegos retro

Se desarrolla el sistema de una biblioteca comunitaria de juegos retro, donde socios coleccionistas pueden retirar en prestamo cartuchos y discos fisicos de consolas clasicas.

La biblioteca cataloga juegos publicados por desarrolladoras, cada uno disponible para una o mas consolas y clasificado en uno o mas generos. Cada juego puede tener varias copias fisicas (ejemplares), cada una con su estado de conservacion. Los socios solicitan prestamos de ejemplares concretos y los devuelven dentro de un plazo.

El sistema debe permitir:

- Administrar el catalogo (desarrolladoras, consolas, generos, juegos y ejemplares).
- Administrar socios.
- Registrar prestamos y devoluciones.
- Consultar el catalogo y el historial con filtros y con informacion combinada de varias tablas.

## Modelo de datos

El esquema debe tener tantas tablas como consideren. Dentro del readme final debe estar la justificacion del DER propuesto para tablas y relaciones.


## Requisitos tecnicos

La aplicacion se organiza en capas con dependencias que van siempre en una sola direccion: endpoint, manager, acceso a datos.

| Elemento                  | Que se espera                                                                                                                                                                                            |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Backend                   | FastAPI, con routers separados por recurso y registrados en la aplicacion principal.                                                                                                                     |
| Base de datos             | PostgreSQL alojado en Supabase. Las credenciales se leen de variables de entorno; nunca se escriben en el codigo ni se suben al repositorio.                                                             |
| Inyeccion de dependencias | Se usa el mecanismo de dependencias de FastAPI para entregar al endpoint su manager y al manager su conexion o cliente de base de datos. Ningun endpoint crea managers ni conexiones por su cuenta.      |
| Managers (POO)            | Una clase por recurso. Concentra la logica de negocio y todo acceso a datos de ese recurso. Los endpoints solo reciben la peticion, delegan en el manager y devuelven la respuesta.                      |
| Interfaces                | Una interfaz (clase abstracta) por manager que declara las operaciones obligatorias. Cada manager concreto implementa su interfaz, y los endpoints dependen de la interfaz y no de la clase concreta.    |
| Modelos Pydantic          | Modelos separados para creacion, actualizacion y lectura de cada recurso, con validaciones propias (rangos, longitudes, formatos, valores permitidos). Ningun endpoint trabaja con diccionarios sueltos. |
| Manejo de errores         | Excepciones propias de negocio, traducidas a respuestas HTTP con codigos apropiados (404, 409, 422, entre otros) y mensajes claros.                                                                      |
| Uso de routers            | Cada rema de recursos (desarrolladora, socio, etc.) tendra su propio router con responsabilidad propia.                                                                                                  |

## Requisitos de la API

Cada recurso principal (desarrolladora, consola, etc.) expone las operaciones de alta, listado, consulta por identificador, modificacion y baja. Las rutas con identificador son dinamicas y las consultas de listado aceptan filtros por parametros de consulta.

### Filtros de consulta obligatorios

| Recurso  | Filtros que debe aceptar el listado                                                                                |
| -------- | ------------------------------------------------------------------------------------------------------------------ |
| juego    | Texto parcial en el titulo, genero, consola, desarrolladora, rango de anios de lanzamiento, clasificacion por edad |
| ejemplar | Juego, consola, estado de conservacion, disponibilidad                                                             |
| socio    | Texto parcial en nombre o apellido, activo o inactivo                                                              |
| prestamo | Socio, ejemplar, estado (vigente, devuelto, vencido)                                                               |

### Rutas dinamicas y consultas relacionadas

Ademas del CRUD, la API debe resolver, como minimo, estas consultas que combinan varias tablas:

- Juegos de una desarrolladora, identificada por ruta dinamica.
- Consolas en las que esta disponible un juego, y juegos disponibles para una consola.
- Generos de un juego, y juegos de un genero.
- Ejemplares de un juego con su disponibilidad actual.
- Historial de prestamos de un socio, con el titulo del juego y la consola de cada ejemplar.
- Prestamos vencidos a la fecha, con los datos del socio y del juego.
- Ficha completa de un juego: desarrolladora, consolas, generos y cantidad de ejemplares disponibles en una sola respuesta.

Las respuestas de estas consultas deben usar modelos Pydantic de lectura propios que anidan la informacion relacionada, sin exponer identificadores internos innecesarios.

## Reglas de negocio

Las siguientes reglas deben implementarse en los managers y nunca en los endpoints ni solo en la base de datos:

1. Un ejemplar no disponible no puede prestarse.
2. Solo un socio activo puede retirar prestamos.
3. Un socio no puede tener mas de 3 prestamos vigentes a la vez.
4. Al registrar un prestamo, el ejemplar pasa a no disponible; al registrar la devolucion, vuelve a estar disponible. Ambos cambios deben ser consistentes entre si.
5. No se puede registrar la devolucion de un prestamo ya devuelto.
6. Un socio con prestamos vencidos no puede retirar nuevos ejemplares.
7. No se puede eliminar una desarrolladora, consola o genero que tenga juegos asociados, ni un juego con ejemplares prestados.
8. El anio de lanzamiento de un juego no puede ser anterior al de la consola mas antigua en la que figura.

## Entregables y restricciones

Se entrega un repositorio con el proyecto completo y un documento breve de disenio. 

Contenido del repositorio:

- Codigo fuente organizado por capas (routers, interfaces, managers, modelos, configuracion, env.example de ser necesario).
- Archivo de dependencias y un README con instrucciones para instalar y ejecutar el proyecto.
- Diagrama entidad-relacion del modelo.

Documento de disenio (maximo 4 paginas):

- Justificacion de las politicas de borrado de cada clave foranea.
- Descripcion de las interfaces y de como se conectan managers, dependencias y endpoints.
- Lista de endpoints con sus filtros.
- Decisiones tomadas ante ambiguedades del enunciado.

## Evaluacion y defensa oral

La nota final combina la calidad del proyecto entregado y la defensa oral.

