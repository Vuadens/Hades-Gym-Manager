# Plan de acción: refactor y documentación

Este plan parte de la [auditoría técnica](auditoria-tecnica.md). Cada tarea cita el ID del hallazgo que resuelve (`B-xx` backend, `F-xx` frontend, `R-xx` repositorio).

## Principios del plan

1. **Primero la red de seguridad, después el refactor.** Los tests se escriben **antes** de tocar el código. Fijan el comportamiento actual y, de paso, cubren el requisito de la cátedra. *(Pragmatic: "Test early, test often, test automatically".)*
2. **Refactor sin cambio de comportamiento separado de las correcciones.** Un PR que reordena código no arregla bugs, y un PR que arregla un bug trae su test. Así cada PR se puede explicar en una frase.
3. **PRs chicos por tema y por submódulo.** El objetivo es que cada uno se lea en menos de 15 minutos. *(Pragmatic: "Take small steps, always".)*
4. **El código se explica solo.** Los comentarios quedan para el *por qué* (zonas horarias, decisiones de modelo), nunca para el *qué*. La explicación paso a paso va en `docs/flujos/`, no en comentarios.
5. **Una convención para todo el código** ([glosario](#glosario-y-convenciones) al final), aplicada por ESLint y Prettier para que no dependa de la memoria de nadie.

---

## Decisiones que necesitamos tomar antes de empezar

| # | Decisión | Opciones | Recomendación |
|:-:|:-|:-|:-|
| D1 | **¿Qué hacemos con el modo demo del frontend?** (F-01) | (a) Eliminarlo. (b) Conservarlo detrás de una interfaz `GymApi` con dos implementaciones (patrón *Adapter*). | **(a) Eliminarlo.** Es la fuente principal del *spaghetti* (65 usos en 23 archivos) y obliga a explicar dos aplicaciones. Fue una decisión del equipo (spec D9) para mostrar la interfaz sin backend, y hoy el deploy de Vercel corre en ese modo. Por eso eliminarlo **obliga a desplegar el backend y una base hosteada**, algo que la FAQ ya exige ("no debe requerir ejecutarse en local"). Con eso, la demo es el deploy real con el seed. Las pantallas que solo existen en demo (Máquinas, Materiales, Finanzas, sueldos) se rehacen contra el backend en la fase 5. |
| D2 | **¿Cómo repartimos los PRs del refactor?** | (a) Una persona hace todo. (b) Cada integrante refactoriza "su" CRUD y su test. | **(b).** La cátedra evalúa "participación de los miembros" y "uso de git". Si cada integrante es autor del refactor y del test de su CRUD, el historial lo muestra y cada uno puede defender su parte. Las tareas transversales (tooling, `asyncHandler`, glosario) las toma una persona y el resto las revisa. |
| D3 | ¿`asyncHandler` propio o migrar a Express 5? (B-11) | (a) Helper de 5 líneas. (b) Express 5, que captura promesas rechazadas solo. | **(a).** Cambio mínimo, fácil de explicar y sin riesgo de romper middlewares. |
| D4 | ¿Cómo se maneja el estado del servidor en el front? (F-02, F-07) | (a) Un store chico de Zustand por recurso + hooks. (b) TanStack Query. | **(a).** No suma dependencias, Zustand ya se conoce y los hooks quedan explícitos para la defensa. (b) es más potente, pero agrega conceptos (caché, invalidación por clave) que hay que saber defender. |
| D5 | ¿Qué hacemos con el descuento del 15% en planes? (B-04) | (a) Borrarlo. (b) Mantenerlo como `precioBase` + precio calculado. | Lo decide el negocio. Si nadie lo pidió, **(a)**. |

---

## Fase 0. Ordenar el repositorio (½ día, una persona)

| Tarea | Resuelve | Comando / detalle |
|:-|:-|:-|
| Actualizar el checkout local del frontend | R-01 | `git -C DSW-Frontend checkout main && git -C DSW-Frontend pull` |
| Borrar la copia suelta del backend dentro del front | R-03 | `rm -rf DSW-Frontend/DSW-Backend` |
| Sacar artefactos versionados y dejar un solo lockfile | R-04 | En el front: `git rm --cached tsconfig*.tsbuildinfo vite.config.js vite.config.d.ts tailwind.config.js tailwind.config.d.ts pnpm-lock.yaml` y sumarlos al `.gitignore`. En el back: `git rm -r --cached .idea`. |
| Declarar el stack en la proposal (React, Vite, Zustand, Tailwind, Express, Prisma, MySQL) y la metodología de trabajo, y confirmar la aprobación con el docente | FAQ, §4.4-4.5 de la auditoría | Sección "Tecnologías" y "Metodología" en `proposal.md` |
| Actualizar los punteros de submódulos en el repo padre | R-02 | Al cerrar cada fase: `git add DSW-Backend DSW-Frontend && git commit` en el padre |

---

## Fase 1. Herramientas y red de tests (2-3 días, en paralelo)

### 1.1 Herramientas comunes (una persona, PR en cada repo)

- ESLint (config Airbnb-TypeScript o `typescript-eslint` recomendado) + Prettier, con reglas `no-explicit-any`, `consistent-type-imports`, `eqeqeq` y `prefer-const`. *(R-05)*
- Scripts `lint`, `format` y `test:coverage` en los dos `package.json`.
- Backend: `jest.config.ts` con `ts-jest`, `setupFiles` que cargue `.env.test` y una base MySQL de test (Docker o una segunda base). *(requisito de aprobación)*
- Frontend: `vitest.config.ts` con `environment: 'jsdom'` y `setupFiles: ['@testing-library/jest-dom/vitest']`, más `cypress.config.ts`. *(requisito de aprobación)*

### 1.2 Tests mínimos exigidos por la cátedra (1 por integrante + integración + componente + e2e)

| Test | Tipo | Qué fija | Responsable sugerido |
|:-|:-|:-|:-|
| `cuota.service.test.ts`: `estadoDeCuotas`, `mesesImpagos`, `primerMesAdeudado` | Unitario (back) | Reglas de mora y vencimiento (día 10) del socio | Integrante del CRUD **Socio** |
| `actividad.service.test.ts` (con el repo mockeado) | Unitario (back) | Nombre único, límite de cupo, planes existentes | Integrante del CRUD **Actividad** |
| `profesor.service.test.ts` (con el repo mockeado) | Unitario (back) | Baja lógica: no permite dar de baja con clases asignadas, ni dos veces; reactivación | Integrante del CRUD **Profesor** |
| `plan.service.test.ts` (con el repo mockeado) | Unitario (back) | Nombre único y la regla del descuento (según D5) | Integrante del CRUD **Plan** |
| `inscripciones.int.test.ts` con `supertest` contra una base de test | Integración (back) | `POST /api/inscripciones`: 201, 400 sin cupo, 403 otro socio, 401 sin token | Todo el equipo |
| `EntityPage.test.tsx` o `Paginacion.test.tsx` | Unitario de componente (front) | Render, búsqueda, paginado, callbacks de output | A definir (rotativo) |
| `reserva.cy.ts` | E2E (front) | Login como socio → reservar → ver en "Mis reservas" → cancelar | Todo el equipo |
| *(extra)* `fechas.test.ts`: `proximaFecha`, `inicioDeClase`, `cumpleEdad` | Unitario (back) | Zona horaria de Argentina | Opcional |

> Cada test se corre una vez **antes** del refactor para guardar la línea de base. La salida de `npm test -- --coverage` se guarda como evidencia en `docs/evidencia-tests/`.

---

## Fase 2. Backend: un estilo único (3-4 días)

PRs en este orden. Ninguno cambia el comportamiento salvo los marcados con 🐛, que llevan su test.

| PR | Contenido | Resuelve |
|:-:|:-|:-|
| B-PR1 | `utils/async-handler.ts`, `schemas/comunes.ts` (`idParamSchema`, `diaSemanaSchema`, `horaSchema`) y `validateParams` que reemplaza `req.params`. `router.param` o `validateParams` en todas las rutas `/:id`. | B-05, B-11, B-14, B-23 |
| B-PR2 | Controllers con un patrón único: `export const xController = { listar, obtener, crear, actualizar, eliminar }`, envueltos en `asyncHandler`. Sin `try/catch` ni `parsearId`. Un PR por recurso, a cargo de su "dueño" (D2). | B-10, B-11, B-17 |
| B-PR3 | Repositories con el mismo patrón de export (objeto). Los `include` compartidos se mueven a `repositories/includes.ts`. Se borra `src/models/`. | B-13, B-21 |
| B-PR4 | Tipos de plan sin `any` (`PlanInput = z.infer<…>`); el controller pasa `req.body` sin reconvertir. 🐛 Decisión D5 sobre el descuento. | B-04, B-12 |
| B-PR5 | 🐛 Transacciones: reserva con cupo atómico, y baja o reactivación de socio en `$transaction`. Test de concurrencia en la integración. | B-01, B-03 |
| B-PR6 | 🐛 `aplicarCambiosDePlan` sale del middleware y pasa a un job diario (`node-cron`) y/o a una llamada explícita dentro de `inscripcionService.reservar` y `cuotaService.registrarCobro`. | B-02 |
| B-PR7 | 🐛 Reglas faltantes y códigos HTTP: `clase.cupo ≤ actividad.cupoMaximo` y la tabla de errores que ya está en el README del back (400/404/409) aplicada en todos los services. | B-06, B-07 |
| B-PR8 | Autorización: los roles se deciden en la ruta y la propiedad en el service. Se quitan los chequeos de rol duplicados. `GET /socios` queda solo para el staff. `Rol.*` en todas las rutas. Se quita el `PUT` duplicado. | B-08, B-09, B-18, B-19 |
| B-PR9 | `authService.registrarSocio`, `errorHandler(err: unknown)`, entorno validado con Zod y CORS restringido. Se borran comentarios de ruido. | B-16, B-20, B-22, B-24 |
| B-PR10 *(opcional, con migración)* | Nombres del modelo: `Actividad.idActividad`, `Cuota.idSocio`, `Usuario.idPersona` (con `@map` para no renombrar columnas). | B-15 |
| B-PR11 | **Endpoints que necesita el front para dejar de duplicar reglas**: `GET /socios/:id/estado-cuotas`, `GET /cuotas?periodo=&incluirPendientes=true` y `puedeReservar`/`motivo` en `GET /clases?fecha=` para el socio autenticado. | habilita F-03 |

---

## Fase 3. Frontend: una sola aplicación (4-5 días)

Depende de D1 y D4 y de B-PR11.

| PR | Contenido | Resuelve |
|:-:|:-|:-|
| F-PR1 | **Se elimina el modo demo**: `data/seed.ts`, `lib/features.ts`, todas las ramas `API_ENABLED`/`MODULOS_DEMO`, `mailer.service`, `photos.service`, `RecoverPage`, `PhotoModal` y los campos solo-demo de `models/` (`sueldos`, `clasesPorPeriodo`, `tipo`, `materiales`, `maquinas`, `metodoPago`, `ocupadosBase`). Las rutas Máquinas, Materiales y Finanzas se ocultan hasta la fase 5. | F-01, F-05, F-09 |
| F-PR2 | **Modelos con el vocabulario del backend** (`diaSemana: 'LUNES'`, `horaDesde`, `planId`, `estado: 'CONFIRMADA'`, `rol: 'ADMINISTRADOR'`). Las etiquetas legibles quedan solo para mostrar (`etiquetaDia('LUNES') → 'Lun'`). Los servicios pierden la mitad de sus mapeadores. | F-14 |
| F-PR3 | **Se parte el `dataStore`**: `hooks/useSocios.ts`, `useClases.ts`, `useInscripciones.ts`, `useCuotas.ts`… Cada uno tiene su store chico y llama a su `*.service.ts`. `EntityPage` recibe `onGuardar`/`onEliminar` tipados en vez de un string de colección. Se borran los 16 casts. | F-02, F-11, F-12 |
| F-PR4 | **Cada página pide sus datos** al montarse, usando los filtros del backend (`?dia=`, `?planId=`, `?fecha=`). Se quitan `hydrate` y la recarga global. | F-07, F-08, F-13 |
| F-PR5 | **Se borran las reglas duplicadas**: `lib/rules.ts` queda solo con etiquetas, y `puedeReservar`, mora y `blockers` se toman del backend (B-PR11). Las edades mínimas y el DNI se validan en el formulario solo como ayuda, y el mensaje final es el del backend. | F-03, F-06 |
| F-PR6 | `useHoy()` en lugar de `export let TODAY`. Se borra el duplicado de `vite-env.d.ts`, el comentario desactualizado de `SocioDetalleDialog` y las cuentas demo fuera de `DEV`. | F-04, F-15, F-16, F-18 |
| F-PR7 | **Tailwind mobile-first**: los tokens pasan a `tailwind.config.ts`, empezando por los componentes comunes (`Shell`, `Tile`, `Fields`, `Dialog`, `EntityPage`) y siguiendo página por página. Breakpoints `sm`, `md` y `lg`. Se borran los `style={{}}` y las clases crípticas. Un PR por grupo de pantallas. | F-17, F-19, F-20, F-21 |
| F-PR8 | `POST /profesores` con `turnos: number[]` (cambio coordinado con el back) y se borra `sincronizarTurnos`. | F-10 |

---

## Fase 4. Documentación paso a paso (en paralelo con las fases 2 y 3)

Estructura propuesta, que cumple el formato de [docs.md](../docs.md):

```
docs/
├── README.md                 ← punto de entrada (índice)
├── auditoria-tecnica.md
├── plan-de-accion.md
├── arquitectura.md           ← capas, diagrama de componentes, glosario
├── flujos/
│   ├── 01-login-y-primer-ingreso.md
│   ├── 02-reservar-clase.md          ← CUU 1
│   ├── 03-registrar-cobro-cuota.md   ← CUU 2
│   ├── 04-listado-clases-por-dia.md
│   └── 05-listado-socios-por-plan.md
├── api/                      ← OpenAPI generado (ver abajo)
├── evidencia-tests/          ← salida de jest/vitest/cypress con coverage
└── minutas/                  ← una por reunión: fecha, presentes, decisiones, tareas
```

**Formato de cada flujo** (es lo que permite "explicar cada línea" en la defensa):

1. **Qué hace el usuario** (una frase y una captura).
2. **Diagrama de secuencia Mermaid** con todas las capas.
3. **Tabla paso a paso**: `# | capa | archivo:línea | qué hace | por qué`. Va desde el `onClick` del componente hasta el `prisma.*` del repositorio y la vuelta.
4. **Reglas de negocio involucradas**, con el código de error que devuelve cada una.
5. **Test que lo cubre** (link al archivo).

**Documentación de la API (aprobación):** generar OpenAPI desde los schemas Zod que ya existen con `@asteasolutions/zod-to-openapi` y servirla con `swagger-ui-express` en `/api/docs`. Los schemas Zod se vuelven la única fuente de verdad de la validación y de la documentación (DRY).

---

## Fase 5. Alcance de aprobación pendiente (después del refactor)

| Ítem | Estado | Comentario |
|:-|:-|:-|
| CRUD Material | Solo demo en el front | Modelo Prisma, endpoints y pantalla |
| CRUD Máquina + CUU 3 "Orden de mantenimiento" | Solo demo en el front (B#18 sin mergear) | La pantalla `MaquinasPage` existe y se adapta al backend |
| CUU 4 "Histórico de sueldo del profesor" | Solo demo en el front | Modelo `HistoricoSueldo` y endpoint |
| UI de Administrador | Backend listo | Pantalla CRUD reutilizando `EntityPage` |
| Listado de mantenimientos por fecha (alcance adicional) | Pendiente | Sale fácil una vez hecho el CUU 3 |
| Deploy del backend con base hosteada | Falta | Requisito de la FAQ ("no debe requerir ejecutarse en local"). Pasa a ser obligatorio si se elimina el modo demo (D1) |
| Deploy y credenciales en la proposal | Parcial | Link de Vercel (hoy en modo demo) y del backend, más usuario de prueba |

---

## Cronograma orientativo

| Semana | Fases | Entregable |
|:-:|:-|:-|
| 1 | 0 + 1 | Repo limpio, lint, **7 tests en verde** (requisito de aprobación cubierto), stack declarado en la proposal |
| 2 | 2 | Backend con estilo único y bugs críticos corregidos; `docs/flujos` 01-03 |
| 3 | 3 | Frontend sin modo demo, hooks por recurso, Tailwind mobile-first |
| 4 | 4 + 5 | OpenAPI, flujos completos, minutas, CRUDs faltantes |

---

## Glosario y convenciones

| Concepto | Nombre en el código | Ejemplo |
|:-|:-|:-|
| Identificador | `id<Entidad>` | `idSocio`, `idClase`, `idPersona` |
| Día de la semana | `diaSemana: DiaSemana` (enum de Prisma) | `'LUNES'` |
| Franja horaria | `horaDesde`, `horaHasta` (`"HH:mm"`) | `'18:00'` |
| Período de cuota | `periodo: "AAAA-MM"` (string), `mesAnio: Date` en la base | `'2026-11'` |
| Acciones de controller | `listar`, `obtener`, `crear`, `actualizar`, `eliminar`, más los verbos del caso de uso (`reservar`, `cancelar`, `registrarCobro`, `darDeBaja`, `reactivar`) | `inscripcionController.reservar` |
| Acciones de repository | `buscarTodos`, `buscarPorId`, `crear`, `actualizar`, `eliminar` | `socioRepository.buscarPorId` |
| Idioma | Dominio en castellano; términos técnicos del framework en inglés (`req`, `res`, `next`, `router`) | |
| Errores | `HttpError(status, mensaje)`, según la tabla del README del back: 400 dato inválido o regla incumplida, 401 sin sesión, 403 sin permiso, 404 no existe, 409 conflicto de estado o valor repetido | |
| Respuestas | `GET` → recurso o lista; `POST` → 201 + recurso; `PATCH` → recurso; `DELETE` → 204 | |
