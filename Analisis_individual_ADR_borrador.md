# Análisis individual – Requerimientos y ADR
**Examen I – Desarrollo de Aplicaciones Móviles · Control Electoral para Organización Política**

| Dato | Detalle |
|---|---|
| Integrante 1 | Oscar Julian Vásquez Borja |

> **Cómo leer este documento:** los RF, RNF y supuestos describen **qué** necesita la organización y no mencionan tecnologías. Las tecnologías aparecen **solo en las ADR**: son mis propuestas para la reunión de consenso.
> Las secciones del enunciado se citan por su título (el documento no tiene numeración): *Acceso*, *Territorio* (Organización del territorio y del personal), *Registro* (Registro de los resultados), *Confidencialidad*, *Seguimiento*, *Conectividad*, *Estabilidad*, *Restricciones*.

---

## 1. Requerimientos identificados

### Requerimientos funcionales

| Código | Requerimiento | Necesidad que lo origina |
|---|---|---|
| RF-01 | No existe auto‑registro. El coordinador provincial crea las cuentas de los coordinadores de recinto, y cada coordinador de recinto crea las de sus veedores. Sin cuenta habilitada no se puede entrar. | Acceso |
| RF-02 | Inicio de sesión con credenciales (correo + clave). | Acceso |
| RF-03 | En el primer ingreso, el usuario debe cambiar obligatoriamente la clave temporal antes de usar la app. | Acceso ("solo el titular conozca su clave") |
| RF-04 | Se registran cédula, nombres, apellidos, teléfono y correo, todos obligatorios. La cédula se valida con el algoritmo oficial ecuatoriano (módulo 10) y no puede repetirse. | Acceso (cédulas mal digitadas) |
| RF-05 | Verificación del correo: la cuenta solo queda operativa cuando el titular confirma que el correo es suyo. | Acceso ("certeza de que es real y le pertenece") |
| RF-06 | Recuperación de clave por cuenta propia, a través del correo verificado. | Acceso (llamadas al coordinador provincial) |
| RF-07 | El coordinador provincial consulta los recintos de su jurisdicción y puede agregar nuevos (cantón, parroquia, nombre, número de JRV). | Territorio |
| RF-08 | El coordinador provincial asigna **un único** coordinador a cada recinto. | Territorio |
| RF-09 | El coordinador de recinto asigna un veedor a cada mesa. Un veedor puede cubrir varias mesas sin límite. | Territorio |
| RF-10 | El coordinador de recinto puede reemplazar a un veedor ausente en el momento, incluso por una persona nueva que registra en ese instante. | Territorio |
| RF-11 | Por cada mesa, el veedor registra **dos actas** (alcaldía y prefectura) con los votos de las 5 organizaciones, los nulos, los blancos y el total de sufragantes. | Registro |
| RF-12 | Validación inmediata al digitar: enteros ≥ 0, ninguna organización supera el total, y votos válidos + nulos + blancos = total de sufragantes. | Registro ("números imposibles") |
| RF-13 | Foto obligatoria de cada acta, con vista previa y confirmación de que es legible antes de guardarla. | Registro (fotos borrosas) |
| RF-14 | Ubicación GPS y fecha/hora obligatorias al registrar el acta. Se pueden consultar (en un mapa o con coordenadas) al revisarla. | Registro y Seguimiento |
| RF-15 | El veedor corrige sus propias actas y el coordinador de recinto corrige cualquier acta de su recinto. Cada cambio queda en un historial (quién, cuándo, qué). | Registro |
| RF-16 | Estado visible de cada operación (pendiente, enviando, guardada, fallida) y prevención de registros duplicados. | Registro ("no sabían si se había enviado") |
| RF-17 | Registro de actas sin conexión, con envío automático al recuperar la señal. | Conectividad |
| RF-18 | Panel del coordinador provincial con el avance por recinto: mesas reportadas y mesas pendientes. | Seguimiento |
| RF-19 | Resultados en tiempo real por candidato (alcaldía y prefectura), por recinto y en total, a medida que llegan las actas. | Seguimiento ("Exit Poll en tiempo real") |
| RF-20 | Captura automática de fallos (cierres inesperados y errores que impiden completar una tarea), con dispositivo, versión de Android y pantalla. Los reportes generados sin conexión se envían después. | Estabilidad |
| RF-21 | Datos precargados: recintos de la provincia y el cantón elegidos, 5 organizaciones políticas y sus candidatos por dignidad. | Restricciones |

### Requerimientos no funcionales

| Código | Requerimiento | Necesidad que lo origina |
|---|---|---|
| RNF-01 | **Confidencialidad por rol y ámbito:** el veedor solo ve y modifica sus mesas; el coordinador de recinto, solo su recinto. Esto debe aplicarse en el servidor, no solo ocultando pantallas. | Confidencialidad |
| RNF-02 | **Seguridad de credenciales:** las claves nunca se guardan ni se muestran en texto plano. La clave temporal deja de servir después del primer cambio. | Acceso |
| RNF-03 | **Usabilidad para adultos mayores:** textos y botones grandes, pocos pasos por flujo y mensajes en lenguaje simple. | Acceso (personas mayores, poca experiencia) |
| RNF-04 | **Integridad y trazabilidad:** cada acta tiene un autor identificable y su historial de correcciones; no se pierden ni duplican datos. | Acceso y Registro |
| RNF-05 | **Tolerancia a la falta de conectividad:** los datos y las fotos capturados sin señal sobreviven al cierre de la app o al reinicio del celular. | Conectividad |
| RNF-06 | **Observabilidad:** se puede saber a cuántos usuarios afecta un error, en qué dispositivo, en qué versión de Android y en qué pantalla. | Estabilidad |
| RNF-07 | **Retroalimentación visual** en cada operación (en proceso, éxito, fallo, validación) con animaciones (Lottie/Reanimated) y mensajes claros, sin modales genéricos. | Registro y rúbrica |
| RNF-08 | **Actualización casi en tiempo real** del panel provincial: pocos segundos después de que el acta llega al servidor. | Seguimiento |
| RNF-09 | **Plataforma y estructura:** React Native, APK instalable en Android y código organizado con Clean Architecture. | Restricciones |
| RNF-10 | **Identidad visual** con los colores de la organización política. | Rúbrica (diseño) |

---

## 2. Supuestos adoptados

| Código | Ambigüedad que resuelve | Supuesto adoptado y justificación |
|---|---|---|
| S-01 | Provincia y cantón no definidos. | Se elige **[provincia/cantón]**, preferiblemente un cantón con pocos recintos, y se precargan sus recintos y mesas reales del CNE. Así los datos de prueba son manejables y verosímiles. |
| S-02 | ¿Quién crea al coordinador provincial? | Hay **un único coordinador provincial, precargado** en el sistema (seed). Es la raíz de la jerarquía y nadie más puede crearlo. |
| S-03 | ¿Un veedor puede cubrir mesas de distintos recintos? | **No.** Un veedor pertenece a un solo recinto y puede cubrir varias mesas de ese recinto. El coordinador "se encarga de sus veedores" y la confidencialidad se define por recinto. |
| S-04 | Reemplazo de un veedor con una persona no afiliada. | El coordinador de recinto le crea la cuenta en el momento (RF-01). Al reasignar la mesa, el veedor anterior pierde el acceso a ella, pero las actas que ya registró conservan su autoría. |
| S-05 | ¿Hasta cuándo se pueden corregir las actas? | Durante toda la jornada, sin bloqueo. Cada corrección se guarda en el historial y prevalece la última versión. |
| S-06 | Validación del total de sufragantes. | Regla principal: válidos + nulos + blancos = total. Si el número de electores inscritos de la JRV está precargado, también se exige total ≤ electores. |
| S-07 | "Saber desde dónde se registró" vs. la imprecisión del GPS. | La ubicación es **obligatoria**: sin GPS no se envía el acta. Si el acta se registra a más de ~500 m del recinto, se **marca como alerta** en lugar de bloquearse, porque el GPS puede fallar en zonas rurales. |
| S-08 | ¿Qué es una "foto legible"? | Vista previa obligatoria con confirmación del veedor, resolución mínima y sin opción de elegir de la galería (solo cámara). Así se evitan fotos antiguas o de otro lugar. |
| S-09 | Significado de "Exit Poll en tiempo real". | Se interpreta como un **conteo rápido** basado en las actas registradas, no como una encuesta a votantes. |
| S-10 | Conflicto cuando veedor y coordinador corrigen la misma acta sin conexión. | **Gana la última escritura** según su marca de tiempo, y ambas versiones quedan en el historial. Es simple y no se pierde nada. |

---

## 3. Registro de decisiones técnicas (ADR)

> En "Propuesta por" va tu nombre. La opción marcada **(Recomendada)** es la que defiendo.

### ADR-01: Framework y forma de compilación (Expo vs. RN CLI)
**Requerimientos relacionados:** RNF-09, RNF-07, RF-13, RF-14, RF-20
**Contexto:** El uso de React Native y la entrega de un APK son obligatorios. La app necesita cámara, GPS, SQLite, Lottie, Reanimated y monitoreo de errores, y el plazo es corto.

| Alternativa | Propuesta por | Ventajas | Desventajas o limitaciones |
|---|---|---|---|
| **Expo (SDK actual) + development build / EAS Build** (Recomendada) | Oscar Vásquez | Módulos oficiales listos para cámara, ubicación, SQLite y archivos. Genera el APK con `eas build` (perfil APK) o localmente con `expo run:android`. Lottie, Reanimated y Sentry tienen plugin para Expo. | Dependencia del ecosistema Expo. EAS en la nube tiene una cola gratuita limitada (el build local es la alternativa). |
| React Native CLI | | Control total del código nativo. | Más configuración manual de Gradle y de los permisos; más riesgo de que el APK falle cerca de la entrega. |
| Expo Go únicamente || Arranque inmediato. | No genera APK y no admite algunas librerías nativas. **Descartada.** |

**Decisión:** Expo con development build. Reduce la configuración nativa y el riesgo con el APK, que cuesta 5 puntos si falta.
**Consecuencias:** Generar un APK de prueba desde el día 1. Usar solo librerías compatibles con Expo.

---

### ADR-02: Arquitectura del código (Clean Architecture)
**Requerimientos relacionados:** RNF-09, RF-12, RF-17
**Contexto:** Clean Architecture es obligatoria. Lo que hay que decidir es **cómo organizarla** para que sea fácil de defender y para que las reglas del acta funcionen también sin conexión.

| Alternativa | Propuesta por | Ventajas | Desventajas o limitaciones |
|---|---|---|---|
| **Capas globales: `domain` (entidades, casos de uso, interfaces de repositorio) / `data` (repositorios, fuentes remota y local) / `presentation` (pantallas, hooks, store) / `di`** (Recomendada) | [Yo] | Es el esquema clásico y fácil de mostrar en la defensa. Las validaciones del acta (RF-12) viven en `domain`, son funciones puras que se pueden probar y funcionan offline. | Las carpetas crecen a medida que se agregan funcionalidades. |
| Por funcionalidad (feature‑first), con capas dentro de cada una | | Escala mejor y agrupa por módulo (auth, actas, territorio). | Más carpetas; en un proyecto pequeño se duplica estructura. |
| MVVM simple sin casos de uso | | Más rápido de escribir. | **No cumple** Clean Architecture. Descartada. |

**Decisión:** Capas globales con casos de uso explícitos (`RegistrarActa`, `CorregirActa`, `AsignarVeedor`…). La UI depende de interfaces y nunca de Supabase directamente.
**Consecuencias:** Más archivos, pero se puede cambiar el backend o la base local sin tocar la UI. Las reglas de negocio quedan aisladas y son fáciles de explicar.

---

### ADR-03: Backend (BaaS): autenticación, base de datos, almacenamiento de fotos y tiempo real
**Requerimientos relacionados:** RF-01 a RF-06, RF-13, RF-18, RF-19, RNF-01, RNF-08
**Contexto:** Se necesita autenticación con correo verificado, una base de datos con jerarquía (provincia → recinto → mesa → acta), almacenamiento de fotos, resultados en tiempo real y agregaciones (sumas por candidato y por recinto).

| Alternativa | Propuesta por | Ventajas | Desventajas o limitaciones |
|---|---|---|---|
| **Supabase** (PostgreSQL + Auth + Storage + Realtime + Edge Functions) (Recomendada) | Oscar Vásquez | El modelo relacional encaja con la jerarquía. Las sumas por candidato son un `SUM … GROUP BY` o una vista SQL. **Row Level Security** para la confidencialidad. Auth incluye verificación por OTP y recuperación. Realtime para el panel. | No trae persistencia offline automática (hay que construirla, ver ADR-06/07). El plan gratuito pausa el proyecto si pasa días sin uso. |
| Firebase (Firestore + Auth + Storage) | | Excelente caché offline incorporada. Crashlytics en el mismo ecosistema. Mucha documentación. | NoSQL: las agregaciones y las relaciones jerárquicas son más incómodas (desnormalizar, contadores). Las reglas de seguridad complejas son difíciles de mantener. |
| Appwrite | | Open source, con auth, base de datos y storage. | SDK de React Native y comunidad menos maduros. Tiempo real y consultas agregadas más limitados. |

**Decisión:** Supabase. El problema central es relacional (jerarquía y sumas de votos) y exige confidencialidad por fila, y PostgreSQL con RLS lo resuelve de forma nativa.
**Consecuencias:** Hay que implementar el modo offline nosotros mismos (ADR-06/07). La creación de usuarios con privilegios se hace en una Edge Function, nunca en la app.

---

### ADR-04: Gestión de accesos (alta, primer ingreso, verificación y recuperación)
**Requerimientos relacionados:** RF-01 a RF-06, RNF-02, RNF-03, S-02, S-04
**Contexto:** Otra persona crea el acceso, pero solo el titular debe conocer su clave. El correo debe ser real y pertenecerle, y la recuperación debe ser autónoma. Los usuarios tienen poca experiencia con el celular.

| Alternativa | Propuesta por | Ventajas | Desventajas o limitaciones |
|---|---|---|---|
| **Alta por el superior con clave temporal → primer ingreso: verificación del correo con código OTP de 6 dígitos + cambio obligatorio de clave. Recuperación también con código OTP.** (Recomendada) | [Yo] | Coincide literalmente con el enunciado ("se lo entregue listo"). El código OTP es más fácil que un enlace para adultos mayores y no requiere *deep links* en el APK. | Requiere una Edge Function con `service_role` para crear usuarios y un indicador `debe_cambiar_clave` en el perfil. |
| Invitación por correo (enlace mágico); el usuario define su clave | | El creador nunca conoce ninguna clave. | El enlace debe abrir la app (deep linking) y es más confuso para personas mayores. |
| Auto‑registro con aprobación del superior | | Menos trabajo para los coordinadores. | **Contradice** el enunciado: ya falló antes por datos incompletos. Descartada. |

**Decisión:** Clave temporal + OTP + cambio obligatorio. Resuelve las tres necesidades (correo verificado, clave privada, recuperación autónoma) con el flujo más simple para el usuario.
**Consecuencias:** La navegación bloquea la app hasta completar el primer ingreso. Hay que configurar plantillas de correo con código en Supabase (y un SMTP propio si se supera el límite de correos del plan gratuito).

---

### ADR-05: Modelo de datos y control de acceso
**Requerimientos relacionados:** RF-07 a RF-11, RF-15, RF-18, RF-19, RNF-01, RNF-04, S-03, S-05
**Contexto:** La confidencialidad no puede depender de que la app "oculte" información: un usuario técnico podría consultar la API directamente.

| Alternativa | Propuesta por | Ventajas | Desventajas o limitaciones |
|---|---|---|---|
| **Tablas relacionales + Row Level Security por rol y ámbito + vistas SQL para resultados** (Recomendada) | [Yo] | La seguridad se aplica en la base de datos para cualquier cliente. Las vistas `avance_por_recinto` y `resultados_por_candidato` alimentan el panel. Una tabla `acta_historial` (vía trigger) da trazabilidad. | Hay que escribir y probar bien las políticas RLS. |
| Filtrar únicamente en la app | | Fácil. | **Inseguro**: no cumple RNF-01. Descartada. |
| API propia (Edge Functions) para todas las operaciones | | Control total de la lógica. | Más código y más latencia; reinventa lo que RLS ya resuelve. |

**Decisión:** RLS + vistas + trigger de historial. La seguridad depende del servidor y no de la interfaz.
**Consecuencias:** Tablas principales: `perfiles(rol, cedula UNIQUE, recinto_id)`, `recintos`, `mesas(veedor_id)`, `organizaciones`, `candidatos`, `actas(id UUID, mesa_id, dignidad, foto_url, lat, lng, precision, fecha)`, `acta_votos` y `acta_historial`. Restricción `UNIQUE(mesa_id, dignidad)` para impedir duplicados en la base de datos.

---

### ADR-06: Almacenamiento local (offline)
**Requerimientos relacionados:** RF-17, RF-16, RNF-05, S-10
**Contexto:** Los veedores rurales terminan el conteo sin señal. Las actas y sus fotos deben guardarse en el celular y sobrevivir al cierre de la app o al reinicio del teléfono.

| Alternativa | Propuesta por | Ventajas | Desventajas o limitaciones |
|---|---|---|---|
| **SQLite (`expo-sqlite`) + fotos en el sistema de archivos (`expo-file-system`)** (Recomendada) | [Yo] | Datos estructurados con transacciones y consultas (por ejemplo, "actas pendientes"). La base guarda solo la ruta de la foto, no la imagen. | Hay que escribir el esquema local y el mapeo. |
| AsyncStorage | | Muy simple (clave‑valor). | Sin consultas ni transacciones; límite de tamaño en Android; no sirve para fotos. |
| WatermelonDB | | Pensada para offline y trae un protocolo de sincronización. | Configuración nativa más compleja y curva de aprendizaje alta para el plazo disponible. |

**Decisión:** SQLite + sistema de archivos. Es robusto y estándar, y cada acta pendiente se puede consultar y mostrar con su estado.
**Consecuencias:** Las fotos se copian a un directorio permanente de la app (no a caché) y se borran después de sincronizarse.

---

### ADR-07: Estrategia de sincronización
**Requerimientos relacionados:** RF-16, RF-17, RNF-04, RNF-05, S-10
**Contexto:** En la elección anterior, los usuarios reintentaban el envío y duplicaban registros. El envío debe ser automático, sin duplicados y con un estado visible en todo momento.

| Alternativa | Propuesta por | Ventajas | Desventajas o limitaciones |
|---|---|---|---|
| **Cola de salida (outbox) en SQLite + ID UUID generado en el celular + `upsert` idempotente + disparo con NetInfo y reintentos con espera creciente** (Recomendada) | [Yo] | Si el envío se repite, actualiza el mismo registro en lugar de crear otro: **elimina los duplicados**. Cada acta tiene un estado claro (`pendiente → subiendo foto → enviando → sincronizada / error`). | Lógica propia que hay que probar (por ejemplo, en modo avión). |
| Mutaciones offline persistidas de TanStack Query | | Menos código. | Las fotos y la persistencia tras un reinicio son frágiles, y se controla menos el estado de cada acta. |
| PowerSync / ElectricSQL | | Sincronización bidireccional automática con Postgres. | Servicio adicional, configuración compleja y dependencia externa para un examen. |

**Decisión:** Outbox + UUID + `upsert`. Ataca directamente el problema de duplicados del enunciado y es fácil de explicar.
**Consecuencias:** Orden fijo: primero se sube la foto y después el acta con su URL. Los conflictos se resuelven con "gana la última escritura" y el historial (S-10). Los errores de sincronización se reportan al monitoreo (ADR-10).

---

### ADR-08: Gestión del estado
**Requerimientos relacionados:** RF-16, RF-19, RNF-07, RNF-09
**Contexto:** Hay estado de sesión (usuario y rol), estado de sincronización (pendientes) y datos del servidor que cambian en vivo (resultados).

| Alternativa | Propuesta por | Ventajas | Desventajas o limitaciones |
|---|---|---|---|
| **Zustand (sesión y estado de sincronización) + TanStack Query (datos del servidor, caché, carga y error)** (Recomendada) | [Yo] | Poco código repetitivo y fácil de explicar. TanStack Query entrega `isLoading` e `isError` para la retroalimentación visual (RNF-07). Realtime solo tiene que invalidar las consultas. | Son dos librerías y hay que tener clara la frontera entre ambas. |
| Redux Toolkit (+ RTK Query) | | Estándar maduro con herramientas de depuración. | Más código repetitivo para una app de este tamaño. |
| Context API + useReducer | | No añade dependencias. | Provoca renders innecesarios y no trae caché, reintentos ni estados de carga. |

**Decisión:** Zustand + TanStack Query. Separa con claridad el estado del cliente y el del servidor con poco código.
**Consecuencias:** Los stores viven en `presentation` y llaman a casos de uso, nunca directamente a Supabase (coherencia con ADR-02).

---

### ADR-09: Navegación por rol
**Requerimientos relacionados:** RF-03, RNF-01, RNF-03, rúbrica (flujos diferenciados por rol)
**Contexto:** Hay tres roles con flujos distintos, además de un flujo obligatorio de primer ingreso.

| Alternativa | Propuesta por | Ventajas | Desventajas o limitaciones |
|---|---|---|---|
| **React Navigation (native stack + bottom tabs) con navegadores condicionales: `AuthStack`, `PrimerIngresoStack`, `VeedorTabs`, `CoordRecintoTabs`, `ProvincialTabs`** (Recomendada) | [Yo] | Un usuario no puede llegar a las pantallas de otro rol porque esas pantallas no existen en su navegador. La lógica es explícita y fácil de defender. No impone una estructura de carpetas. | Hay que tipar las rutas a mano. |
| Expo Router (rutas por archivos) | | Rutas automáticas y deep linking incluido. | La carpeta `app/` impone su estructura y choca con las capas de Clean Architecture. Los grupos por rol son menos evidentes. |
| Una sola pila con validación del rol en cada pantalla | | Simple al inicio. | Propensa a errores de acceso y con flujos confusos. Descartada. |

**Decisión:** React Navigation con navegadores por rol, elegidos según la sesión y el indicador `debe_cambiar_clave`.
**Consecuencias:** La navegación queda en `presentation/navigation`. Las animaciones de transición usan Reanimated.

---

### ADR-10: Monitoreo de errores
**Requerimientos relacionados:** RF-20, RNF-06, RF-17
**Contexto:** El equipo debe enterarse de los fallos sin depender del usuario: cuántos afectados, dispositivo, versión de Android, pantalla, errores no fatales y envío diferido cuando no hay conexión. En la defensa hay que provocar un fallo y mostrar el reporte.

| Alternativa | Propuesta por | Ventajas | Desventajas o limitaciones |
|---|---|---|---|
| **Sentry (`@sentry/react-native` con plugin Expo)** (Recomendada) | [Yo] | Captura cierres nativos y de JavaScript, y errores manejados con `captureException` (por ejemplo, "falló la subida de la foto"). La integración con React Navigation indica la pantalla. Cuenta los usuarios afectados y registra dispositivo y versión de Android. **Guarda los eventos sin conexión y los envía después.** Plan gratuito suficiente. | Hay que subir los *source maps* para ver trazas legibles (el plugin lo automatiza en el build). |
| Firebase Crashlytics | | Muy buena captura de cierres nativos; gratuito. | Exige un proyecto Firebase solo para esto (mezcla ecosistemas con Supabase). Los errores de JavaScript y la pantalla se ven con menos detalle. |
| Bugsnag | | Buena estabilidad y métricas. | Plan gratuito más limitado y menos documentación para Expo. |

**Decisión:** Sentry. Cubre literalmente cada punto de la sección *Estabilidad*, incluidos los envíos diferidos sin conexión.
**Consecuencias:** Etiquetar los eventos con rol y recinto (sin datos sensibles como la cédula). Preparar un botón o escenario de prueba para provocar el fallo en la defensa.

---

### ADR-11: Captura de evidencia (foto + ubicación)
**Requerimientos relacionados:** RF-13, RF-14, S-07, S-08
**Contexto:** Las fotos deben ser legibles, tomadas en el lugar y con ubicación consultable al revisar el acta.

| Alternativa | Propuesta por | Ventajas | Desventajas o limitaciones |
|---|---|---|---|
| **`expo-camera` (cámara dentro de la app, sin galería) + `expo-location` con alta precisión + mapa en la revisión (`react-native-maps`)** (Recomendada) | [Yo] | Impide subir fotos viejas de la galería. Permite pantalla de vista previa, guía de encuadre y control de calidad. La ubicación y la hora se capturan junto con la foto. | `react-native-maps` en Android requiere una API key de Google Maps (alternativa: botón "Ver en Maps" con las coordenadas). |
| `expo-image-picker` (cámara del sistema) | | Muy simple. | Menos control sobre la calidad y la vista previa; en algunos casos permite elegir de la galería. |
| Foto opcional / ubicación opcional | | Menos fricción. | **Contradice** el enunciado ("un acta sin esa evidencia no nos sirve"). Descartada. |

**Decisión:** Cámara propia + GPS obligatorio + mapa al revisar.
**Consecuencias:** Pedir los permisos con mensajes claros. Comprimir la foto (por ejemplo, ancho de ~1600 px y JPEG al 70 %) para que suba con mala señal. Mostrar una alerta si el acta se registró a más de ~500 m del recinto (S-07).

---
