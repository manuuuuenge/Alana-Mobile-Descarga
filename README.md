# Alana Mobile — Inventario Electromédico

**Aplicación móvil offline-first para el inventariado de equipamiento electromédico en entornos hospitalarios.**

Diseñada para que un técnico de mantenimiento pueda recorrer un hospital, fotografiar un equipo, recibir una propuesta de datos por IA, completar la ficha y guardar — todo sin depender de tener red estable. La sincronización con la nube ocurre automáticamente en cuanto vuelve la cobertura.

---

## Qué hace

- **Captura guiada del equipo** mediante 3 fotos (placa, equipo y etiqueta de inventario) o, en modo Instalaciones, una sola foto para mobiliario.
- **Reconocimiento por IA** que pre-rellena denominación, marca, modelo, número de serie y código de inventario a partir de las fotos.
- **Validación contextual** por hospital: cada centro tiene sus propias reglas de ubicación, codificación y campos obligatorios, validadas tanto al guardar como al sincronizar.
- **Catálogo precargado** del centro para detectar duplicados en el momento del barrido y evitar repetir trabajo.
- **Funcionamiento 100% offline**: la fuente de verdad operativa es la base de datos local en el dispositivo; la nube es el destino de sincronización, no un requisito de uso.
- **Filosofía núcleo**: jamás se pierde un equipo capturado. Múltiples redes de seguridad (cola de pendientes, cola de errores, recovery LASTRESORT, restauración automática desde snapshot) garantizan que el trabajo del técnico siempre llega a destino.

---

## Centros soportados

La aplicación incluye lógica específica por hospital para respetar las particularidades contractuales y operativas de cada centro:

- **CAUSA** — Complejo Asistencial Universitario de Salamanca (HUSAL, Virgen de la Vega, Béjar, Montalvos, Ciudad Rodrigo)
- **CAULE** — Complejo Asistencial Universitario de León
- **CHUS Santiago** — Complejo Hospitalario Universitario de Santiago de Compostela
- **Gerencia de Atención Primaria de Mallorca**
- **GASPA** — Gerencia de Atención Sanitaria de Palencia
- **Hospital Universitario Nuestra Señora de la Candelaria** (Tenerife)

Cada centro tiene su propia cascada de ubicaciones, validaciones de código de inventario, campos opcionales/obligatorios, y en algunos casos integración con escáner de código de barras nativo.

---

## Características destacadas

### Para el técnico de campo

- **Listado virtualizado** con búsqueda instantánea, filtros combinables (inventariado, fecha, ubicación, PPT, etc.) y persistencia entre sesiones.
- **Modo ráfaga** para procesar lotes de fotos en una pasada.
- **Resumen diario** automático al final de la jornada con métricas de productividad.
- **Pre-llenado predictivo de ubicación** basado en el último equipo guardado.
- **Reconocimiento por voz** en zonas concretas (Mallorca, Inca).
- **Tutorial interactivo** la primera vez que se entra a cada pantalla.

### Para el administrador

- **Personalización local** por dispositivo: campos personalizados, ubicaciones a medida, lista de técnicos por centro — todo sin necesidad de redespliegue.
- **Dashboard de productividad** con métricas por técnico, día y centro.
- **Gamificación opcional** mediante ranking diario anónimo.
- **Importación/exportación** del inventario a Excel.

### Multilingüe

Interfaz completa en **6 idiomas**:

- Español (castellano)
- Euskera
- Gallego
- Catalán
- Valenciano
- Portugués

---

## Arquitectura técnica (resumen)

| Capa | Tecnología |
|------|------------|
| UI | React 18 + TypeScript 5 + Tailwind 3 + shadcn/ui (Radix) |
| Build | Vite 5 con SWC, code-splitting agresivo y chunks lazy por centro |
| Móvil | Capacitor 7 (Android e iOS) |
| Almacenamiento local | SQLite nativo (`@capacitor-community/sqlite`) + fallback `localStorage` en web |
| Backend | Firebase 12 (Firestore + Auth) con reglas multi-tenant por centro |
| Estado | TanStack Query + Context API |
| Captura | Cámara nativa + ML Kit barcode scanning + Quagga2 + jsQR |
| Internacionalización | Sistema propio sobre LanguageContext |

### Patrones clave

- **Offline-first verdadero**: cada operación se persiste localmente y se sincroniza después. La UI nunca espera a la red.
- **Defense in depth para la sincronización**: probe de identidad ante fallos de red, deduplicación de pendientes, timeout en consultas, recovery LASTRESORT y restauración automática de backup si una tabla queda vacía teniendo snapshot con datos.
- **Detección de red multicapa**: combina el plugin de Capacitor, `navigator.onLine` y un health-check periódico a un endpoint externo, con clasificación conservadora (`offline`/`poor`/`good`) que requiere 2 fallos consecutivos para degradar.
- **Endurecimiento Android**: HTTPS only, sin backup cloud, sin transferencia entre dispositivos, splash configurable, R8 + minify + shrinker en producción.

---

## Plataformas

- **Android** (principal) — minSdk 24 (Android 7.0), targetSdk 35
- **iOS** — soportado vía Capacitor (compilación bajo demanda)
- **Web** — compilable como PWA standalone

---

## Privacidad y seguridad

- **Reglas Firestore multi-tenant** versionadas con default-deny global: cada técnico solo puede leer/escribir documentos de los centros a los que pertenece.
- **Sin tráfico en claro**: `usesCleartextTraffic` desactivado en Android.
- **Sin backup cloud ni transferencia entre dispositivos**: las credenciales cacheadas localmente no salen del dispositivo del técnico.
- **Logs PII gateados**: cualquier log que pueda identificar a un usuario o equipo está restringido a builds de desarrollo.
- **Personalización siempre local**: lo que el admin configura en su dispositivo no se sube a Firestore — protege la información organizativa interna de cada centro.

---

## Releases

Las notas de versión visibles para el técnico se mantienen en código (`src/constants/releaseNotes.ts`) para que sean accesibles offline desde la propia app, en la pantalla "Últimas Mejoras".

La versión actual incluye, entre otras cosas:

- Marca visual "PPT" + filtro "Solo PPT" en CAUSA para acotar el listado al alcance contractual del Pliego de Prescripciones Técnicas.
- Modo CAUSA-REBARRIDO: posibilidad de re-inventariar equipos ya hechos para enriquecer datos.
- Internacionalización completa de los flujos críticos.
- Optimizaciones de RAM/GPU para gama baja (Oppo A5 Pro, Realme entry, Xiaomi gama media, Samsung A20–A50).

---

## Equipo

Desarrollado por **AINATEC** para hospitales y servicios públicos de salud que necesitan inventariar su parque de equipamiento electromédico de forma rigurosa, multi-centro y resistente a entornos con red intermitente.

---

## Licencia

Software propietario. Todos los derechos reservados.
