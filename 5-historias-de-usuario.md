# 3. Historias de usuario

> Documento derivado de `1-descripcion-general-del-producto.md` y `2-arquitectura-del-sistema.md`.
> Primera revisión del backlog — plantilla ampliada respecto a la propuesta inicial (ver nota al final).

---

## Plantilla utilizada

```
## [ID] Story: Como XXX, quiero XXX, para XXX
- Épica:
- Fase: MVP-A | MVP-B | Fase 2
- Prioridad (MoSCoW): Must | Should | Could
- Tamaño estimado: S | M | L

## AC (Given/When/Then)
[happy path + edge cases]

## Definition of Done
[criterios transversales, no repetidos en los AC]

## Contexto técnico (para el agente)
[referencias a doc. 1 y doc. 2]

## Métrica relacionada
[si aplica]

## Notas / riesgos abiertos
[si aplica]
```

---

# Épica 0 — Diseño visual de la plataforma

## HU-00 Story: Como product owner, quiero tener el diseño de alta fidelidad de las pantallas principales en sus 3 breakpoints, para que el desarrollo de MVP-A parta de una base visual validada y no de wireframes de baja fidelidad
- Épica: Diseño
- Fase: Previa a MVP-A (Fase 0 — Discovery, doc. 1 §1.12)
- Prioridad: Must
- Tamaño estimado: M

### AC (Given/When/Then)
- **Happy path:** Given el documento de descripción del producto (doc. 1) traducido a prompts de objetivo/layout/contenido/audiencia, When se generan con el agente de Figma (First Draft) los bocetos de landing, panel "Mis sesiones" y vista de invitado, Then se obtiene un primer boceto de alta fidelidad para cada una en **desktop, tablet y móvil**.
- **Edge case — editor de canvas (patrón no estándar):** Given que el editor de canvas es un patrón de UI poco convencional (lienzo infinito + toolbar flotante) que la generación por prompt no maneja bien, When se diseña esta pantalla, Then se hace a mano, apoyándose en referencias visuales de tldraw, no por generación automática.
- **Edge case — sistema de diseño inconsistente entre bocetos generados:** Given que First Draft no parte de un sistema de diseño propio y cada pantalla puede generarse con estilos ligeramente distintos, When se revisan los bocetos generados, Then se unifican manualmente colores, tipografía y espaciados en un sistema de diseño único antes de darlas por cerradas.
- **Edge case — estados de vista:** Given los 5 estados obligatorios de doc. 2 §2.2.10 (cargando, vacío, error, sin conexión, sin permiso), When se diseña una pantalla que los requiere (p. ej. el panel), Then el diseño incluye al menos los estados de carga y vacío, no solo el estado "feliz" con datos.
- **Edge case — accesibilidad:** Given los requisitos de contraste AA de doc. 2 §2.2.12, When se define la paleta de color, Then se verifica el contraste antes de pasar el diseño a desarrollo, no después.
- **Edge case — áreas táctiles:** Given el requisito de 44×44px mínimo en controles del canvas (doc. 2 §2.2.2), When se diseñan los controles para tablet/móvil, Then se respeta ese mínimo en el propio diseño.
- **Edge case — handoff a desarrollo:** Given el fichero de Figma con las pantallas terminadas, When se activa el Dev Mode MCP Server sobre ese fichero, Then Claude Code y/o GitHub Copilot pueden leer medidas, colores y componentes exactos para implementar, en vez de trabajar a partir de capturas de pantalla.

### Definition of Done
- Diseño en Figma con las 3 variantes de breakpoint por pantalla.
- Sistema de diseño (colores, tipografía, espaciados) unificado manualmente tras la generación con First Draft.
- Dev Mode MCP Server activado en el fichero y probado con al menos una consulta desde Claude Code o Copilot.
- Sustituye formalmente a los wireframes de baja fidelidad de doc. 1 §1.3, que quedan como histórico.

### Contexto técnico (para el agente)
- Doc. 1 §1.1 (objetivo y valor del producto, base para los prompts de First Draft), §1.2 (funcionalidades de MVP-A a diseñar), §1.3 (wireframes de partida, referencia para los prompts).
- Doc. 2 §2.2.2 (breakpoints exactos: <640px móvil, 640–1024px tablet, >1024px desktop; reglas mobile-first; presupuesto de rendimiento de la vista de invitado, que condiciona cuánto "peso visual" puede llevar esa pantalla).
- Herramienta: **Figma**, con **First Draft** para landing/panel/vista de invitado y diseño manual para el editor de canvas; **Dev Mode MCP Server** como puente hacia Claude Code/Copilot.

### Notas / riesgos abiertos
- First Draft genera sobre los componentes propios de Figma, no sobre un sistema de diseño personalizado — el paso de unificación manual (ver AC) es obligatorio, no opcional.
- El Dev Mode MCP Server es relativamente reciente; conviene probar la conexión con Claude Code en un componente sencillo antes de depender de él para todo el handoff.

---

# Épica 1 — Autenticación y cuenta del anfitrión

## HU-01 Story: Como visitante, quiero registrarme con email y contraseña, para poder crear y gestionar mis propios canvases
- Épica: Autenticación
- Fase: MVP-A
- Prioridad: Must
- Tamaño estimado: S

### AC (Given/When/Then)
- **Happy path:** Given que soy un visitante no registrado, When relleno email y contraseña válidos y confirmo, Then se crea mi cuenta y recibo un email de verificación.
- **Edge case — email ya registrado:** Given que el email ya existe, When intento registrarme, Then veo un mensaje de error genérico que no confirma ni desmiente la existencia de la cuenta (evita enumeración).
- **Edge case — contraseña débil:** Given una contraseña de menos de 10 caracteres o presente en listas filtradas, When la envío, Then el registro se rechaza con el motivo explicado.
- **Edge case — canvas antes de verificar:** Given que mi email no está verificado, When intento crear un canvas, Then se me bloquea y se me pide verificar primero.

### Definition of Done
- Textos en ES/EN/DE/NL.
- Cobertura de test para el caso de enumeración de cuentas.
- Verificado en los 3 breakpoints (móvil/tablet/desktop).

### Contexto técnico (para el agente)
- Doc. 1 §1.2 (MVP-A: "Registro/login de anfitriones"), §1.10 (política de contraseñas, verificación de email).
- Doc. 2 §2.5.1 (Supabase Auth, hash gestionado por el proveedor, política de contraseñas ≥10 caracteres, verificación obligatoria antes del primer canvas, mensajes idénticos ante enumeración).

### Métrica relacionada
- Activación (doc. 1 §1.6): % de registrados que crean su primera sesión en <7 días.

---

## HU-02 Story: Como visitante, quiero registrarme/iniciar sesión con Google, para no tener que recordar otra contraseña
- Épica: Autenticación
- Fase: MVP-A
- Prioridad: Should
- Tamaño estimado: S

### AC (Given/When/Then)
- **Happy path:** Given que elijo "Continuar con Google", When autorizo el acceso, Then entro directamente sin pasos adicionales, verificado por defecto.
- **Edge case:** Given que ya tengo una cuenta creada por email con el mismo correo, When inicio con Google, Then las cuentas se vinculan automáticamente en vez de crear un duplicado.

### Definition of Done
- Probado con al menos una cuenta Google real, no solo mock.

### Contexto técnico (para el agente)
- Doc. 1 §1.2, §1.7 (Google OAuth como parte del stack).
- Doc. 2 §2.5.1 (Supabase Auth + Google OAuth).

---

## HU-03 Story: Como anfitrión, quiero recuperar mi contraseña si la olvido, para no perder acceso a mis canvases
- Épica: Autenticación
- Fase: MVP-A
- Prioridad: Must
- Tamaño estimado: S

### AC (Given/When/Then)
- **Happy path:** Given que olvidé mi contraseña, When solicito recuperación con mi email, Then recibo un enlace de un solo uso con caducidad corta.
- **Edge case — email no registrado:** Given un email que no existe, When solicito recuperación, Then el sistema responde con el mismo mensaje y tiempo que si existiera (anti-enumeración).
- **Edge case — enlace caducado:** Given un enlace de recuperación caducado, When lo uso, Then se rechaza con opción de solicitar uno nuevo.

### Definition of Done
- Email de recuperación en los 4 idiomas, según preferencia del usuario.

### Contexto técnico (para el agente)
- Doc. 1 §1.9 (derechos y minimización), §1.10.
- Doc. 2 §2.5.1 (enumeración de cuentas), componente "Emails" = Resend (§2.2.1).

---

# Épica 2 — Gestión del canvas (anfitrión)

## HU-04 Story: Como anfitrión, quiero crear un canvas nuevo con formas, texto e imágenes, para organizar visualmente el contenido de mi sesión
- Épica: Canvas
- Fase: MVP-A
- Prioridad: Must
- Tamaño estimado: M

### AC (Given/When/Then)
- **Happy path:** Given que estoy en mi panel, When pulso "Nueva sesión", Then se abre un canvas vacío donde puedo añadir formas, texto e imágenes.
- **Edge case — imagen no válida:** Given un fichero que no es png/jpeg/webp/svg o supera 10 MB, When intento subirlo, Then se rechaza con el motivo.
- **Edge case — SVG malicioso:** Given un SVG con `<script>` embebido, When se sube, Then el servidor lo sanea o lo rechaza si el saneado lo altera significativamente.

### Definition of Done
- Guardado automático probado tras pérdida de red simulada (ver HU-06).
- Accesible por teclado salvo el propio lienzo (doc. 2 §2.2.12).

### Contexto técnico (para el agente)
- Doc. 1 §1.2 (MVP-A), §1.7 (tldraw como motor de canvas).
- Doc. 2 ADR-01 (adaptador `CanvasEngine`, ningún componente importa el SDK directamente), §2.2.5 (interfaz `CanvasEngineProps`), §2.5.3 (saneado de SVG, lista blanca de tipos, límite de 10 MB).

### Notas / riesgos abiertos
- La licencia *hobby* de tldraw (ADR-02) mantiene visible una marca de agua "made with tldraw" en HTTPS hasta que haya cobro — debe comunicarse en la propia UI de creación, no solo en la documentación.

---

## HU-05 Story: Como anfitrión, quiero que mi trabajo se guarde automáticamente, para no perder cambios si se corta la conexión
- Épica: Canvas
- Fase: MVP-A
- Prioridad: Must
- Tamaño estimado: M

### AC (Given/When/Then)
- **Happy path:** Given que edito el canvas, When dejo de interactuar 2 segundos (o pasan 30 segundos como máximo), Then el documento se guarda vía HTTP sin acción del usuario.
- **Edge case — corte de red:** Given que pierdo conexión mientras edito, When la red vuelve, Then el guardado se reintenta con espera creciente y no se pierde ningún cambio, apoyándose en la copia local de IndexedDB.
- **Edge case — cierre accidental de pestaña:** Given cambios sin confirmar aún en servidor, When cierro la pestaña, Then al reabrir el canvas se recupera desde IndexedDB si el servidor no tiene la versión más reciente.

### Definition of Done
- Test automatizado del escenario de pérdida de red de 30s.

### Contexto técnico (para el agente)
- Doc. 1 §1.2 (guardado CRUD en Supabase, sin sincronización en vivo en MVP-A).
- Doc. 2 §2.2.6 (`local-store`: guardado por HTTP con retardo de 2s, forzado cada 30s, copia en IndexedDB, reintento con espera creciente).

---

# Épica 3 — Panel "Mis sesiones"

## HU-06 Story: Como anfitrión, quiero ver un listado de mis canvases, para retomar rápidamente el que necesito
- Épica: Panel
- Fase: MVP-A
- Prioridad: Must
- Tamaño estimado: S

### AC (Given/When/Then)
- **Happy path:** Given que tengo canvases creados, When entro a "Mis sesiones", Then los veo listados con nombre, fecha de última edición y una miniatura.
- **Edge case — sin canvases:** Given que soy un usuario nuevo sin canvases, When entro al panel, Then veo un estado vacío con una llamada a la acción para crear el primero, no una tabla en blanco.

### Definition of Done
- Estados de carga, vacío y error implementados (los 5 estados de doc. 2 §2.2.10 aplican también aquí, salvo "sin conexión" y "sin permiso" que no tocan a este listado).

### Contexto técnico (para el agente)
- Doc. 1 §1.2, §1.3 (wireframe-2-dashboard.svg).
- Doc. 2 §2.2.10 (los cinco estados de vista obligatorios).

---

## HU-07 Story: Como anfitrión, quiero renombrar o eliminar un canvas, para mantener mi panel ordenado
- Épica: Panel
- Fase: MVP-A
- Prioridad: Should
- Tamaño estimado: S

### AC (Given/When/Then)
- **Happy path:** Given un canvas mío, When pulso "Eliminar" y confirmo, Then el canvas y sus enlaces de invitado dejan de estar accesibles.
- **Edge case — invitado con la sesión abierta al borrar:** Given un invitado viendo el canvas en ese momento, When el anfitrión lo elimina, Then la siguiente petición del invitado (o la sala en MVP-B) falla con un mensaje traducido, no con un error técnico.

### Definition of Done
- El borrado dispara también la purga de ficheros asociados en Storage (no solo el registro en base de datos).

### Contexto técnico (para el agente)
- Doc. 1 §1.9 (retención de datos, borrado de sesiones inactivas).
- Doc. 2 §2.5.2 (matriz de permisos: solo el anfitrión puede renombrar/borrar), §2.5.7 (el borrado alcanza base de datos y ficheros como operación con reintentos).

---

# Épica 4 — Compartir y acceso de invitado

## HU-08 Story: Como anfitrión, quiero generar un enlace para compartir mi canvas, para que otras personas puedan verlo sin registrarse
- Épica: Invitado
- Fase: MVP-A
- Prioridad: Must
- Tamaño estimado: M

### AC (Given/When/Then)
- **Happy path:** Given un canvas guardado, When pulso "Compartir", Then obtengo un enlace único que puedo copiar y enviar por cualquier canal.
- **Edge case — canvas sin guardar aún:** Given un canvas recién creado sin ningún guardado confirmado, When intento compartirlo, Then se me indica que debo guardar al menos una vez (la render que ve el invitado se genera al guardar).

### Definition of Done
- El token es un UUID v4 (122 bits de entropía), nunca un correlativo.

### Contexto técnico (para el agente)
- Doc. 1 §1.2 (link para compartir, control del link), §1.10 (tokens UUID v4 no adivinables).
- Doc. 2 §2.5.2 regla 4 (tokens UUID v4, 122 bits), ADR-09 (la render se genera al guardar, no bajo demanda).

---

## HU-09 Story: Como anfitrión, quiero revocar un enlace de invitado en cualquier momento, para controlar quién sigue teniendo acceso
- Épica: Invitado
- Fase: MVP-A
- Prioridad: Must
- Tamaño estimado: S

### AC (Given/When/Then)
- **Happy path:** Given un enlace activo, When lo revoco desde mi panel, Then cualquier petición posterior con ese token devuelve un 404 genérico, sin importar cuánta caché intermedia exista.
- **Edge case — imagen ya descargada por el invitado:** Given que el invitado ya cargó una imagen del canvas antes de la revocación, When reviso el acceso tras revocar, Then esa imagen concreta puede seguir siendo accesible hasta que caduque su URL firmada (máx. 1 hora) — limitación conocida que debe comunicarse al anfitrión, no un fallo.

### Definition of Done
- Texto de aviso en la UI que explica la ventana de 1 hora para imágenes ya firmadas.

### Contexto técnico (para el agente)
- Doc. 1 §1.2 (control del link), §1.10.
- Doc. 2 ADR-10 (se cachea el contenido, nunca el permiso), §2.4.4 (estrategia de caché, ventana de 1h en URLs firmadas), §2.5.2 regla 6 (revocación combina marca en BD + expiración corta).

---

## HU-10 Story: Como invitado, quiero abrir un enlace y ver el canvas sin necesidad de registrarme, para participar en la sesión con fricción mínima
- Épica: Invitado
- Fase: MVP-A
- Prioridad: Must
- Tamaño estimado: M

### AC (Given/When/Then)
- **Happy path:** Given un enlace válido, When lo abro desde cualquier dispositivo, Then veo el canvas (última versión guardada) con zoom y desplazamiento, sin pedirme login.
- **Edge case — enlace inválido/revocado/caducado:** Given cualquiera de estos casos, When abro el enlace, Then veo el mismo mensaje 404 genérico y en el mismo tiempo de respuesta en todos los casos (para no filtrar cuál era el motivo).
- **Edge case — red móvil lenta:** Given una conexión 4G de gama media, When abro el enlace, Then la vista es interactiva (LCP <2,5s) porque no se carga el motor de canvas completo, solo la render vectorial.

### Definition of Done
- Presupuesto de rendimiento verificado en CI con Lighthouse (LCP <2,5s, <150KB JS) según doc. 2 §2.2.2.
- Página servida con `noindex`.

### Contexto técnico (para el agente)
- Doc. 1 §1.2, §1.3 (wireframe-4-invitado.svg).
- Doc. 2 ADR-09 (render SVG estática, no SDK), §2.2.8 (generación de la render, fallback a PNG si el SVG es muy pesado), §2.5.2 (secuencia completa de validación del token).

### Notas / riesgos abiertos
- **Pendiente de decidir:** el wireframe actual (doc. 1 §1.3) muestra al invitado introduciendo un nombre antes de entrar, pero ADR-09 indica que en MVP-A el invitado solo mira, sin presencia ni identidad — no quedaría claro para qué se le pediría un nombre. Propongo **quitar el paso de "nombre" en MVP-A** y reservarlo para MVP-B (donde sí hay presencia). Confirmar antes de implementar esta historia.
- **Pendiente de decidir (ver análisis previo):** si la render se actualiza sola mientras el invitado tiene la página abierta y el anfitrión sigue guardando cambios, o si requiere refrescar manualmente. Afecta directamente a esta historia y debería resolverse como AC explícito antes de desarrollarla.

### Métrica relacionada
- Métrica interina de MVP-A (doc. 1 §1.6): canvases compartidos y vistos por al menos un invitado / semana.

---

# Épica 5 — Internacionalización

## HU-11 Story: Como usuario (anfitrión o invitado), quiero usar la plataforma en mi idioma, para entender todo sin fricción
- Épica: i18n
- Fase: MVP-A
- Prioridad: Must
- Tamaño estimado: S

### AC (Given/When/Then)
- **Happy path:** Given que mi navegador está en alemán, When entro por primera vez, Then la interfaz se muestra en alemán automáticamente, con opción de cambiarlo manualmente.
- **Edge case — clave de traducción ausente:** Given una clave sin traducir en un idioma, When se renderiza esa pantalla, Then se retrocede al español (idioma por defecto) y se emite un evento de telemetría, en vez de mostrar la clave en bruto.
- **Edge case — textos largos en alemán/neerlandés:** Given que estos idiomas ocupan más espacio que el español, When se renderiza cualquier botón o etiqueta, Then no hay desbordes de layout (verificado con pseudo-locale en CI).

### Definition of Done
- Probado contra la pseudo-locale generada en CI (doc. 2 §2.2.11).

### Contexto técnico (para el agente)
- Doc. 1 §1.11.
- Doc. 2 §2.2.11 (next-intl, formato ICU, prohibido concatenar cadenas, pseudo-locale en CI, fallback ruidoso en desarrollo).

---

# Épica 6 — Exportación

## HU-12 Story: Como anfitrión, quiero exportar mi canvas a PNG o PDF, para usarlo fuera de la plataforma
- Épica: Exportación
- Fase: MVP-A
- Prioridad: Should
- Tamaño estimado: S

### AC (Given/When/Then)
- **Happy path:** Given un canvas con contenido, When pulso "Exportar" y elijo formato, Then se genera el fichero en mi propio navegador y se descarga.
- **Edge case — fase privada (licencia hobby activa):** Given que el proyecto aún no tiene cobro a usuarios, When exporto, Then el fichero incluye la marca de agua "made with tldraw", y la UI me lo advierte antes de exportar, no después.

### Definition of Done
- Ningún contenido del canvas sale del navegador durante la exportación (cero llamadas a un servicio externo de renderizado).

### Contexto técnico (para el agente)
- Doc. 1 §1.2.
- Doc. 2 ADR-08 (exportación 100% en cliente), ADR-02 (marca de agua durante fase privada, consecuencia de producto a comunicar).

---

# Épica 7 — Cumplimiento y cuenta (GDPR)

## HU-13 Story: Como anfitrión, quiero descargar todos mis datos, para ejercer mi derecho de portabilidad
- Épica: GDPR
- Fase: MVP-A
- Prioridad: Must
- Tamaño estimado: M

### AC (Given/When/Then)
- **Happy path:** Given que estoy autenticado, When solicito "Descargar mis datos" desde mi perfil, Then recibo un export con mi perfil, mis canvases y metadatos de mis enlaces.
- **Edge case — export en curso:** Given una solicitud ya en curso, When solicito otra, Then se me informa del estado en lugar de duplicar el proceso.

### Contexto técnico (para el agente)
- Doc. 1 §1.9 (derechos ARCO-POL).
- Doc. 2 §2.5.7 (exportación completa desde el propio perfil).

---

## HU-14 Story: Como anfitrión, quiero eliminar mi cuenta y todos mis datos, para ejercer mi derecho al olvido
- Épica: GDPR
- Fase: MVP-A
- Prioridad: Must
- Tamaño estimado: M

### AC (Given/When/Then)
- **Happy path:** Given que confirmo la eliminación (con doble confirmación), When se procesa, Then se borran mi perfil, mis canvases, mis ficheros en Storage y, si aplica, mis salas de sincronización.
- **Edge case — fallo parcial:** Given que uno de los sistemas (BD, Storage, salas) falla durante el borrado, Then la operación se reintenta hasta completarse en todos los sistemas, no se marca como "hecha" con datos residuales en alguno.

### Definition of Done
- Operación implementada con reintentos, no como llamadas sueltas e independientes.

### Contexto técnico (para el agente)
- Doc. 1 §1.9.
- Doc. 2 §2.5.7 (el borrado alcanza BD, ficheros y salas de sincronización como operación con reintentos).

---

# Épica 8 — Responsive y accesibilidad

## HU-15 Story: Como anfitrión, quiero editar mi canvas desde una tablet con lápiz o el dedo, para trabajar sin necesitar un ordenador
- Épica: Responsive
- Fase: MVP-A
- Prioridad: Should
- Tamaño estimado: M

### AC (Given/When/Then)
- **Happy path:** Given que abro el editor en una tablet (640–1024px), When interactúo con gestos táctiles o lápiz, Then puedo pellizcar para hacer zoom y dibujar con precisión, con la barra de herramientas lateral compacta.
- **Edge case — móvil (<640px):** Given que abro el editor en un móvil, When uso el canvas, Then la barra de herramientas se colapsa en una hoja inferior deslizable en vez de ocupar espacio fijo.
- **Edge case — rotación de pantalla:** Given que giro el dispositivo de vertical a horizontal, When el layout se recalcula, Then el canvas conserva el encuadre y el nivel de zoom que tenía.

### Definition of Done
- Probado con Playwright en los 3 viewports definidos (390×844, 820×1180, 1440×900) con `hasTouch: true`.
- Áreas táctiles ≥44×44px verificadas.
- Probado en al menos un dispositivo físico Android y un iPhone antes de la beta.

### Contexto técnico (para el agente)
- Doc. 1 (requisito transversal responsive en §1.2 y §1.7).
- Doc. 2 §2.2.2 (tabla de breakpoints y estrategia de layout, `touch-action: none` solo en el área del canvas, unidades `dvh`), §2.2.2 tabla de verificación en CI.

---

# Épica 9 — Colaboración en tiempo real (MVP-B)

## HU-16 Story: Como invitado con permiso de edición, quiero modificar el canvas en tiempo real junto al anfitrión, para colaborar de forma efectiva durante la sesión
- Épica: Colaboración
- Fase: MVP-B
- Prioridad: Must (dentro de MVP-B)
- Tamaño estimado: L

### AC (Given/When/Then)
- **Happy path:** Given un enlace con permiso de edición, When entro y modifico una forma, Then el anfitrión y el resto de participantes ven el cambio en tiempo real.
- **Edge case — conflicto simultáneo:** Given que dos participantes mueven la misma forma a la vez, When ambos cambios llegan al servidor, Then ambos convergen al mismo estado final, sin que ninguno de los dos "gane" de forma inconsistente para los demás.
- **Edge case — pérdida de red de 30s:** Given que pierdo conexión brevemente, When se restablece, Then me reconecto automáticamente y mis cambios locales se reaplican sobre el estado del servidor, sin pérdida.
- **Edge case — enlace de solo lectura que intenta editar:** Given un token con rol `view`, When se intenta enviar una escritura, Then se rechaza en el propio servidor de sincronización, no solo ocultando el botón en la interfaz.

### Definition of Done
- Los 7 comportamientos obligatorios de doc. 2 §2.2.7 tienen test automatizado (no solo los cubiertos en los AC de arriba).

### Contexto técnico (para el agente)
- Doc. 1 §1.2 (MVP-B: edición en tiempo real multi-cursor).
- Doc. 2 §2.2.7 (Worker + Durable Object, modelo push/pull/rebase), §2.5.2 (matriz de permisos, rol `edit` vs `view`).

### Métrica relacionada
- North Star (doc. 1 §1.6): sesiones colaborativas activas semanales con ≥2 participantes.

---

## HU-17 Story: Como participante de una sesión, quiero ver quién más está conectado y dónde tiene el cursor, para coordinarme sin pisar el trabajo de otros
- Épica: Colaboración
- Fase: MVP-B
- Prioridad: Should
- Tamaño estimado: M

### AC (Given/When/Then)
- **Happy path:** Given varios participantes conectados, When trabajo en el canvas, Then veo sus cursores y avatares en tiempo real.
- **Edge case — el anfitrión revoca el enlace con el invitado dentro:** Given esta situación, When ocurre, Then el invitado es expulsado de la sala en menos de 60 segundos.

### Contexto técnico (para el agente)
- Doc. 1 §1.2 (indicadores de presencia).
- Doc. 2 §2.2.7 (tabla de comportamientos obligatorios, fila de revocación).

---

## HU-18 Story: Como anfitrión, quiero decidir si mi enlace es de solo lectura o de edición, para controlar el nivel de colaboración según el caso de uso
- Épica: Colaboración
- Fase: MVP-B
- Prioridad: Should
- Tamaño estimado: S

### AC (Given/When/Then)
- **Happy path:** Given que genero un enlace, When elijo el rol "solo lectura" o "edición", Then los invitados con ese enlace reciben exactamente ese permiso, verificado en el servidor y no solo en la interfaz.
- **Edge case — cambio de rol con invitados ya conectados:** Given un enlace de edición ya compartido, When el anfitrión lo cambia a solo lectura, Then los invitados conectados en ese momento pierden la capacidad de editar de inmediato.

### Contexto técnico (para el agente)
- Doc. 1 §1.2 (toggle de permisos por link).
- Doc. 2 §2.5.2 (matriz de permisos efectiva).

---

## Resumen de priorización sugerida para el primer sprint de MVP-A

| Prioridad | Historias |
|---|---|
| Paso 0 — antes de escribir código | HU-00 |
| Imprescindibles para poder demostrar el flujo end-to-end | HU-01, HU-04, HU-05, HU-06, HU-08, HU-10 |
| Siguiente lote (completa el MVP-A funcional) | HU-02, HU-03, HU-07, HU-09, HU-11, HU-13, HU-14 |
| Puede ir en paralelo o quedar para pulir antes de la beta | HU-12, HU-15 |
| MVP-B (no arrancar sin validación previa, ver doc. 1 §1.12) | HU-16, HU-17, HU-18 |

---
*Documento vivo — primera revisión. Dos decisiones quedan pendientes antes de cerrar HU-10 (ver "Notas / riesgos abiertos" en esa historia): el paso de nombre del invitado y la actualización en vivo de la render. HU-00 ya tiene la herramienta de diseño decidida (Figma + Dev Mode MCP Server).*
