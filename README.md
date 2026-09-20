# Roblox Mirror — Entrega

## Qué es esto realmente

Este paquete contiene **cuatro artefactos ejecutables** (uno por fase acumulada) que juntos
demuestran de forma *funcional y verificable* un subconjunto del proyecto descrito en la
especificación original. No es una réplica completa de Roblox — eso es, por diseño del propio
documento (sección 35, "Limitaciones honestas"), un objetivo de años de ingeniería, no de una
sesión de chat. Lo que sí se entrega es real: código que corre, no una simulación de progreso.

Siguiendo la sección 30 del documento ("Compatibility Score — no usar un único porcentaje
superficial"), aquí va el estado por subsistema en vez de un número inventado del "100%":

| Subsistema                        | Estado                                                             |
|------------------------------------|----------------------------------------------------------------------|
| Renderer (WebGL2, Three.js)        | ✅ Funcional — baseplate, Noob, luces, sombras, cámara 3ra persona    |
| Input (teclado, mouse, touch)      | ✅ Funcional — WASD, salto, drag-to-look, joystick táctil             |
| DataModel / Instances / Services   | ✅ Funcional — jerarquía real, propiedades, eventos `Connect`/`fire`  |
| Studio (Explorer/Properties/Editor)| ✅ Funcional — árbol navegable, edición de propiedades en vivo        |
| Importador `.rbxlx`                | ✅ Funcional — parser XML real, Compatibility Report real             |
| Importador `.rbxl` (binario)       | ❌ No implementado — requiere parser de formato binario propietario   |
| Scripting                          | ⚠️ Parcial — subconjunto JS-hosted que imita la superficie de la API de Luau (`print`, `wait`, `Connect`, servicios). **No** es una VM de Luau real |
| Physics engine real                | ❌ No implementado — el movimiento actual es un controlador simple, no un motor físico completo |
| Networking cliente/servidor real   | ❌ No implementado — Client/Server hoy es una distinción lógica (RunContext), no dos procesos separados con replicación de red real |
| Security Lab (copia aislada)       | ✅ Funcional — clonación real del DataModel, nunca toca el proyecto original |
| Network Monitor                    | ✅ Funcional — registra cada `FireServer`/`FireClient` con validación  |
| Server Authority Analyzer          | ✅ Funcional — detecta el patrón "cliente decide un valor crítico"    |
| Threat Simulation                  | ✅ Funcional — invalid args, spam, manipulación de estado client-side |
| Vulnerability Report               | ✅ Funcional — genera `SECURITY_REPORT.md` real desde los findings    |
| Snapshots                          | ✅ Funcional — crear/restaurar el árbol completo de la Security Copy  |
| Defense Generator                  | ✅ Funcional — genera validación + test de regresión + explicación por finding (para revisión humana, no se aplica solo) |
| Multi-Place                        | ❌ No implementado                                                    |
| Terrain, Animation, Audio avanzado | ❌ No implementado                                                    |

## Archivos incluidos

- `mirror-phase1.html` — Fase 1: renderer, baseplate, Noob, cámara, input.
- `mirror-phase2.html` — + Fase 2/4: DataModel, Instances, Services, Studio (Explorer/Properties/Script Editor/Play-Stop).
- `mirror-phase3.html` — + Fase 5: Import Wizard con parser `.rbxlx` real y Compatibility Report.
- `mirror-final.html` — el artefacto más completo: DataModel/Studio (Fase 2/4) + **Import Wizard con parser `.rbxlx` real** (Fase 5) + Security Lab/Network Monitor/Server Authority Analyzer/Threat Simulation/Snapshots/Vulnerability Report (Fase 7) + Defense Generator (Fase 8). **Ábrelo primero.**

### Manejo de `.rbxl` (binario)

Mirror **no** convierte `.rbxl` a `.rbxlx` dentro del navegador — no existe una implementación
verificada del formato binario en este proyecto, y simularla habría significado arriesgarse a
un import silenciosamente incorrecto, justo lo que la sección 8 del documento prohíbe
("nunca inventar código"). En vez de eso, cuando sueltas un `.rbxl` en el Importer, Mirror te
muestra el comando exacto (con tu nombre de archivo ya insertado) para convertirlo con
`rbxmk` vía Termux en Android, y en cuanto tengas el `.rbxlx` resultante lo sueltas en el mismo
Importer para completar la importación real.

Cada archivo es autocontenido (HTML + JS, Three.js vía CDN) y corre en cualquier navegador
moderno sin instalar nada, cumpliendo la sección 17/19 del documento (web, sin app nativa,
sin depender de un servidor para ejecutar la experiencia importada).

## Lo que haría falta para un "100%" real

Por orden de impacto, según las Fases 6/9 del documento original:

1. **VM de Luau real** (vía WASM) — hoy los scripts corren sobre un subconjunto JS, no sobre
   bytecode Luau. Es el ítem que más limita compatibilidad real con proyectos existentes.
2. **Parser `.rbxl` binario** — hoy solo se soporta `.rbxlx` (XML).
3. **Motor de física real** (colisiones, restricciones, ragdoll) en vez del controlador simple actual.
4. **Separación cliente/servidor real** con replicación de red, no solo `RunContext` lógico.
5. **Terrain, animaciones, audio, Multi-Place.**

## Nota de seguridad

El Security Lab nunca ejecuta nada fuera del propio Mirror Runtime: no hay acceso a
filesystem, comandos del sistema operativo, credenciales, cookies del navegador, red externa
arbitraria, ni a otros procesos — tal como exige la sección 26 del documento original. Todo el
"ataque" simulado ocurre contra una copia clonada del propio proyecto, en memoria, en la
pestaña del navegador.
