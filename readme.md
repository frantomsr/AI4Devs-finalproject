## Índice

0. [Ficha del proyecto](#0-ficha-del-proyecto)
1. [Descripción general del producto](#1-descripción-general-del-producto)
2. [Arquitectura del sistema](#2-arquitectura-del-sistema)
3. [Modelo de datos](#3-modelo-de-datos)
4. [Especificación de la API](#4-especificación-de-la-api)
5. [Historias de usuario](#5-historias-de-usuario)
6. [Tickets de trabajo](#6-tickets-de-trabajo)
7. [Pull requests](#7-pull-requests)

---

## 0. Ficha del proyecto
Francisco Tomás Santos Rebollo

### **0.2. Nombre del proyecto:**
HoldingField
### **0.3. Descripción breve del proyecto:**
Este proyecto consiste en una plataforma web que permitira compartir tableros tipo canva para realizar sesiones de coaching. El resultado final permitira a coaches compartir sesiones de trabajo con sus coachees con diferentes plantillas y elementos. ademas se propondran diferentes modelos de subscripcion y tipos de usuarios.

Fase Inicial del proyecto: en esta primera fase del proyecto se creara el registro del coach en base de datos y un tablero y un tipo de elemento para realizar la sesión. La apliación no será todavia compartida.
### **0.4. URL del proyecto:**
https://github.com/frantomsr/AI4Devs-finalproject.git

### **0.5. URL o archivo comprimido del repositorio**
https://github.com/frantomsr/AI4Devs-finalproject.git


---

## 1. Descripción general del producto

> Describe en detalle los siguientes aspectos del producto:

### **1.1. Objetivo:**

> Propósito del producto. Qué valor aporta, qué soluciona, y para quién.

### **1.2. Características y funcionalidades principales:**

> Enumera y describe las características y funcionalidades específicas que tiene el producto para satisfacer las necesidades identificadas.

### **1.3. Diseño y experiencia de usuario:**

> Proporciona imágenes y/o videotutorial mostrando la experiencia del usuario desde que aterriza en la aplicación, pasando por todas las funcionalidades principales.

### **1.4. Instrucciones de instalación:**
> Documenta de manera precisa las instrucciones para instalar y poner en marcha el proyecto en local (librerías, backend, frontend, servidor, base de datos, migraciones y semillas de datos, etc.)

---

## 2. Arquitectura del Sistema

### **2.1. Diagrama de arquitectura:**
El sistema combina tres patrones: monolito modular sobre Jamstack (Next.js en Vercel), Backend for Frontend para aislar al invitado de Supabase, y puertos y adaptadores aplicados solo al motor de canvas y a la persistencia. Se documentan vistas de contexto y de contenedores para MVP-A (canvas individual, invitado recibe una render estática) y MVP-B (colaboración en tiempo real vía Cloudflare Workers + Durable Objects). Se justifican los beneficios (coste bajo, superficie de ataque pequeña, reversibilidad) y los sacrificios asumidos (dependencia de Supabase, RLS saltada en el flujo de invitado, sin fusión offline en MVP-B).

**Documentación completa:** [2-arquitectura-del-sistema.md](2-arquitectura-del-sistema.md)

### **2.2. Descripción de componentes principales:**

Enumera los componentes técnicos del sistema: aplicación Next.js/React, visor del invitado sin SDK de canvas, adaptadores `CanvasEngine` y `CanvasStore`, middleware de borde, BFF con Route Handlers, identidad dual (Supabase Auth para el anfitrión, JWT propio para el invitado), Postgres con RLS, Storage y el servidor de sincronización en Cloudflare (MVP-B). Detalla también la estrategia responsive mobile-first, la internacionalización con `next-intl` y la generación de la render que se sirve al invitado.

**Documentación completa:** [2-arquitectura-del-sistema.md](2-arquitectura-del-sistema.md)

### **2.3. Descripción de alto nivel del proyecto y estructura de ficheros**

Monorepo único que combina la aplicación web (App Router de Next.js) y el Worker de sincronización. La carpeta `lib/canvas/` concentra el núcleo hexagonal (interfaces de motor y persistencia), `lib/auth/` separa las dos identidades del sistema y `lib/db/` aísla la credencial privilegiada usada por el invitado. Tres reglas de linter, verificadas en CI, impiden que el SDK del motor, la credencial privilegiada o el acceso directo a datos se importen fuera de sus módulos designados.

**Documentación completa:** [2-arquitectura-del-sistema.md](2-arquitectura-del-sistema.md)

### **2.4. Infraestructura y despliegue**

Infraestructura en Vercel (funciones en Frankfurt), Supabase (Frankfurt) y, en MVP-B, Cloudflare con jurisdicción UE, manteniendo todo el tratamiento de datos dentro del EEE. El despliegue sigue un pipeline con CI bloqueante (lint, tests unitarios, autorización, e2e responsive, presupuesto de rendimiento, i18n, auditoría de seguridad) y un gate manual de checklist OWASP/GDPR antes de producción. La caché se rige por la regla de cachear contenido, nunca permiso, para no comprometer la revocación inmediata de enlaces.

**Documentación completa:** [2-arquitectura-del-sistema.md](2-arquitectura-del-sistema.md)

### **2.5. Seguridad**

Dos modelos de amenaza separados: el anfitrión (autenticado, protegido por Row Level Security en Postgres) y el invitado (acceso público mediante un secreto de 122 bits en la URL, sin dato persistente). La autorización, validación de entrada, cabeceras de seguridad y limitación de tasa en dos capas (IP en el borde, sesión en Postgres) cubren el Top 10 de OWASP. Se documentan además la gestión de secretos, la privacidad del invitado y el procedimiento ante incidentes.

**Documentación completa:** [2-arquitectura-del-sistema.md](2-arquitectura-del-sistema.md)

### **2.6. Tests**

La integración continua ejecuta jobs bloqueantes: `test:unit` (cuotas, firma de tokens, saneado), `test:authz` (aislamiento entre usuarios y alcance del token de invitado), `test:e2e` y `test:e2e:responsive` (flujos completos en móvil, tablet y desktop con eventos táctiles reales), `perf:budget` (presupuesto de rendimiento de la vista de invitado) e `i18n:check` (paridad de idiomas). Ningún job es opcional y todos fallan el build ante una regresión.

**Documentación completa:** [2-arquitectura-del-sistema.md](2-arquitectura-del-sistema.md)

---

## 3. Modelo de Datos

### **3.1. Diagrama del modelo de datos:**

Diagrama entidad-relación en mermaid sobre PostgreSQL (Supabase, Frankfurt) con RLS activa en todas las tablas. El modelo se rige por seis principios: no se almacena ningún dato del invitado, la propiedad de un canvas es inmutable, el snapshot editable vive separado de los metadatos de lectura frecuente, las cuotas son datos y no código, toda tabla con datos personales declara su retención, y las claves primarias son UUID opacos.

**Documentación completa:** [3-modelo-de-datos.md](3-modelo-de-datos.md)

### **3.2. Descripción de entidades principales:**

Diez entidades: `plan_limits` (cuotas por plan), `profiles` (anfitrión registrado), `usage_counters` (consumo mensual), `canvases` y `canvas_documents` (metadatos y snapshot, separados por patrón de escritura), `share_links` (enlace de invitado con token UUID), `canvas_assets` (imágenes), `canvas_views` (visitas seudonimizadas), `security_events` y `rate_limit_counters`. Cada una detalla tipo, restricciones (`NOT NULL`, `CHECK`, `UNIQUE`), claves foráneas y su comportamiento en cascada o anulación.

**Documentación completa:** [3-modelo-de-datos.md](3-modelo-de-datos.md)

---

## 4. Especificación de la API

API interna con patrón Backend for Frontend (Route Handlers de Next.js bajo `/api`), sin garantía de compatibilidad externa y con doble esquema de identidad: cookie de sesión del anfitrión y cookie de sesión efímera del invitado, ambas `HttpOnly`. Especificada en OpenAPI 3.1, cubre endpoints de cuenta, canvases, documento, compartición, invitado, ficheros y tiempo real, con formato de error uniforme, rate limiting y paginación por cursor. Autenticación, subida de ficheros y sincronización en tiempo real quedan deliberadamente fuera de esta API y se resuelven por SDK de Supabase, URL firmada y WebSocket respectivamente.

**Documentación completa:** [4-especificaciones-de-la-api.md](4-especificaciones-de-la-api.md)

---

## 5. Historias de Usuario

> Documenta 3 de las historias de usuario principales utilizadas durante el desarrollo, teniendo en cuenta las buenas prácticas de producto al respecto.

**Historia de Usuario 1**

**Historia de Usuario 2**

**Historia de Usuario 3**

---

## 6. Tickets de Trabajo

> Documenta 3 de los tickets de trabajo principales del desarrollo, uno de backend, uno de frontend, y uno de bases de datos. Da todo el detalle requerido para desarrollar la tarea de inicio a fin teniendo en cuenta las buenas prácticas al respecto. 

**Ticket 1**

**Ticket 2**

**Ticket 3**

---

## 7. Pull Requests

> Documenta 3 de las Pull Requests realizadas durante la ejecución del proyecto

**Pull Request 1**

**Pull Request 2**

**Pull Request 3**

