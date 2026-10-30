# Dispensación — Receta Ya

Repositorio local independiente del microservicio de dispensaciones. Usa Java 21, Spring Boot 4.1.1, H2, Flyway y JWT emitidos por Auth. Escucha en `127.0.0.1:8083`.

## Requisitos de la evaluación

La pauta solicita mostrar endpoints REST, capas internas, mapeo JPA, `pom.xml`, ramas Git, respuestas de Postman, configuración relacional, Maven/JAR y arranque local reproducible. El video conjunto debe durar de 3 a 8 minutos y mostrar resultados reales con voces y audio claros.

La API permite solicitar una dispensación y consultar las propias. **La integración local aún no valida una receta reservada desde Recetas**: con la configuración de demostración, el intento de entrega responde `503`. Ese código debe mostrarse como limitación, nunca como dispensación exitosa.

## Obtener el código

Este repositorio **aún no tiene remoto de GitHub**. Desde su raíz local se puede comprobar la clonación con los siguientes comandos. Tras publicarlo, sustituir `.` por su URL real.

### Windows (PowerShell)

```powershell
git clone . ..\dispensacion-copia
Set-Location ..\dispensacion-copia
git switch develop
java -version
```

### macOS y Linux (Bash)

```bash
git clone . ../dispensacion-copia
cd ../dispensacion-copia
git switch develop
java -version
```

Se necesita JDK 21, Git, puerto `8083` libre y acceso a Maven Central para la primera descarga. Maven Wrapper viene incluido; en macOS y Linux usar `bash ./mvnw`.

## Compilar, probar y empaquetar

Desde la raíz de este repositorio, donde está `pom.xml`:

```powershell
# Windows
.\mvnw.cmd clean verify
Get-Item .\target\dispensacion-0.0.1-SNAPSHOT.jar
```

```bash
# macOS y Linux
bash ./mvnw clean verify
ls -lh target/dispensacion-0.0.1-SNAPSHOT.jar
```

`verify` compila, ejecuta pruebas y empaqueta. Para IE7 también puede mostrarse `clean package`. Comprobar `BUILD SUCCESS` y el JAR. Detener cualquier proceso que use ese JAR antes de ejecutar `clean`.

## Configurar y ejecutar

Usar el **mismo** emisor y secreto JWT que Auth, generado localmente con al menos 32 caracteres UTF-8. No publicarlo. Para la demostración local, mantener SQS deshabilitado.

```powershell
# Windows: terminal de Dispensación
$env:RECETAYA_JWT_EMISOR = 'http://localhost:8081'
$env:RECETAYA_JWT_SECRETO = Read-Host 'El mismo secreto JWT usado en Auth'
$env:RECETAYA_SQS_HABILITADA = 'false'
java -jar .\target\dispensacion-0.0.1-SNAPSHOT.jar
```

```bash
# macOS y Linux: terminal de Dispensación
export RECETAYA_JWT_EMISOR='http://localhost:8081'
read -r -s -p 'El mismo secreto JWT usado en Auth: ' RECETAYA_JWT_SECRETO; printf '\n'
export RECETAYA_JWT_SECRETO
export RECETAYA_SQS_HABILITADA=false
java -jar target/dispensacion-0.0.1-SNAPSHOT.jar
```

Ejecutar desde la raíz: la URL H2 configurada en `application.yaml` es relativa a `./data/`. Verificar `GET http://localhost:8083/actuator/health`: debe responder `200` y `UP`.

## Pruebas en Postman

Iniciar Auth y usar JWT de un usuario `FARMACEUTICO`. La receta de prueba debe crearse antes en Recetas. Ocultar el token y usar datos ficticios.

| Método | Ruta | Resultado esperado en la demostración local |
|---|---|---|
| `GET` | `/api/v1/dispensaciones` | `200`, lista propia, posiblemente vacía. |
| `GET` | `/api/v1/dispensaciones/{id}` | Consulta una dispensación existente. |
| `GET` | `/api/v1/dispensaciones/{id}/estado` | Consulta su estado. |
| `GET` | `/api/v1/dispensaciones/{id}/detalle` | Consulta sus medicamentos. |
| `POST` | `/api/v1/dispensaciones` | `503` esperado con el validador de Recetas no disponible. |

Cuerpo para `POST`, usando el UUID obtenido al crear la receta:

```json
{"recetaId":"<UUID de receta>"}
```

Mostrar el `200` de la lista y el código/mensaje **real** del intento de dispensación. No afirmar que se entregaron medicamentos ni que se consumieron existencias. Una petición con JWT de médico debe ser denegada por permisos (`403`).

## Base de datos y estructura

- `pom.xml` incluye Web MVC, Security, JPA, H2, Flyway, Validation, Actuator, Jackson, SQS y pruebas.
- `src/main/resources/application.yaml` configura H2, Flyway, `ddl-auto: validate`, JWT, puerto y consola H2 local.
- `src/main/resources/bd/migracion/` define tablas de dispensación, detalles y solicitudes de consumo.
- `entidad/Dispensacion.java` tiene detalles con `@OneToMany`; `DispensacionDetalle` referencia su dispensación con `@ManyToOne`.
- `repositorio/`, `servicio/`, `controlador/` y `transferencia/` separan persistencia, reglas, rutas REST y cuerpos JSON.

La consola H2 local está en `http://localhost:8083/consola-h2` según la configuración actual. Consultar la URL, usuario y contraseña en `application.yaml` sin mostrarlos durante la grabación. El `503` del flujo local puede impedir que se cree una fila de dispensación; no presentar filas inexistentes como evidencia de persistencia.

## Ramas y entrega

Las ramas locales `main`, `develop` y `feature/dispensacion-service` se crearon para continuar el desarrollo separado. El historial del código se extrajo del repositorio original; no se ha configurado remoto ni se ha publicado este repositorio. Antes de entregar, completar y comprobar su URL de GitHub, mostrar `git branch -a` y `git log --oneline`, y aportar el enlace real en AVA y al docente.
