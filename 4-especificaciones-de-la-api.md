# 4. Especificación de la API

## 4.0. Naturaleza de la API


La aplicación expone una **API interna con patrón Backend for Frontend**, implementada como Route Handlers de Next.js bajo `/api`. No es una API pública: no está pensada para integradores externos, no ofrece compatibilidad hacia atrás garantizada y su único consumidor es el propio frontend. La API pública figura en la fase de escala del producto y, cuando llegue, será una superficie distinta y versionada, no esta.

Esa distinción tiene una consecuencia práctica: la especificación que sigue documenta un contrato **interno pero estricto**. Estricto porque es la frontera de seguridad del sistema —es donde se valida la entrada, se resuelve la identidad y se autoriza el acceso— y porque el desarrollo asistido por IA necesita un contrato explícito para no inventar formas de petición distintas en cada pantalla.

**Tres cosas que deliberadamente no pasan por esta API:**

| Fuera del alcance | Por dónde va | Motivo |
|---|---|---|
| Autenticación del anfitrión | SDK de Supabase desde el navegador | Es el proveedor quien gestiona credenciales y sesión; reimplementarlo añadiría superficie de ataque sin ganancia |
| Subida de ficheros | Del navegador a Storage con URL firmada | Evita el límite de tamaño de payload de las funciones y ahorra ancho de banda de cómputo. La API solo emite y autoriza la URL |
| Sincronización en tiempo real | WebSocket contra el servidor de sincronización | Es un protocolo con estado y no encaja en REST. La API solo emite el token que permite conectarse |

---

## 4.1. Convenciones

**Identidad.** Dos esquemas de autenticación distintos, nunca intercambiables. El anfitrión presenta la cookie de sesión del proveedor de autenticación. El invitado presenta una cookie de sesión propia, firmada por el servidor, con alcance a un único canvas y un permiso. Ambas son `HttpOnly`. El identificador de canvas de un invitado se toma siempre del contenido verificado de su cookie, nunca de la ruta ni del cuerpo, y por eso los endpoints de invitado no reciben ningún identificador como parámetro.

**Formato de error.** Todas las respuestas de error comparten forma: un objeto `error` con un `code` estable en inglés, pensado para que el cliente decida, y un `message` ya traducido al idioma de la petición, pensado para mostrarse. El código es el contrato; el mensaje puede cambiar sin previo aviso.

**Idioma.** Se resuelve por el mismo orden que el resto de la aplicación: parámetro explícito, cookie de preferencia y cabecera `Accept-Language`.

**Limitación de tasa.** Las respuestas afectadas incluyen cabeceras `RateLimit-Limit`, `RateLimit-Remaining` y `RateLimit-Reset`. Al superar el límite se devuelve `429` con `Retry-After`, sin revelar qué límite concreto se alcanzó ni cuánto consumo queda.

**Caché.** Ninguna respuesta de `/api` es cacheable. Todas dependen de identidad o de estado revocable. Lo cacheable del sistema son los ficheros, direccionados por contenido, nunca por enlace.

**Paginación.** Por cursor opaco, no por número de página, para que el listado del panel sea estable mientras se editan canvases.

**Idempotencia.** Las escrituras que crean recursos aceptan `Idempotency-Key`, de modo que un reintento por red inestable —el caso normal en móvil, que es donde vive el invitado— no duplique un canvas ni un enlace.

**Versionado.** Sin prefijo de versión: al ser un BFF, cliente y servidor se despliegan juntos. La regla que lo hace seguro está en el proceso de despliegue: los cambios de contrato deben ser compatibles hacia atrás durante una ventana de despliegue, porque un cliente antiguo puede seguir vivo en la pestaña de alguien.

---

## 4.2. Mapa de endpoints

| Método y ruta | Consumidor | Propósito | Fase |
|---|---|---|---|
| `GET /api/health` | Monitorización | Comprueba base de datos, almacenamiento y sincronización | A |
| `GET /api/me` | Anfitrión | Perfil, límites del plan y consumo actual | A |
| `PATCH /api/me` | Anfitrión | Idioma, nombre visible y consentimiento comercial | A |
| `POST /api/me/exports` | Anfitrión | Solicita la exportación de todos sus datos | A |
| `GET /api/me/exports/{exportId}` | Anfitrión | Estado y enlace de descarga de la exportación | A |
| `DELETE /api/me` | Anfitrión | Borrado de cuenta y de todo su contenido | A |
| `GET /api/canvases` | Anfitrión | Listado paginado del panel | A |
| `POST /api/canvases` | Anfitrión | Crea un canvas, sujeto a cuota | A |
| `GET /api/canvases/{canvasId}` | Anfitrión | Metadatos y documento de un canvas | A |
| `PATCH /api/canvases/{canvasId}` | Anfitrión | Renombra un canvas | A |
| `PUT /api/canvases/{canvasId}/document` | Anfitrión | Autoguardado del documento | A |
| `DELETE /api/canvases/{canvasId}` | Anfitrión | Borrado lógico | A |
| `POST /api/canvases/{canvasId}/restore` | Anfitrión | Restaura desde la papelera | A |
| `POST /api/canvases/{canvasId}/render` | Anfitrión | Registra la render generada en cliente | A |
| `GET /api/canvases/{canvasId}/share-links` | Anfitrión | Enlaces activos de un canvas | A |
| `POST /api/canvases/{canvasId}/share-links` | Anfitrión | Genera un enlace de invitado | A |
| `PATCH /api/share-links/{shareLinkId}` | Anfitrión | Cambia permiso o caducidad | B |
| `POST /api/share-links/{shareLinkId}/revoke` | Anfitrión | Revoca de inmediato | A |
| `POST /api/guest/session` | Invitado | Canjea el token del enlace por una sesión efímera | A |
| `POST /api/guest/session/renew` | Invitado | Renueva la sesión revalidando el enlace | A |
| `GET /api/guest/canvas` | Invitado | Devuelve la render y los metadatos visibles | A |
| `POST /api/uploads` | Ambos | Emite una URL firmada de subida | A |
| `POST /api/rooms/token` | Ambos | Emite el token de conexión a la sala | B |

---

## 4.3. Especificación OpenAPI

```yaml
openapi: 3.1.0

info:
  title: API interna de la plataforma de canvas colaborativo
  version: "1.0.0"
  description: |
    Backend for Frontend de la aplicación. Consumidor único: el propio frontend.
    No es una API pública y no garantiza compatibilidad hacia atrás fuera de la
    ventana de despliegue.
    Toda petición de invitado resuelve su canvas a partir de la cookie de sesión
    verificada, nunca de parámetros de la petición.

servers:
  - url: https://app.example.com/api
    description: Producción
  - url: https://staging.example.com/api
    description: Preproducción

tags:
  - name: Sistema
  - name: Cuenta
  - name: Canvases
  - name: Documento
  - name: Compartición
  - name: Invitado
  - name: Ficheros
  - name: Tiempo real

security:
  - hostSession: []

paths:

  /health:
    get:
      tags: [Sistema]
      summary: Estado real de las dependencias
      description: |
        Comprueba de forma efectiva la base de datos, el almacenamiento y, en
        MVP-B, el servicio de sincronización. No devuelve 200 sin verificarlas.
      security: []
      responses:
        "200":
          description: Todas las dependencias responden
          content:
            application/json:
              schema: { $ref: "#/components/schemas/Health" }
        "503":
          description: Alguna dependencia no responde
          content:
            application/json:
              schema: { $ref: "#/components/schemas/Health" }

  /me:
    get:
      tags: [Cuenta]
      summary: Perfil, límites y consumo
      responses:
        "200":
          description: Datos del usuario autenticado
          content:
            application/json:
              schema: { $ref: "#/components/schemas/Me" }
        "401": { $ref: "#/components/responses/NoAutenticado" }
    patch:
      tags: [Cuenta]
      summary: Actualiza preferencias del perfil
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              additionalProperties: false
              properties:
                displayName: { type: string, maxLength: 100 }
                locale: { $ref: "#/components/schemas/Locale" }
                marketingOptIn: { type: boolean }
      responses:
        "200":
          description: Perfil actualizado
          content:
            application/json:
              schema: { $ref: "#/components/schemas/Profile" }
        "401": { $ref: "#/components/responses/NoAutenticado" }
        "422": { $ref: "#/components/responses/EntradaInvalida" }
    delete:
      tags: [Cuenta]
      summary: Borra la cuenta y todo su contenido
      description: |
        Operación asíncrona con reintentos: elimina datos, ficheros y salas de
        sincronización. Se registra como evento de seguridad.
      parameters:
        - name: X-Confirm
          in: header
          required: true
          schema: { type: string, enum: [DELETE-MY-ACCOUNT] }
      responses:
        "202": { description: Borrado aceptado y en curso }
        "401": { $ref: "#/components/responses/NoAutenticado" }

  /me/exports:
    post:
      tags: [Cuenta]
      summary: Solicita la exportación de los datos personales
      responses:
        "202":
          description: Exportación encolada
          content:
            application/json:
              schema:
                type: object
                required: [exportId, status]
                properties:
                  exportId: { type: string, format: uuid }
                  status: { type: string, enum: [pending] }
        "401": { $ref: "#/components/responses/NoAutenticado" }
        "429": { $ref: "#/components/responses/LimiteSuperado" }

  /me/exports/{exportId}:
    parameters:
      - $ref: "#/components/parameters/ExportId"
    get:
      tags: [Cuenta]
      summary: Estado de una exportación
      responses:
        "200":
          description: Estado actual
          content:
            application/json:
              schema:
                type: object
                required: [exportId, status]
                properties:
                  exportId: { type: string, format: uuid }
                  status: { type: string, enum: [pending, ready, failed] }
                  downloadUrl:
                    type: string
                    format: uri
                    description: URL firmada, presente solo si el estado es ready
                  expiresAt: { type: string, format: date-time }
        "401": { $ref: "#/components/responses/NoAutenticado" }
        "404": { $ref: "#/components/responses/NoEncontrado" }

  /canvases:
    get:
      tags: [Canvases]
      summary: Listado del panel
      parameters:
        - name: cursor
          in: query
          schema: { type: string }
          description: Cursor opaco devuelto por la página anterior
        - name: limit
          in: query
          schema: { type: integer, minimum: 1, maximum: 50, default: 20 }
        - name: includeDeleted
          in: query
          schema: { type: boolean, default: false }
          description: Incluye la papelera, que se conserva 30 días
      responses:
        "200":
          description: Página de resultados
          content:
            application/json:
              schema:
                type: object
                required: [items]
                properties:
                  items:
                    type: array
                    items: { $ref: "#/components/schemas/CanvasSummary" }
                  nextCursor:
                    type: [string, "null"]
        "401": { $ref: "#/components/responses/NoAutenticado" }
    post:
      tags: [Canvases]
      summary: Crea un canvas
      parameters:
        - $ref: "#/components/parameters/IdempotencyKey"
      requestBody:
        content:
          application/json:
            schema:
              type: object
              additionalProperties: false
              properties:
                title: { type: string, minLength: 1, maxLength: 200 }
      responses:
        "201":
          description: Canvas creado
          content:
            application/json:
              schema: { $ref: "#/components/schemas/CanvasSummary" }
        "401": { $ref: "#/components/responses/NoAutenticado" }
        "403":
          description: Cuota del plan agotada
          content:
            application/json:
              schema: { $ref: "#/components/schemas/Error" }
              examples:
                cuotaMensual:
                  value:
                    error:
                      code: QUOTA_MONTHLY_CANVASES_EXCEEDED
                      message: Has alcanzado el límite de sesiones de este mes.
                cuotaActivos:
                  value:
                    error:
                      code: QUOTA_ACTIVE_CANVASES_EXCEEDED
                      message: Tienes demasiadas sesiones abiertas.

  /canvases/{canvasId}:
    parameters:
      - $ref: "#/components/parameters/CanvasId"
    get:
      tags: [Canvases]
      summary: Metadatos y documento del canvas
      responses:
        "200":
          description: Canvas completo
          content:
            application/json:
              schema: { $ref: "#/components/schemas/CanvasDetail" }
        "401": { $ref: "#/components/responses/NoAutenticado" }
        "404": { $ref: "#/components/responses/NoEncontrado" }
    patch:
      tags: [Canvases]
      summary: Renombra el canvas
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              additionalProperties: false
              required: [title]
              properties:
                title: { type: string, minLength: 1, maxLength: 200 }
      responses:
        "200":
          description: Canvas actualizado
          content:
            application/json:
              schema: { $ref: "#/components/schemas/CanvasSummary" }
        "404": { $ref: "#/components/responses/NoEncontrado" }
        "422": { $ref: "#/components/responses/EntradaInvalida" }
    delete:
      tags: [Canvases]
      summary: Borrado lógico
      description: Recuperable durante 30 días; después se elimina de forma definitiva.
      responses:
        "204": { description: Enviado a la papelera }
        "404": { $ref: "#/components/responses/NoEncontrado" }

  /canvases/{canvasId}/restore:
    parameters:
      - $ref: "#/components/parameters/CanvasId"
    post:
      tags: [Canvases]
      summary: Restaura un canvas de la papelera
      responses:
        "200":
          description: Restaurado
          content:
            application/json:
              schema: { $ref: "#/components/schemas/CanvasSummary" }
        "404": { $ref: "#/components/responses/NoEncontrado" }
        "409":
          description: El canvas ya fue purgado definitivamente
          content:
            application/json:
              schema: { $ref: "#/components/schemas/Error" }

  /canvases/{canvasId}/document:
    parameters:
      - $ref: "#/components/parameters/CanvasId"
    put:
      tags: [Documento]
      summary: Autoguardado del documento
      description: |
        Sustitución completa del snapshot. El cliente envía la versión de esquema
        del motor para que sea posible migrar documentos antiguos.
        En MVP-B, con el canvas en modo sincronizado, este endpoint responde 409:
        la fuente de verdad es la sala.
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              additionalProperties: false
              required: [document, schemaVersion]
              properties:
                document: { type: object, description: Snapshot nativo del motor }
                schemaVersion: { type: integer, minimum: 1 }
      responses:
        "200":
          description: Guardado
          content:
            application/json:
              schema:
                type: object
                required: [updatedAt, sizeBytes]
                properties:
                  updatedAt: { type: string, format: date-time }
                  sizeBytes: { type: integer }
        "409":
          description: El canvas está en modo sincronizado
          content:
            application/json:
              schema: { $ref: "#/components/schemas/Error" }
        "413":
          description: El documento supera el tamaño máximo de 5 MB
          content:
            application/json:
              schema: { $ref: "#/components/schemas/Error" }
        "429": { $ref: "#/components/responses/LimiteSuperado" }

  /canvases/{canvasId}/render:
    parameters:
      - $ref: "#/components/parameters/CanvasId"
    post:
      tags: [Documento]
      summary: Registra la render generada en el navegador del anfitrión
      description: |
        La render se produce en cliente y se sube con URL firmada. Este endpoint
        solo confirma el resultado y actualiza la marca de versión, que es la
        clave con la que la render se sirve como recurso inmutable.
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              additionalProperties: false
              required: [storagePath, format]
              properties:
                storagePath: { type: string }
                format: { type: string, enum: [svg, png] }
      responses:
        "200":
          description: Render registrada
          content:
            application/json:
              schema:
                type: object
                properties:
                  renderGeneratedAt: { type: string, format: date-time }
        "404": { $ref: "#/components/responses/NoEncontrado" }
        "422": { $ref: "#/components/responses/EntradaInvalida" }

  /canvases/{canvasId}/share-links:
    parameters:
      - $ref: "#/components/parameters/CanvasId"
    get:
      tags: [Compartición]
      summary: Enlaces del canvas
      responses:
        "200":
          description: Enlaces existentes, activos y revocados
          content:
            application/json:
              schema:
                type: array
                items: { $ref: "#/components/schemas/ShareLink" }
        "404": { $ref: "#/components/responses/NoEncontrado" }
    post:
      tags: [Compartición]
      summary: Genera un enlace de invitado
      description: |
        El token completo solo se devuelve en esta respuesta. Después no vuelve a
        exponerse por ningún endpoint: es un secreto, no un identificador.
      parameters:
        - $ref: "#/components/parameters/IdempotencyKey"
      requestBody:
        content:
          application/json:
            schema:
              type: object
              additionalProperties: false
              properties:
                permission: { type: string, enum: [view, edit], default: view }
                expiresAt: { type: [string, "null"], format: date-time }
      responses:
        "201":
          description: Enlace creado
          content:
            application/json:
              schema:
                allOf:
                  - $ref: "#/components/schemas/ShareLink"
                  - type: object
                    required: [url]
                    properties:
                      url:
                        type: string
                        format: uri
                        description: URL completa con el token, mostrada una sola vez
        "403":
          description: El plan no permite enlaces de edición
          content:
            application/json:
              schema: { $ref: "#/components/schemas/Error" }

  /share-links/{shareLinkId}:
    parameters:
      - $ref: "#/components/parameters/ShareLinkId"
    patch:
      tags: [Compartición]
      summary: Cambia permiso o caducidad
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              additionalProperties: false
              properties:
                permission: { type: string, enum: [view, edit] }
                expiresAt: { type: [string, "null"], format: date-time }
      responses:
        "200":
          description: Enlace actualizado
          content:
            application/json:
              schema: { $ref: "#/components/schemas/ShareLink" }
        "404": { $ref: "#/components/responses/NoEncontrado" }

  /share-links/{shareLinkId}/revoke:
    parameters:
      - $ref: "#/components/parameters/ShareLinkId"
    post:
      tags: [Compartición]
      summary: Revoca el enlace de inmediato
      description: |
        Marca el enlace como revocado y, en MVP-B, ordena a la sala expulsar a los
        participantes que entraron con él. Las URL de imágenes ya firmadas siguen
        siendo válidas hasta que caduquen, en un plazo máximo de una hora.
      responses:
        "204": { description: Revocado }
        "404": { $ref: "#/components/responses/NoEncontrado" }

  /guest/session:
    post:
      tags: [Invitado]
      summary: Canjea el token del enlace por una sesión de invitado
      description: |
        Valida el enlace y devuelve una cookie de sesión firmada, con alcance a un
        único canvas y un permiso, sin crear ningún registro del invitado.
        Toda respuesta de fallo es idéntica e indistinguible, con el mismo tiempo
        de respuesta, para no revelar si un token existe.
      security: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [token]
              properties:
                token: { type: string, format: uuid }
      responses:
        "200":
          description: Sesión creada
          headers:
            Set-Cookie:
              schema: { type: string }
              description: guest_session, HttpOnly, Secure, SameSite=Lax, 60 minutos
          content:
            application/json:
              schema: { $ref: "#/components/schemas/GuestSession" }
        "404":
          description: Enlace inexistente, caducado o revocado
          content:
            application/json:
              schema: { $ref: "#/components/schemas/Error" }
        "429": { $ref: "#/components/responses/LimiteSuperado" }

  /guest/session/renew:
    post:
      tags: [Invitado]
      summary: Renueva la sesión de invitado
      description: |
        Revalida el enlace contra la base de datos. Es el mecanismo que hace que
        una revocación surta efecto pese a que la cookie siga siendo válida
        criptográficamente.
      security:
        - guestSession: []
      responses:
        "200":
          description: Sesión renovada
          content:
            application/json:
              schema: { $ref: "#/components/schemas/GuestSession" }
        "401":
          description: El enlace ya no es válido
          content:
            application/json:
              schema: { $ref: "#/components/schemas/Error" }

  /guest/canvas:
    get:
      tags: [Invitado]
      summary: Contenido visible para el invitado
      description: |
        El canvas se resuelve a partir de la cookie verificada. No admite ningún
        parámetro de identificación, por diseño.
      security:
        - guestSession: []
      responses:
        "200":
          description: Render y metadatos visibles
          content:
            application/json:
              schema: { $ref: "#/components/schemas/GuestCanvas" }
        "401":
          description: Sesión ausente, caducada o revocada
          content:
            application/json:
              schema: { $ref: "#/components/schemas/Error" }

  /uploads:
    post:
      tags: [Ficheros]
      summary: Emite una URL firmada de subida
      description: |
        Valida tipo, tamaño y cuota antes de firmar. El fichero viaja del navegador
        al almacenamiento sin pasar por la función. Un invitado solo puede solicitar
        subidas si su enlace es de edición.
      security:
        - hostSession: []
        - guestSession: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              additionalProperties: false
              required: [purpose, mimeType, sizeBytes]
              properties:
                purpose: { type: string, enum: [asset, render, thumbnail] }
                mimeType:
                  type: string
                  enum: [image/png, image/jpeg, image/webp, image/svg+xml]
                sizeBytes: { type: integer, minimum: 1 }
      responses:
        "201":
          description: URL firmada emitida
          content:
            application/json:
              schema:
                type: object
                required: [uploadUrl, storagePath, expiresAt]
                properties:
                  uploadUrl: { type: string, format: uri }
                  storagePath: { type: string }
                  expiresAt: { type: string, format: date-time }
        "403":
          description: Cuota de almacenamiento agotada o permiso insuficiente
          content:
            application/json:
              schema: { $ref: "#/components/schemas/Error" }
        "413":
          description: El fichero supera el máximo del plan
          content:
            application/json:
              schema: { $ref: "#/components/schemas/Error" }
        "429": { $ref: "#/components/responses/LimiteSuperado" }

  /rooms/token:
    post:
      tags: [Tiempo real]
      summary: Emite el token de conexión a la sala
      description: |
        Disponible en MVP-B. Revalida el enlace o la propiedad del canvas y emite un
        token de vida corta que el servidor de sincronización verifica. Se firma con
        una clave distinta de la de la cookie de invitado, para que comprometer el
        servidor de sincronización no permita falsificar sesiones.
      security:
        - hostSession: []
        - guestSession: []
      responses:
        "201":
          description: Token emitido
          content:
            application/json:
              schema:
                type: object
                required: [token, roomId, wsUrl, expiresAt, role]
                properties:
                  token: { type: string }
                  roomId: { type: string }
                  wsUrl: { type: string, format: uri }
                  role: { type: string, enum: [editor, viewer] }
                  expiresAt: { type: string, format: date-time }
        "403":
          description: Sala llena para el plan, o permiso insuficiente
          content:
            application/json:
              schema: { $ref: "#/components/schemas/Error" }
              examples:
                salaLlena:
                  value:
                    error:
                      code: ROOM_PARTICIPANT_LIMIT_REACHED
                      message: La sesión ha alcanzado el número máximo de participantes.
        "429": { $ref: "#/components/responses/LimiteSuperado" }

components:

  securitySchemes:
    hostSession:
      type: apiKey
      in: cookie
      name: sb-access-token
      description: Sesión del anfitrión gestionada por el proveedor de autenticación.
    guestSession:
      type: apiKey
      in: cookie
      name: guest_session
      description: |
        Sesión efímera del invitado. JWT firmado por el servidor con alcance a un
        canvas y un permiso. No contiene ningún dato personal.

  parameters:
    CanvasId:
      name: canvasId
      in: path
      required: true
      schema: { type: string, format: uuid }
    ShareLinkId:
      name: shareLinkId
      in: path
      required: true
      schema: { type: string, format: uuid }
    ExportId:
      name: exportId
      in: path
      required: true
      schema: { type: string, format: uuid }
    IdempotencyKey:
      name: Idempotency-Key
      in: header
      required: false
      schema: { type: string, maxLength: 128 }
      description: Evita duplicados si el cliente reintenta tras un fallo de red.

  responses:
    NoAutenticado:
      description: Falta la sesión o no es válida
      content:
        application/json:
          schema: { $ref: "#/components/schemas/Error" }
    NoEncontrado:
      description: |
        El recurso no existe o no pertenece a quien lo pide. Se responde lo mismo en
        ambos casos, para no confirmar la existencia de recursos ajenos.
      content:
        application/json:
          schema: { $ref: "#/components/schemas/Error" }
    EntradaInvalida:
      description: La petición no supera la validación de esquema
      content:
        application/json:
          schema: { $ref: "#/components/schemas/Error" }
    LimiteSuperado:
      description: Se ha superado el límite de tasa
      headers:
        Retry-After:
          schema: { type: integer }
          description: Segundos que deben transcurrir antes de reintentar
      content:
        application/json:
          schema: { $ref: "#/components/schemas/Error" }

  schemas:

    Locale:
      type: string
      enum: [es, en, de, nl]

    Error:
      type: object
      required: [error]
      properties:
        error:
          type: object
          required: [code, message]
          properties:
            code:
              type: string
              description: Código estable, en inglés. Es el contrato para el cliente.
            message:
              type: string
              description: Mensaje traducido, pensado para mostrarse. Puede cambiar.
            details:
              type: object
              description: Presente solo en errores de validación.

    Health:
      type: object
      required: [status, checks]
      properties:
        status: { type: string, enum: [ok, degraded, down] }
        checks:
          type: object
          properties:
            database: { type: boolean }
            storage: { type: boolean }
            realtime: { type: [boolean, "null"] }

    Profile:
      type: object
      required: [id, locale, plan]
      properties:
        id: { type: string, format: uuid }
        displayName: { type: [string, "null"] }
        locale: { $ref: "#/components/schemas/Locale" }
        plan: { type: string, enum: [free, pro, team] }
        marketingOptIn: { type: boolean }
        createdAt: { type: string, format: date-time }

    Me:
      type: object
      required: [profile, limits, usage]
      properties:
        profile: { $ref: "#/components/schemas/Profile" }
        limits:
          type: object
          properties:
            maxActiveCanvases: { type: integer }
            maxCanvasesPerMonth: { type: integer }
            maxParticipants: { type: integer }
            maxStorageMb: { type: integer }
            maxAssetMb: { type: integer }
            realtimeEnabled: { type: boolean }
            versionHistory: { type: boolean }
            whiteLabel: { type: boolean }
        usage:
          type: object
          properties:
            activeCanvases: { type: integer }
            canvasesCreatedThisMonth: { type: integer }
            storageBytesUsed: { type: integer }

    CanvasSummary:
      type: object
      required: [id, title, engine, updatedAt]
      properties:
        id: { type: string, format: uuid }
        title: { type: string }
        engine: { type: string, enum: [local, sync] }
        thumbnailUrl: { type: [string, "null"], format: uri }
        lastOpenedAt: { type: [string, "null"], format: date-time }
        updatedAt: { type: string, format: date-time }
        deletedAt: { type: [string, "null"], format: date-time }
        activeShareLinks: { type: integer }

    CanvasDetail:
      allOf:
        - $ref: "#/components/schemas/CanvasSummary"
        - type: object
          properties:
            document: { type: [object, "null"] }
            schemaVersion: { type: integer }
            sizeBytes: { type: integer }
            syncRoomId: { type: [string, "null"] }

    ShareLink:
      type: object
      required: [id, canvasId, permission, createdAt]
      properties:
        id: { type: string, format: uuid }
        canvasId: { type: string, format: uuid }
        permission: { type: string, enum: [view, edit] }
        expiresAt: { type: [string, "null"], format: date-time }
        revokedAt: { type: [string, "null"], format: date-time }
        createdAt: { type: string, format: date-time }
        viewCount: { type: integer, description: Visitas seudonimizadas registradas }

    GuestSession:
      type: object
      required: [permission, expiresAt]
      properties:
        permission: { type: string, enum: [view, edit] }
        expiresAt: { type: string, format: date-time }
        canvasTitle: { type: string }
        hostDisplayName: { type: [string, "null"] }

    GuestCanvas:
      type: object
      required: [title, permission]
      properties:
        title: { type: string }
        permission: { type: string, enum: [view, edit] }
        render:
          type: object
          description: Presente cuando el permiso es de solo lectura
          properties:
            url: { type: string, format: uri }
            format: { type: string, enum: [svg, png] }
            generatedAt: { type: string, format: date-time }
            width: { type: integer }
            height: { type: integer }
        realtime:
          type: object
          description: Presente cuando el permiso es de edición, en MVP-B
          properties:
            available: { type: boolean }
```

---

## 4.4. Ejemplos

### Compartir un canvas

```http
POST /api/canvases/7c3f.../share-links
Content-Type: application/json
Idempotency-Key: 4b1e-...

{ "permission": "view", "expiresAt": "2026-10-01T18:00:00Z" }
```

```http
HTTP/1.1 201 Created
Cache-Control: no-store

{
  "id": "0f21...",
  "canvasId": "7c3f...",
  "permission": "view",
  "expiresAt": "2026-10-01T18:00:00Z",
  "revokedAt": null,
  "createdAt": "2026-09-05T09:12:00Z",
  "viewCount": 0,
  "url": "https://app.example.com/s/8d7a1c2e-..."
}
```

El campo `url` aparece únicamente aquí. Un `GET` posterior sobre los enlaces del canvas devuelve todo menos el token: es un secreto, y una vez entregado no vuelve a exponerse.

### Entrar como invitado

```http
POST /api/guest/session
Content-Type: application/json

{ "token": "8d7a1c2e-..." }
```

```http
HTTP/1.1 200 OK
Set-Cookie: guest_session=eyJhbGciOi...; HttpOnly; Secure; SameSite=Lax; Max-Age=3600
Cache-Control: no-store

{
  "permission": "view",
  "expiresAt": "2026-09-05T10:12:00Z",
  "canvasTitle": "Retrospectiva de equipo",
  "hostDisplayName": "Marta"
}
```

Si el enlace estuviera revocado, caducado o no existiera, la respuesta sería un `404` idéntico en los tres casos y con el mismo tiempo de proceso. Distinguirlos permitiría a un atacante confirmar qué tokens existen.

### Obtener el contenido

```http
GET /api/guest/canvas
Cookie: guest_session=eyJhbGciOi...
```

```http
HTTP/1.1 200 OK
Cache-Control: no-store

{
  "title": "Retrospectiva de equipo",
  "permission": "view",
  "render": {
    "url": "https://storage.example.com/renders/7c3f.../1757062320.svg?token=...",
    "format": "svg",
    "generatedAt": "2026-09-05T09:32:00Z",
    "width": 2400,
    "height": 1600
  }
}
```

La respuesta de la API no es cacheable, pero la render sí lo es de forma indefinida: su URL incluye la marca de generación, de modo que cada versión del canvas es un recurso distinto e inmutable. Es lo que permite servir el contenido pesado desde caché sin comprometer la revocación del enlace.

### Autoguardado rechazado por límite de tasa

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 12
RateLimit-Limit: 60
RateLimit-Remaining: 0

{
  "error": {
    "code": "RATE_LIMITED",
    "message": "Demasiadas peticiones. Inténtalo de nuevo en unos segundos."
  }
}
```

El código no indica qué límite se alcanzó ni cuál de las dos capas lo aplicó. El cliente reacciona igual en ambos casos: espera y reintenta, apoyándose mientras tanto en su copia local.

---

## 4.5. Verificación del contrato

La especificación no es documentación acompañante: es la fuente de la que se derivan los tipos del cliente, y su cumplimiento se comprueba de forma automática.

- Los tipos de TypeScript del cliente se generan a partir de este fichero, de modo que una divergencia entre lo documentado y lo consumido rompe la compilación.
- Los esquemas de validación de cada endpoint y los esquemas de esta especificación se contrastan en las pruebas: si un endpoint acepta un campo que aquí no figura, el test falla.
- Un análisis estático de la especificación se ejecuta en integración continua y bloquea la fusión ante respuestas sin definir, esquemas huérfanos u operaciones sin seguridad declarada.
- Las pruebas de extremo a extremo recorren los flujos del apartado anterior contra la implementación real, no contra simulaciones.
