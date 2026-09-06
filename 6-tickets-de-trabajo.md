# 4. Tareas técnicas — HU-00, HU-01, HU-04

> Documento derivado de `3-historias-de-usuario.md`. Primer lote de desglose en tareas.

## Plantilla utilizada

```
## [ID-Tarea] Tarea: [Título corto, en infinitivo]
- Historia asociada: HU-XX
- Tipo: Frontend | Backend | Diseño | Infra/DevOps | QA | Documentación
- Estimación (story points): 1 | 2 | 3 | 5 | 8
- Prioridad: Must | Should | Could
- Estado: Por hacer | En progreso | Hecho
- Dependencias: [otras tareas, si las hay]

### Descripción
[qué hay que hacer, en un par de frases]

### Checklist de finalización
- [ ] ...

### Referencias técnicas
[sección de doc. 1 / doc. 2 / ADR / historia]

### Notas
[si aplica]
```

---

# Tareas de HU-00 — Diseño visual de la plataforma

## T-00-1 Tarea: Redactar los prompts de First Draft
- Historia asociada: HU-00
- Tipo: Diseño
- Estimación: 2
- Prioridad: Must
- Estado: Por hacer
- Dependencias: —

### Descripción
Traducir el doc. 1 (§1.1, §1.2) en un prompt de objetivo/layout/contenido/audiencia para cada una de las 3 pantallas que sí se generan por IA: landing, panel "Mis sesiones" y vista de invitado.

### Checklist de finalización
- [ ] Prompt de landing redactado y revisado
- [ ] Prompt de panel redactado y revisado
- [ ] Prompt de vista de invitado redactado y revisado

### Referencias técnicas
- HU-00, doc. 1 §1.1, §1.2, §1.3.

---

## T-00-2 Tarea: Generar bocetos con Figma First Draft (desktop)
- Historia asociada: HU-00
- Tipo: Diseño
- Estimación: 3
- Prioridad: Must
- Estado: Por hacer
- Dependencias: T-00-1

### Descripción
Ejecutar First Draft con los 3 prompts para obtener un primer boceto en desktop de landing, panel y vista de invitado.

### Checklist de finalización
- [ ] 3 bocetos generados en desktop
- [ ] Revisión rápida de que cada boceto cubre el contenido mínimo de doc. 1 §1.2

### Referencias técnicas
- HU-00.

---

## T-00-3 Tarea: Adaptar los bocetos a tablet y móvil
- Historia asociada: HU-00
- Tipo: Diseño
- Estimación: 3
- Prioridad: Must
- Estado: Por hacer
- Dependencias: T-00-2

### Descripción
Crear las variantes de tablet (640–1024px) y móvil (<640px) de las 3 pantallas generadas, siguiendo la estrategia de layout de doc. 2 §2.2.2 (barra inferior deslizable en móvil, barra lateral compacta en tablet).

### Checklist de finalización
- [ ] Variante tablet de las 3 pantallas
- [ ] Variante móvil de las 3 pantallas
- [ ] Verificado contra la tabla de breakpoints de doc. 2 §2.2.2

### Referencias técnicas
- Doc. 2 §2.2.2 (tabla de breakpoints y estrategia de layout).

---

## T-00-4 Tarea: Diseñar a mano el editor de canvas
- Historia asociada: HU-00
- Tipo: Diseño
- Estimación: 5
- Prioridad: Must
- Estado: Por hacer
- Dependencias: —

### Descripción
Diseñar la pantalla del editor (lienzo + toolbar + panel de participantes) en los 3 breakpoints, sin generación por IA, apoyándose en referencias visuales de tldraw.

### Checklist de finalización
- [ ] Layout desktop (herramientas izquierda, participantes arriba derecha, canvas centro)
- [ ] Layout tablet (barra lateral compacta)
- [ ] Layout móvil (barra inferior deslizable, panel de participantes como sobreimpresión)

### Referencias técnicas
- HU-00 (edge case "editor de canvas"), doc. 1 §1.3 (wireframe-3-canvas.svg como punto de partida), doc. 2 §2.2.2.

---

## T-00-5 Tarea: Unificar el sistema de diseño
- Historia asociada: HU-00
- Tipo: Diseño
- Estimación: 3
- Prioridad: Must
- Estado: Por hacer
- Dependencias: T-00-3, T-00-4

### Descripción
Revisar las 4 pantallas ya generadas/diseñadas y unificar manualmente colores, tipografía y espaciados en un único sistema de diseño reutilizable.

### Checklist de finalización
- [ ] Paleta de color única definida
- [ ] Escala tipográfica única definida
- [ ] Espaciados consistentes entre las 4 pantallas

### Referencias técnicas
- HU-00 (edge case "sistema de diseño inconsistente").

---

## T-00-6 Tarea: Verificar contraste AA y áreas táctiles
- Historia asociada: HU-00
- Tipo: QA
- Estimación: 2
- Prioridad: Must
- Estado: Por hacer
- Dependencias: T-00-5

### Descripción
Comprobar que la paleta final cumple contraste AA y que todos los controles táctiles de tablet/móvil miden al menos 44×44px.

### Checklist de finalización
- [ ] Contraste AA verificado en textos y controles
- [ ] Áreas táctiles ≥44×44px verificadas en tablet y móvil

### Referencias técnicas
- Doc. 2 §2.2.12 (contraste AA), §2.2.2 (áreas táctiles).

---

## T-00-7 Tarea: Diseñar estados de carga y vacío del panel
- Historia asociada: HU-00
- Tipo: Diseño
- Estimación: 2
- Prioridad: Should
- Estado: Por hacer
- Dependencias: T-00-2

### Descripción
Añadir al diseño del panel "Mis sesiones" los estados de carga (esqueleto) y vacío (usuario sin canvases), no solo el estado con datos.

### Checklist de finalización
- [ ] Estado de carga diseñado
- [ ] Estado vacío diseñado con llamada a la acción

### Referencias técnicas
- Doc. 2 §2.2.10 (los 5 estados obligatorios).

---

## T-00-8 Tarea: Activar el Dev Mode MCP Server en el fichero de Figma
- Historia asociada: HU-00
- Tipo: Infra/DevOps
- Estimación: 1
- Prioridad: Must
- Estado: Por hacer
- Dependencias: T-00-5

### Descripción
Habilitar el servidor MCP de Dev Mode sobre el fichero de Figma ya unificado, para exponer medidas, colores y componentes a agentes externos.

### Checklist de finalización
- [ ] MCP Server activado en el fichero
- [ ] Acceso probado desde un cliente MCP genérico

### Referencias técnicas
- HU-00 (edge case "handoff a desarrollo").

---

## T-00-9 Tarea: Conectar Claude Code/Copilot al MCP Server y validar lectura
- Historia asociada: HU-00
- Tipo: Infra/DevOps
- Estimación: 2
- Prioridad: Must
- Estado: Por hacer
- Dependencias: T-00-8

### Descripción
Configurar la conexión MCP en el entorno de desarrollo y comprobar que Claude Code (y/o Copilot) puede leer las especificaciones de al menos un componente sencillo del diseño.

### Checklist de finalización
- [ ] Conexión MCP configurada
- [ ] Lectura de specs validada sobre un componente de prueba

### Referencias técnicas
- HU-00 (Definition of Done y notas sobre madurez del MCP Server).

---

# Tareas de HU-01 — Registro con email y contraseña

## T-01-1 Tarea: Implementar el formulario de registro
- Historia asociada: HU-01
- Tipo: Frontend
- Estimación: 3
- Prioridad: Must
- Estado: Por hacer
- Dependencias: T-00-5 (diseño de la pantalla de registro)

### Descripción
Construir el formulario de registro (email + contraseña) siguiendo el diseño de Figma, con validación en cliente.

### Checklist de finalización
- [ ] Formulario maquetado según el diseño
- [ ] Validación en cliente (formato de email, contraseña mínima)

### Referencias técnicas
- Doc. 1 §1.2, HU-01.

---

## T-01-2 Tarea: Configurar Supabase Auth (email/password) en staging
- Historia asociada: HU-01
- Tipo: Infra/DevOps
- Estimación: 2
- Prioridad: Must
- Estado: Por hacer
- Dependencias: —

### Descripción
Activar el proveedor de email/contraseña en el proyecto Supabase de staging y conectar las variables de entorno (doc. 1 §1.4).

### Checklist de finalización
- [ ] Proveedor email/password activado en Supabase
- [ ] Variables de entorno configuradas en `.env.local`

### Referencias técnicas
- Doc. 1 §1.4, §1.7. Doc. 2 §2.5.1.

---

## T-01-3 Tarea: Implementar la política de contraseñas
- Historia asociada: HU-01
- Tipo: Backend
- Estimación: 3
- Prioridad: Must
- Estado: Por hacer
- Dependencias: T-01-2

### Descripción
Forzar mínimo de 10 caracteres y comprobación contra listas de contraseñas filtradas en el flujo de registro.

### Checklist de finalización
- [ ] Validación de longitud mínima en servidor
- [ ] Comprobación contra lista de contraseñas filtradas integrada

### Referencias técnicas
- Doc. 2 §2.5.1 ("Política de contraseñas").

---

## T-01-4 Tarea: Implementar mensajes anti-enumeración
- Historia asociada: HU-01
- Tipo: Backend
- Estimación: 2
- Prioridad: Must
- Estado: Por hacer
- Dependencias: T-01-2

### Descripción
Asegurar que el mensaje de error ante un email ya registrado es idéntico (texto y tiempo de respuesta) al de cualquier otro fallo de validación.

### Checklist de finalización
- [ ] Mensaje unificado implementado
- [ ] Tiempo de respuesta equiparado entre casos

### Referencias técnicas
- HU-01 (edge case "email ya registrado"), doc. 2 §2.5.1 ("Enumeración de cuentas").

---

## T-01-5 Tarea: Implementar verificación de email
- Historia asociada: HU-01
- Tipo: Backend
- Estimación: 3
- Prioridad: Must
- Estado: Por hacer
- Dependencias: T-01-2

### Descripción
Enviar email de verificación al registrarse (vía Resend) y marcar la cuenta como verificada al confirmar el enlace.

### Checklist de finalización
- [ ] Envío de email de verificación funcionando
- [ ] Estado "verificado" persistido correctamente

### Referencias técnicas
- Doc. 1 §1.7 (Resend), doc. 2 §2.2.1 (componente "Emails").

---

## T-01-6 Tarea: Bloquear creación de canvas sin email verificado
- Historia asociada: HU-01
- Tipo: Backend
- Estimación: 2
- Prioridad: Must
- Estado: Por hacer
- Dependencias: T-01-5

### Descripción
Añadir la comprobación de email verificado como requisito previo en el endpoint de creación de canvas.

### Checklist de finalización
- [ ] Comprobación añadida en el BFF
- [ ] Mensaje explicativo mostrado en el frontend si no está verificado

### Referencias técnicas
- HU-01 (edge case "canvas antes de verificar"), doc. 2 §2.5.1.

---

## T-01-7 Tarea: Traducir el formulario a los 4 idiomas
- Historia asociada: HU-01
- Tipo: Documentación / Frontend
- Estimación: 2
- Prioridad: Must
- Estado: Por hacer
- Dependencias: T-01-1

### Descripción
Añadir las cadenas del formulario de registro (labels, errores, email de verificación) a los catálogos ES/EN/DE/NL.

### Checklist de finalización
- [ ] Catálogos actualizados en `messages/{es,en,de,nl}.json`
- [ ] Revisado contra pseudo-locale (doc. 2 §2.2.11)

### Referencias técnicas
- Doc. 1 §1.11, doc. 2 §2.2.11.

---

## T-01-8 Tarea: Tests del flujo de registro
- Historia asociada: HU-01
- Tipo: QA
- Estimación: 3
- Prioridad: Must
- Estado: Por hacer
- Dependencias: T-01-3, T-01-4, T-01-6

### Descripción
Cubrir con tests automatizados: registro feliz, email duplicado, contraseña débil, y bloqueo de creación de canvas sin verificar.

### Checklist de finalización
- [ ] Test de registro feliz
- [ ] Test de email duplicado (verifica mensaje anti-enumeración)
- [ ] Test de contraseña débil
- [ ] Test de bloqueo sin verificación

### Referencias técnicas
- HU-01 (todos los AC).

---

## T-01-9 Tarea: Verificar el formulario en los 3 breakpoints
- Historia asociada: HU-01
- Tipo: QA
- Estimación: 2
- Prioridad: Should
- Estado: Por hacer
- Dependencias: T-01-1

### Descripción
Probar el formulario de registro en los 3 viewports definidos (390×844, 820×1180, 1440×900) con Playwright.

### Checklist de finalización
- [ ] Probado en móvil
- [ ] Probado en tablet
- [ ] Probado en desktop

### Referencias técnicas
- Doc. 2 §2.2.2 (tabla de verificación en CI).

---

# Tareas de HU-04 — Crear canvas con formas, texto e imágenes

## T-04-1 Tarea: Implementar el adaptador `CanvasEngine`
- Historia asociada: HU-04
- Tipo: Frontend
- Estimación: 5
- Prioridad: Must
- Estado: Por hacer
- Dependencias: —

### Descripción
Crear la interfaz `CanvasEngine`/`CanvasEngineProps` (ADR-01) y su implementación concreta con tldraw, de forma que ningún otro componente importe el SDK directamente.

### Checklist de finalización
- [ ] Interfaz definida en `lib/canvas/engine.ts`
- [ ] Implementación tldraw aislada en `lib/canvas/tldraw/`
- [ ] Regla de linter que bloquea imports directos del SDK fuera de esa carpeta

### Referencias técnicas
- Doc. 2 ADR-01, §2.2.5, §2.3 (reglas estructurales del linter).

---

## T-04-2 Tarea: Integrar tldraw en modo local en el editor
- Historia asociada: HU-04
- Tipo: Frontend
- Estimación: 5
- Prioridad: Must
- Estado: Por hacer
- Dependencias: T-04-1, T-00-4 (diseño del editor)

### Descripción
Montar el editor de canvas en la ruta `(app)/canvas/[id]/` usando el adaptador `CanvasEngine`, sin sincronización en vivo (MVP-A).

### Checklist de finalización
- [ ] Editor renderiza formas, texto e imágenes
- [ ] Carga diferida del motor (solo en esta pantalla, doc. 2 §2.2.2)

### Referencias técnicas
- Doc. 1 §1.2, §1.7. Doc. 2 §2.2.1, §2.2.2 ("carga diferida del motor de canvas").

---

## T-04-3 Tarea: Implementar subida de imágenes con validación
- Historia asociada: HU-04
- Tipo: Backend / Frontend
- Estimación: 3
- Prioridad: Must
- Estado: Por hacer
- Dependencias: T-04-2

### Descripción
Permitir subir imágenes al canvas validando tipo real por contenido (no por extensión), lista blanca (png/jpeg/webp/svg) y tamaño máximo de 10 MB, con subida directa a Storage vía URL firmada.

### Checklist de finalización
- [ ] Validación de tipo real implementada
- [ ] Límite de 10 MB aplicado
- [ ] Subida directa a Storage con URL firmada (sin pasar por función serverless)

### Referencias técnicas
- HU-04 (edge case "imagen no válida"), doc. 2 §2.1.3 (subida directa a Storage), §2.5.3.

---

## T-04-4 Tarea: Implementar saneado de SVG en servidor
- Historia asociada: HU-04
- Tipo: Backend
- Estimación: 3
- Prioridad: Must
- Estado: Por hacer
- Dependencias: T-04-3

### Descripción
Sanear cualquier SVG subido eliminando `script`, `foreignObject`, manejadores de eventos y referencias externas; rechazar el fichero si el saneado lo altera significativamente.

### Checklist de finalización
- [ ] Saneado implementado y probado con un SVG malicioso de ejemplo
- [ ] Rechazo correcto cuando el saneado altera el fichero de forma significativa

### Referencias técnicas
- HU-04 (edge case "SVG malicioso"), doc. 2 §2.5.3.

---

## T-04-5 Tarea: Solicitar la licencia hobby de tldraw
- Historia asociada: HU-04
- Tipo: Infra/DevOps
- Estimación: 1
- Prioridad: Must
- Estado: Por hacer
- Dependencias: —

### Descripción
Solicitar al equipo de tldraw la licencia hobby para poder usar el SDK en el entorno de staging/producción en HTTPS, lo antes posible dado que la concesión es discrecional.

### Checklist de finalización
- [ ] Solicitud enviada
- [ ] Clave recibida y configurada en staging

### Referencias técnicas
- Doc. 2 ADR-02, §2.4.2 (licencia por entorno).

---

## T-04-6 Tarea: Añadir aviso de marca de agua en la UI
- Historia asociada: HU-04
- Tipo: Frontend
- Estimación: 1
- Prioridad: Should
- Estado: Por hacer
- Dependencias: T-04-5

### Descripción
Mostrar un aviso visible en la pantalla de creación/exportación indicando que, durante la fase privada, el canvas y las exportaciones llevan la marca "made with tldraw".

### Checklist de finalización
- [ ] Aviso visible antes de exportar
- [ ] Texto traducido a los 4 idiomas

### Referencias técnicas
- HU-04 (notas / riesgos abiertos), doc. 2 ADR-02.

---

## T-04-7 Tarea: Verificar accesibilidad por teclado
- Historia asociada: HU-04
- Tipo: QA
- Estimación: 2
- Prioridad: Should
- Estado: Por hacer
- Dependencias: T-04-2

### Descripción
Comprobar que toda la interfaz del editor es navegable por teclado salvo el propio lienzo, según doc. 2 §2.2.12.

### Checklist de finalización
- [ ] Navegación por teclado verificada en toolbar y menús
- [ ] Foco visible en todos los controles

### Referencias técnicas
- Doc. 2 §2.2.12.

---

## T-04-8 Tarea: Tests de creación de canvas y ficheros inválidos
- Historia asociada: HU-04
- Tipo: QA
- Estimación: 3
- Prioridad: Must
- Estado: Por hacer
- Dependencias: T-04-3, T-04-4

### Descripción
Cubrir con tests automatizados: creación feliz de canvas, rechazo de ficheros con tipo/tamaño inválido, y rechazo/saneado de SVG malicioso.

### Checklist de finalización
- [ ] Test de creación feliz
- [ ] Test de fichero inválido (tipo/tamaño)
- [ ] Test de SVG malicioso

### Referencias técnicas
- HU-04 (todos los AC).

---

## Resumen de estimación total (story points)

| Historia | Nº de tareas | Story points totales |
|---|---|---|
| HU-00 | 9 | 23 |
| HU-01 | 9 | 20 |
| HU-04 | 8 | 23 |
| **Total** | **26** | **66** |

---
*Documento vivo — primer lote de tareas (HU-00, HU-01, HU-04). El resto de historias se desglosan en los siguientes lotes.*
