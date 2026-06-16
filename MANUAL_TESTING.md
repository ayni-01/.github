# Manual de Pruebas — Somos Ayni Microservicios

Guía paso a paso para probar los 7 microservicios de Somos Ayni usando Postman. Cubre tanto el entorno local como el entorno desplegado en Render.

---

## Requisitos previos

| Herramienta | Versión mínima | Descarga |
|---|---|---|
| Postman | Cualquiera | [postman.com/downloads](https://www.postman.com/downloads) |
| Java 21 + Maven | Solo para local | [adoptium.net](https://adoptium.net) |
| Docker | Solo para local | [docker.com](https://www.docker.com) |

---

## 1. Configurar Postman

### 1.1 Importar la colección

1. Abrir Postman
2. Clic en **Import**
3. Pegar la URL directa de la colección:
   ```
   https://raw.githubusercontent.com/ayni-01/.github/main/somos-ayni-microservices.postman_collection.json
   ```
4. Clic en **Import**

La colección aparece con 7 carpetas (una por servicio) + Health Checks.

---

### 1.2 Crear los Environments

Debes crear **dos environments**: uno para pruebas locales y otro para Render.

#### Environment: `Ayni — Local`

En Postman → **Environments → Add** → nombre: `Ayni — Local`

| Variable | Valor |
|---|---|
| `BASE_IDENTIDAD` | `http://localhost:8081` |
| `BASE_PERFILES` | `http://localhost:8082` |
| `BASE_RETOS` | `http://localhost:8083` |
| `BASE_POSTULACIONES` | `http://localhost:8084` |
| `BASE_HABILIDADES` | `http://localhost:8085` |
| `BASE_NOTIFICACIONES` | `http://localhost:8086` |
| `BASE_METRICAS` | `http://localhost:8087` |
| `TOKEN_EMPRESA` | *(dejar vacío — se rellena automáticamente)* |
| `TOKEN_TALENTO` | *(dejar vacío — se rellena automáticamente)* |
| `ID_EMPRESA` | *(dejar vacío — se rellena automáticamente)* |
| `ID_TALENTO` | *(dejar vacío — se rellena automáticamente)* |
| `RETO_ID` | *(dejar vacío — se rellena automáticamente)* |
| `POSTULACION_ID` | *(dejar vacío — se rellena automáticamente)* |

Guardar → **Save**.

#### Environment: `Ayni — Render`

En Postman → **Environments → Add** → nombre: `Ayni — Render`

| Variable | Valor |
|---|---|
| `BASE_IDENTIDAD` | `https://ayni-identidad-service.onrender.com` |
| `BASE_PERFILES` | `https://ayni-perfiles-service.onrender.com` |
| `BASE_RETOS` | `https://ayni-retos-service.onrender.com` |
| `BASE_POSTULACIONES` | `https://ayni-postulaciones-service.onrender.com` |
| `BASE_HABILIDADES` | `https://ayni-habilidades-service.onrender.com` |
| `BASE_NOTIFICACIONES` | `https://ayni-notificaciones-service.onrender.com` |
| `BASE_METRICAS` | `https://ayni-metricas-service.onrender.com` |
| `TOKEN_EMPRESA` | *(dejar vacío)* |
| `TOKEN_TALENTO` | *(dejar vacío)* |
| `ID_EMPRESA` | *(dejar vacío)* |
| `ID_TALENTO` | *(dejar vacío)* |
| `RETO_ID` | *(dejar vacío)* |
| `POSTULACION_ID` | *(dejar vacío)* |

Guardar → **Save**.

---

### 1.3 Seleccionar el environment activo

En la esquina superior derecha de Postman, selecciona el environment según dónde quieras probar:

```
[No Environment ▼]  →  [Ayni — Local ▼]
                    →  [Ayni — Render ▼]
```

> **Nota Render:** el plan gratuito duerme los servicios tras 15 min de inactividad. El primer request puede tardar hasta 60 segundos. Si ves timeout, espera y reintenta.

---

## 2. Flujo de pruebas completo

Sigue este orden exacto. Los scripts de test automáticos guardan los tokens e IDs en las variables del environment.

---

### Paso 1 — Health Checks

Antes de cualquier prueba, verifica que todos los servicios estén activos.

Carpeta: **Health Checks**

Ejecuta los 7 requests de health check. Todos deben responder:

```json
{ "status": "UP" }
```

**HTTP esperado:** `200 OK`

> Si alguno responde `503` o no responde, el servicio aún está iniciando. Espera 30 segundos y reintenta.

---

### Paso 2 — Registro de usuarios

Carpeta: **1 — Identidad (Auth)**

#### 2.1 Registro Empresa

**Request:** `Registro Empresa`
```json
{
  "email": "empresa@somosayni.com",
  "password": "Pass1234!",
  "rol": "EMPRESA"
}
```

**Respuesta esperada `201 Created`:**
```json
{
  "usuarioId": "8940856d-386f-...",
  "email": "empresa@somosayni.com",
  "token": "eyJhbGciOiJIUzUxMiJ9..."
}
```

✅ El script guarda automáticamente `TOKEN_EMPRESA` e `ID_EMPRESA` en el environment.

---

#### 2.2 Registro Talento

**Request:** `Registro Talento`
```json
{
  "email": "talento@somosayni.com",
  "password": "Pass1234!",
  "rol": "TALENTO"
}
```

**Respuesta esperada `201 Created`:**
```json
{
  "usuarioId": "5f63b2d8-...",
  "email": "talento@somosayni.com",
  "token": "eyJhbGciOiJIUzUxMiJ9..."
}
```

✅ Guarda automáticamente `TOKEN_TALENTO` e `ID_TALENTO`.

---

#### 2.3 Login (opcional — para renovar token)

**Request:** `Login Empresa`
```json
{
  "email": "empresa@somosayni.com",
  "password": "Pass1234!"
}
```

**Respuesta esperada `200 OK`:**
```json
{
  "usuarioId": "8940856d-...",
  "email": "empresa@somosayni.com",
  "token": "eyJhbGciOiJIUzUxMiJ9..."
}
```

---

### Paso 3 — Crear perfiles

Carpeta: **2 — Perfiles**

#### 3.1 Perfil de Empresa

**Request:** `Crear Perfil Empresa`

Header automático: `Authorization: Bearer {{TOKEN_EMPRESA}}`

```json
{
  "razonSocial": "Tech Perú SAC",
  "ruc": "20123456789",
  "sector": "Tecnología"
}
```

**Respuesta esperada `201 Created`:**
```json
{
  "id": "4bbd4f05-...",
  "usuarioId": "8940856d-...",
  "razonSocial": "Tech Perú SAC",
  "ruc": "20123456789",
  "estadoValidacion": "PENDIENTE"
}
```

---

#### 3.2 Perfil de Talento

**Request:** `Crear Perfil Talento`

Header automático: `Authorization: Bearer {{TOKEN_TALENTO}}`

```json
{
  "nombreCompleto": "Juan Pérez"
}
```

**Respuesta esperada `201 Created`:**
```json
{
  "id": "71f09979-...",
  "usuarioId": "5f63b2d8-...",
  "nombreCompleto": "Juan Pérez",
  "disponibilidad": "A_CONSULTAR"
}
```

---

### Paso 4 — Publicar un reto

Carpeta: **3 — Retos**

**Request:** `Publicar Reto`

Header automático: `Authorization: Bearer {{TOKEN_EMPRESA}}`

```json
{
  "titulo": "Desarrollar API REST con Spring Boot",
  "descripcion": "Crear una API RESTful completa con autenticación JWT, documentación Swagger y pruebas unitarias",
  "empresaId": "{{ID_EMPRESA}}",
  "modalidad": "REMOTO",
  "duracionDias": 30
}
```

**Respuesta esperada `201 Created`:**
```json
{
  "id": "a82ac793-...",
  "empresaId": "8940856d-...",
  "titulo": "Desarrollar API REST con Spring Boot",
  "estado": "ACTIVO",
  "cuposDisponibles": 10
}
```

✅ El script guarda `RETO_ID` automáticamente.

---

#### 4.1 Listar retos disponibles

**Request:** `Listar Retos`

Header automático: `Authorization: Bearer {{TOKEN_TALENTO}}`

**Respuesta esperada `200 OK`:**
```json
[
  {
    "id": "a82ac793-...",
    "titulo": "Desarrollar API REST con Spring Boot",
    "estado": "ACTIVO"
  }
]
```

---

### Paso 5 — Postular al reto

Carpeta: **4 — Postulaciones**

**Request:** `Postular a Reto`

Header automático: `Authorization: Bearer {{TOKEN_TALENTO}}`

```json
{
  "retoId": "{{RETO_ID}}",
  "urlSolucion": "https://github.com/juanperez/api-rest-solution"
}
```

**Respuesta esperada `201 Created`:**
```json
{
  "id": "a703c7aa-...",
  "talentoId": "5f63b2d8-...",
  "retoId": "a82ac793-...",
  "urlSolucion": "https://github.com/juanperez/api-rest-solution",
  "fechaEnvio": "2026-06-16T...",
  "estado": "EN_REVISION"
}
```

✅ El script guarda `POSTULACION_ID` automáticamente.

---

#### 5.1 Ver postulaciones del reto (vista empresa)

**Request:** `Ver Postulaciones del Reto`

Header automático: `Authorization: Bearer {{TOKEN_EMPRESA}}`

**Respuesta esperada `200 OK`:**
```json
[
  {
    "id": "a703c7aa-...",
    "talentoId": "5f63b2d8-...",
    "urlSolucion": "https://github.com/...",
    "estado": "EN_REVISION"
  }
]
```

---

### Paso 6 — Evaluar postulación

**Request:** `Evaluar Postulación (APROBADO)`

Header automático: `Authorization: Bearer {{TOKEN_EMPRESA}}`

```json
{
  "puntuacion": 92,
  "feedback": "Excelente implementación con Spring Boot y JWT",
  "resultado": "APROBADO"
}
```

**Respuesta esperada `200 OK`:**
```json
{
  "id": "aebee461-...",
  "postulacionId": "a703c7aa-...",
  "reclutadorId": "8940856d-...",
  "puntuacion": 92,
  "feedback": "Excelente implementación con Spring Boot y JWT",
  "resultado": "APROBADO"
}
```

> Para rechazar: usar `Evaluar Postulación (RECHAZADO)` con `"resultado": "RECHAZADO"`.

---

### Paso 7 — Otorgar insignia

Carpeta: **5 — Habilidades e Insignias**

**Request:** `Otorgar Insignia al Talento`

Header automático: `Authorization: Bearer {{TOKEN_EMPRESA}}`

```json
{
  "talentoId": "{{ID_TALENTO}}",
  "retoId": "{{RETO_ID}}",
  "titulo": "API REST con Spring Boot",
  "tipo": "VERIFICADO"
}
```

**Respuesta esperada `201 Created`:**
```json
{
  "id": "396d6556-...",
  "talentoId": "5f63b2d8-...",
  "retoId": "a82ac793-...",
  "titulo": "API REST con Spring Boot",
  "tipo": "VERIFICADO",
  "fechaOtorgada": "2026-06-16"
}
```

---

#### 7.1 Ver portafolio del talento

**Request:** `Ver Portafolio Talento`

**Respuesta esperada `200 OK`:**
```json
{
  "habilidades": [],
  "insignias": [
    {
      "titulo": "API REST con Spring Boot",
      "tipo": "VERIFICADO"
    }
  ]
}
```

---

### Paso 8 — Enviar notificación

Carpeta: **6 — Notificaciones**

**Request:** `Enviar Notificación`

Header automático: `Authorization: Bearer {{TOKEN_EMPRESA}}`

```json
{
  "destinatarioId": "{{ID_TALENTO}}",
  "tipo": "APROBADO",
  "mensaje": "¡Felicitaciones! Tu postulación al reto fue aprobada."
}
```

**Respuesta esperada `201 Created`:**
```json
{
  "id": "551abee9-...",
  "destinatarioId": "5f63b2d8-...",
  "tipo": "APROBADO",
  "mensaje": "¡Felicitaciones! Tu postulación al reto fue aprobada.",
  "leida": false,
  "fechaCreacion": "2026-06-16T..."
}
```

---

#### 8.1 Ver notificaciones del talento

**Request:** `Ver Mis Notificaciones`

Header automático: `Authorization: Bearer {{TOKEN_TALENTO}}`

**Respuesta esperada `200 OK`:**
```json
[
  {
    "id": "551abee9-...",
    "tipo": "APROBADO",
    "mensaje": "¡Felicitaciones!...",
    "leida": false
  }
]
```

---

### Paso 9 — Consultar métricas

Carpeta: **7 — Métricas**

#### 9.1 Métricas generales de la empresa

**Request:** `Métricas de la Empresa`

Header automático: `Authorization: Bearer {{TOKEN_EMPRESA}}`

**Respuesta esperada `200 OK`:**
```json
{
  "empresaId": "8940856d-...",
  "retosActivos": 1,
  "nuevasPostulaciones": 1,
  "talentosEvaluados": 1,
  "talentosAprobados": 1
}
```

---

#### 9.2 Embudo de conversión

**Request:** `Embudo de Conversión`

**Respuesta esperada `200 OK`:**
```json
[
  {
    "retoId": "a82ac793-...",
    "tituloReto": "Desarrollar API REST con Spring Boot",
    "postulados": 1,
    "evaluados": 1,
    "aprobados": 1,
    "tasaConversion": 100.0
  }
]
```

---

## 3. Tipos de errores comunes

| HTTP | Causa | Solución |
|---|---|---|
| `401 Unauthorized` | Token no enviado o expirado | Ejecutar `Login Empresa` o `Login Talento` para renovar el token |
| `403 Forbidden` | Error de negocio (registro duplicado, perfil ya existe) | Usar un email distinto o verificar que el recurso no existe ya |
| `400 Bad Request` | Campos inválidos o faltantes en el body | Revisar que el JSON tenga todos los campos requeridos |
| `404 Not Found` | El ID en la URL no existe | Verificar que `RETO_ID` o `POSTULACION_ID` están seteados |
| Timeout / no responde | Servicio dormido (plan gratuito Render) | Esperar 60 segundos y reintentar |

---

## 4. Valores válidos de enums

### `rol` (registro de usuario)
| Valor | Descripción |
|---|---|
| `TALENTO` | Estudiante o egresado que postula a retos |
| `EMPRESA` | Empresa que publica retos |
| `ADMIN` | Administrador del sistema |

### `tipo` (notificaciones)
| Valor | Descripción |
|---|---|
| `NUEVA_POSTULACION` | Nueva candidatura recibida |
| `APROBADO` | Postulación aprobada |
| `RECHAZADO` | Postulación rechazada |
| `RETO_CERRADO` | Un reto fue cerrado |

### `resultado` (evaluar postulación)
| Valor | Descripción |
|---|---|
| `APROBADO` | Talento supera el reto |
| `RECHAZADO` | Talento no supera el reto |

### `tipo` (insignias)
| Valor | Descripción |
|---|---|
| `VERIFICADO` | Habilidad verificada mediante reto |

---

## 5. URLs de Swagger por servicio

Accede a la documentación interactiva de cada servicio (requiere que el servicio esté activo):

| Servicio | Swagger UI |
|---|---|
| identidad | https://ayni-identidad-service.onrender.com/swagger-ui.html |
| perfiles | https://ayni-perfiles-service.onrender.com/swagger-ui.html |
| retos | https://ayni-retos-service.onrender.com/swagger-ui.html |
| postulaciones | https://ayni-postulaciones-service.onrender.com/swagger-ui.html |
| habilidades | https://ayni-habilidades-service.onrender.com/swagger-ui.html |
| notificaciones | https://ayni-notificaciones-service.onrender.com/swagger-ui.html |
| metricas | https://ayni-metricas-service.onrender.com/swagger-ui.html |

Para autenticarte en Swagger:
1. Ejecutar `POST /api/v1/auth/login` → copiar el `token`
2. Clic en **Authorize** (candado arriba a la derecha)
3. Pegar: `Bearer eyJhbGciOiJIUzUxMiJ9...`
4. Clic en **Authorize** → **Close**
