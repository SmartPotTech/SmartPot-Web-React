<!-- portada
eyebrow: Component Technical Documentation
titulo: SmartPot-Web-React (Legacy Stack A)
acento: Web
subtitulo: Legacy Single-Page Web Application
bajada: Legacy React PWA: monitoring dashboard, crop lifecycle configuration, graphical telemetry visualizer, AI integration, notification preferences, security hardening, and test suites.
documento: SmartPot-Web-React Documentation
version: 1.1 · October 2026
equipo: SmartPotTech
proyecto: smartpot.app
-->

# SmartPot-Web-React (Legacy Stack A)

> [!WARNING]
> **Componente Legado (Stack A)**: Este componente ha sido reemplazado en producción por la PWA canónica en SolidJS 1.9 + Bun ([`SmartPot-Web`](https://github.com/SmartPotTech/SmartPot-Web)). Este documento describe la implementación original en React 19 como registro histórico.

## Document Summary

| Field                          | Value                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
|--------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Proyecto                       | SmartPot · [smartpot.app](https://smartpot.app)                                                                                                                                                                                                                                                                                                                                                                                                         |
| Componente                     | [SmartPot-Web](https://github.com/SmartPotTech/SmartPot-Web)                                                                                                                                                                                                                                                                                                                                                                                            |
| Versión                        | 1.1 · octubre 2026                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| Alcance                        | Pantallas, creación de cultivos, ilustración del cultivo, sesión, caché, identidad visual, seguridad web, configuración y pruebas                                                                                                                                                                                                                                                                                                                       |
| Platform Documentation | [Technical Documentation](https://github.com/SmartPotTech/.github/blob/main/docs/SmartPot_Technical_Documentation.md), [Project Journey](https://github.com/SmartPotTech/.github/blob/main/docs/SmartPot_Project_Journey.md), [Software Lifecycle](https://github.com/SmartPotTech/.github/blob/main/docs/SmartPot_Software_Lifecycle.md), and [Global Diagrams](https://github.com/SmartPotTech/.github/blob/main/docs/README.md#global-diagrams) |
| Maintenance                  | Generated from `docs/` in this repository via `.github/docs/tools`; updated with every component change                                                                                                                                                                                                                                                                                                                   |

<!-- parte: PART I | Component Overview -->

## 1. Purpose

### In Plain Terms

SmartPot-Web es lo que ve la persona: una página pública que explica SmartPot y, tras ingresar, una aplicación que se
instala en el teléfono o el computador. Muestra cada cultivo en vivo, lo compara con el rango ideal de su especie, deja
crear cultivos reales o virtuales, dar órdenes a los actuadores y ver qué piensa y qué aprende el asistente. Todo lo que
muestra viene de la API.

| Pantalla                      | Qué permite                                                                                                                                                                                                                                     |
|-------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Inicio público                | Presentación, funciones, especies y preguntas frecuentes; indexable                                                                                                                                                                             |
| Panel general                 | Salud de cada cultivo, comparación de una variable, últimas lecturas y análisis de la IA de toda la cuenta                                                                                                                                      |
| Mis cultivos                  | Tarjetas con tipo, especie, forma, estado y salud; asistente de creación                                                                                                                                                                        |
| Detalle                       | La ilustración del cultivo en su lugar sobre todas las secciones: resumen, asistente IA con el consejo de lugar, control con switches, historial, dispositivo (reales) o simulación (virtuales) y ajustes con el lugar y los avisos del cultivo |
| Control general y acciones    | Modo automático y órdenes en bloque; sugerencias e historial de órdenes                                                                                                                                                                         |
| Aprendizaje, alertas y perfil | Lo que aprende la IA, notificaciones, Aplicaciones (Telegram), datos y contraseña                                                                                                                                                               |

## 2. Component Architecture

<!-- diagrama: SmartPot_Web_Global_Component | titulo=SmartPot-Web Component Architecture | lamina=H -->

```mermaid
%%{init: {"theme": "base", "fontFamily": "Segoe UI, Arial, sans-serif", "themeVariables": {"fontFamily": "Segoe UI, Arial, sans-serif", "fontSize": "15px", "primaryColor": "#DDF5EA", "primaryTextColor": "#17261F", "primaryBorderColor": "#067A52", "secondaryColor": "#E3F2FB", "secondaryTextColor": "#17261F", "secondaryBorderColor": "#1F6FA0", "tertiaryColor": "#F2F7F4", "tertiaryTextColor": "#17261F", "tertiaryBorderColor": "#D5E3DC", "lineColor": "#5B6B63", "textColor": "#17261F", "mainBkg": "#DDF5EA", "nodeBorder": "#067A52", "clusterBkg": "#F7FAF8", "clusterBorder": "#D5E3DC", "edgeLabelBackground": "#FFFFFF", "actorBkg": "#067A52", "actorBorder": "#0B3D2B", "actorTextColor": "#FFFFFF", "actorLineColor": "#5B6B63", "signalColor": "#17261F", "signalTextColor": "#17261F", "labelBoxBkgColor": "#0B3D2B", "labelBoxBorderColor": "#0B3D2B", "labelTextColor": "#FFFFFF", "loopTextColor": "#0B3D2B", "noteBkgColor": "#FDF4DD", "noteBorderColor": "#C98D12", "noteTextColor": "#17261F", "activationBkgColor": "#DDF5EA", "activationBorderColor": "#067A52", "attributeBackgroundColorOdd": "#FFFFFF", "attributeBackgroundColorEven": "#F2F7F4"}, "layout": "elk", "elk": {"nodePlacementStrategy": "BRANDES_KOEPF", "mergeEdges": false, "cycleBreakingStrategy": "GREEDY"}}}%%
flowchart LR
  browser["Web Browser / PWA Client"]
  subgraph react["SmartPot-Web-React · React 19 · Vite + Tailwind"]
    direction TB
    router["React Router v7/v8<br/>View Navigation"]
    pages["Pages & Features<br/>Dashboard · Crops · Control · Actions"]
    apiClient["Axios HTTP Client<br/>JWT Bearer Auth"]
  end
  api["SmartPot-API-Java"]
  browser --> router --> pages --> apiClient --> api
  classDef core fill:#067A52,stroke:#0B3D2B,color:#FFFFFF
  classDef leaf fill:#DDF5EA,stroke:#067A52,color:#17261F
  class browser,api core
  class react,router,pages,apiClient leaf
```

Cada función vive en `src/features/<función>` con sus páginas, componentes y pruebas. `lib/api` es el único lugar que
habla con la API: arma las peticiones con el token, traduce los errores a `ApiError` en español y declara los tipos de
cada respuesta. `useResource` carga y refresca los datos mientras la pestaña está visible.

<!-- parte: PART II | Crops and Interactive Scene -->

## 3. Crop Lifecycle and Creation

<!-- diagrama: SmartPot_Web_01_Crop_Creation_Flow | titulo=Asistente para crear un cultivo -->

```mermaid
%%{init: {"theme": "base", "fontFamily": "Segoe UI, Arial, sans-serif", "themeVariables": {"fontFamily": "Segoe UI, Arial, sans-serif", "fontSize": "15px", "primaryColor": "#DDF5EA", "primaryTextColor": "#17261F", "primaryBorderColor": "#067A52", "secondaryColor": "#E3F2FB", "secondaryTextColor": "#17261F", "secondaryBorderColor": "#1F6FA0", "tertiaryColor": "#F2F7F4", "tertiaryTextColor": "#17261F", "tertiaryBorderColor": "#D5E3DC", "lineColor": "#5B6B63", "textColor": "#17261F", "mainBkg": "#DDF5EA", "nodeBorder": "#067A52", "clusterBkg": "#F7FAF8", "clusterBorder": "#D5E3DC", "edgeLabelBackground": "#FFFFFF", "actorBkg": "#067A52", "actorBorder": "#0B3D2B", "actorTextColor": "#FFFFFF", "actorLineColor": "#5B6B63", "signalColor": "#17261F", "signalTextColor": "#17261F", "labelBoxBkgColor": "#0B3D2B", "labelBoxBorderColor": "#0B3D2B", "labelTextColor": "#FFFFFF", "loopTextColor": "#0B3D2B", "noteBkgColor": "#FDF4DD", "noteBorderColor": "#C98D12", "noteTextColor": "#17261F", "activationBkgColor": "#DDF5EA", "activationBorderColor": "#067A52", "attributeBackgroundColorOdd": "#FFFFFF", "attributeBackgroundColorEven": "#F2F7F4"}}}%%
flowchart TB
  start(["Crops Page › Create Crop"]) --> modal["Crop Creation Wizard"]
  modal --> kind{"Select Kind: Real or Virtual"}
  kind -->|"Real"| realForm["Name, Species, System Form"] --> postReal["POST /api/v1/crops"]
  kind -->|"Virtual"| virtForm["Simulation Mode & Location"] --> postVirt["POST /api/v1/crops"]
  postReal --> showKey["Display Device Key (One-time modal)"]
  showKey --> detail(["Crop Detail View"])
  postVirt --> detail
  classDef core fill:#067A52,stroke:#0B3D2B,color:#FFFFFF
  classDef leaf fill:#DDF5EA,stroke:#067A52,color:#17261F
  class start,detail core
  class modal,kind,realForm,virtForm,postReal,postVirt,showKey leaf
```

| Decisión     | Detalle                                                                                                                                                                                       |
|--------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Tipo         | Real (un ESP32 con el firmware, físico o en Wokwi) o virtual (lo simula SmartPot). No cambia después; Ajustes lo muestra y explica cómo crear otro                                            |
| Forma        | Maceta, tubos NFT, torre vertical o balsa flotante; se puede cambiar en Ajustes y solo afecta la ilustración                                                                                  |
| Lugar        | Bajo techo o al aire libre, cuánto sol recibe y la ubicación; se puede cambiar en Ajustes                                                                                                     |
| Vista previa | La escena del cultivo con la especie y la forma elegidas y los actuadores con los que nace                                                                                                    |
| Real         | Muestra la clave una sola vez y la guía: circuito y pines, firmware y `config.py` con la red WiFi y la CA del broker (que toma de `/ca.crt`), o el proyecto de Wokwi con la red `Wokwi-GUEST` |
| Virtual      | Día y noche, clima real (buscador de lugares o ubicación del dispositivo) o manual; abre el cultivo y su configuración queda en la pestaña Simulación                                         |

## 4. Ilustración del cultivo

<!-- diagrama: SmartPot_Web_02_Live_Scene | titulo=Cómo se dibuja la ilustración del cultivo | lamina=H -->

```mermaid
%%{init: {"theme": "base", "fontFamily": "Segoe UI, Arial, sans-serif", "themeVariables": {"fontFamily": "Segoe UI, Arial, sans-serif", "fontSize": "15px", "primaryColor": "#DDF5EA", "primaryTextColor": "#17261F", "primaryBorderColor": "#067A52", "secondaryColor": "#E3F2FB", "secondaryTextColor": "#17261F", "secondaryBorderColor": "#1F6FA0", "tertiaryColor": "#F2F7F4", "tertiaryTextColor": "#17261F", "tertiaryBorderColor": "#D5E3DC", "lineColor": "#5B6B63", "textColor": "#17261F", "mainBkg": "#DDF5EA", "nodeBorder": "#067A52", "clusterBkg": "#F7FAF8", "clusterBorder": "#D5E3DC", "edgeLabelBackground": "#FFFFFF", "actorBkg": "#067A52", "actorBorder": "#0B3D2B", "actorTextColor": "#FFFFFF", "actorLineColor": "#5B6B63", "signalColor": "#17261F", "signalTextColor": "#17261F", "labelBoxBkgColor": "#0B3D2B", "labelBoxBorderColor": "#0B3D2B", "labelTextColor": "#FFFFFF", "loopTextColor": "#0B3D2B", "noteBkgColor": "#FDF4DD", "noteBorderColor": "#C98D12", "noteTextColor": "#17261F", "activationBkgColor": "#DDF5EA", "activationBorderColor": "#067A52", "attributeBackgroundColorOdd": "#FFFFFF", "attributeBackgroundColorEven": "#F2F7F4"}}}%%
flowchart LR
  subgraph state["React State Hooks"]
    direction TB
    crop["useCropState()<br/>crop details & latest telemetry"]
    acts["useActuators()<br/>active states"]
  end
  subgraph scene["CropScene SVG Component"]
    direction TB
    plant["Dynamic Plant Graphic<br/>colored by health status"]
    actuators["Actuator Indicators<br/>pump, lamp, fan icons"]
  end
  state --> scene
  classDef core fill:#067A52,stroke:#0B3D2B,color:#FFFFFF
  classDef leaf fill:#DDF5EA,stroke:#067A52,color:#17261F
  class state,scene,crop,acts,plant,actuators leaf
```

| Pieza                      | Qué dibuja                                                                                                                                                                                                                                                                            |
|----------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Maceta                     | Depósito con la bomba, tubo que gotea sobre el sustrato y dosificadores sobre la tapa                                                                                                                                                                                                 |
| Tubos NFT                  | Tres tubos con cuatro plantas cada uno, colector de entrada y retorno al depósito; la película de solución corre si la bomba está encendida                                                                                                                                           |
| Torre vertical             | Columna con bolsillos escalonados; la bomba sube la solución y cae en gotas por dentro                                                                                                                                                                                                |
| Balsa flotante             | Estanque con la balsa, raíces en la solución y burbujas desde la piedra difusora                                                                                                                                                                                                      |
| Luz ultravioleta           | Barra con LED; encendida ilumina las plantas                                                                                                                                                                                                                                          |
| Ventilador y humidificador | Aspas que giran con corriente de aire; bruma que sube                                                                                                                                                                                                                                 |
| Plantas                    | La especie tal como es: lechuga en roseta, espinaca de hoja ancha, albahaca de hojas pareadas, tomate con tutor, flores y frutos rojos, fresa con flor blanca y frutos, pimentón rojo y amarillo; las hojas con el color de su salud: verde sano, amarillento en riesgo, ocre crítico |

La escena usa la paleta de SmartPot, anuncia su contenido con `aria-label` (qué se ve y qué está encendido) y sus
animaciones se detienen si la persona prefiere menos movimiento. Va encima de todas las secciones del detalle (
`CropHero`). Junto a ella, cada actuador aparece como una etiqueta con su estado; no hay botones: las órdenes se dan en
Control. Si el cultivo no está conectado no se ilustra: se explica cómo conectarlo (reales) o se lleva a la pestaña
Simulación para reanudarla (virtuales). La pestaña Simulación, que solo aparece en los virtuales, tiene dos partes como
Dispositivo: arriba el estado en línea con Pausar y Reanudar; abajo cómo se simula (modo, lugar, medidores y frecuencia)
con «Aplicar cambios».

El lugar cambia el fondo: bajo techo, una ventana con el cielo de afuera (sol que entra si es soleada, cortina si no
tiene luz natural); al aire libre, el clima del lugar, con una malla de media sombra o un árbol que da sombra. Junto a
la escena se escribe dónde está el cultivo y el clima de afuera; solo los virtuales llevan una etiqueta de tipo.

En Control, cada actuador es un switch: al encenderlo se elige la duración (o sin límite en la luz, el ventilador y el
humidificador), mientras espera la confirmación muestra «Enviando…» y después la cuenta regresiva. La API rechaza con
409 la orden que no cambia nada, así un doble clic nunca repite órdenes. En Ajustes, cada cultivo elige sus avisos por
Telegram cuando el chat está vinculado; Perfil › Aplicaciones muestra el canal en gris, con lo que le falta al
servidor, si no está disponible.

<!-- parte: PART III | State, Configuration and Operations -->

## 5. Sesión, caché y seguridad

<!-- diagrama: SmartPot_Web_03_Session_And_Cache | titulo=Sesión y caché de la PWA -->

```mermaid
%%{init: {"theme": "base", "fontFamily": "Segoe UI, Arial, sans-serif", "themeVariables": {"fontFamily": "Segoe UI, Arial, sans-serif", "fontSize": "15px", "primaryColor": "#DDF5EA", "primaryTextColor": "#17261F", "primaryBorderColor": "#067A52", "secondaryColor": "#E3F2FB", "secondaryTextColor": "#17261F", "secondaryBorderColor": "#1F6FA0", "tertiaryColor": "#F2F7F4", "tertiaryTextColor": "#17261F", "tertiaryBorderColor": "#D5E3DC", "lineColor": "#5B6B63", "textColor": "#17261F", "mainBkg": "#DDF5EA", "nodeBorder": "#067A52", "clusterBkg": "#F7FAF8", "clusterBorder": "#D5E3DC", "edgeLabelBackground": "#FFFFFF", "actorBkg": "#067A52", "actorBorder": "#0B3D2B", "actorTextColor": "#FFFFFF", "actorLineColor": "#5B6B63", "signalColor": "#17261F", "signalTextColor": "#17261F", "labelBoxBkgColor": "#0B3D2B", "labelBoxBorderColor": "#0B3D2B", "labelTextColor": "#FFFFFF", "loopTextColor": "#0B3D2B", "noteBkgColor": "#FDF4DD", "noteBorderColor": "#C98D12", "noteTextColor": "#17261F", "activationBkgColor": "#DDF5EA", "activationBorderColor": "#067A52", "attributeBackgroundColorOdd": "#FFFFFF", "attributeBackgroundColorEven": "#F2F7F4"}}}%%
flowchart TB
  login(["User Sign In"]) --> postAuth["POST /api/v1/auth/login"]
  postAuth --> store["Store JWT Token in localStorage"]
  store --> client["API Client Axios Interceptor<br/>Authorization: Bearer token"]
  client --> api["SmartPot-API-Java"]
  classDef core fill:#067A52,stroke:#0B3D2B,color:#FFFFFF
  classDef leaf fill:#DDF5EA,stroke:#067A52,color:#17261F
  class login,api core
  class postAuth,store,client leaf
```

| Control               | Detalle                                                                                                                                               |
|-----------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------|
| CSP                   | `script-src 'self'`, `style-src 'self'`, `connect-src` limitado a la API                                                                              |
| Cabeceras             | `X-Frame-Options: DENY`, `nosniff`, `Referrer-Policy` y `Permissions-Policy` (geolocalización solo del propio sitio y solo cuando la persona la pide) |
| Sesión                | JWT en `sessionStorage`, o en `localStorage` con «Mantener sesión iniciada»; un 401 la cierra                                                         |
| Clave del dispositivo | Solo se muestra al crear un cultivo real o al rotarla                                                                                                 |
| SEO                   | Metadatos, Open Graph, JSON-LD (`Organization`, `WebSite`, `SoftwareApplication`, `FAQPage`), `robots.txt` y `sitemap.xml`; `/app` no se indexa       |

## 6. Configuration & Environment

| Variable       | Dónde                | Uso                                                                                              |
|----------------|----------------------|--------------------------------------------------------------------------------------------------|
| `VITE_API_URL` | `.env` en desarrollo | URL de la API para `pnpm dev`                                                                    |
| `API_URL`      | Contenedor           | Se escribe al arrancar en `/config.js` y en la CSP: la misma imagen sirve para cualquier entorno |

Identidad visual: tokens de la paleta SmartPot en `src/styles/index.css`, tipografías Outfit e Inter servidas desde la
app y una paleta categórica de 8 colores validada para daltonismo en las comparativas.

## 7. Automated Testing Suite

`pnpm lint`, `pnpm typecheck`, `pnpm test` y `pnpm build`. Las 65 pruebas de Vitest y Testing Library cubren el cliente
HTTP y sus errores, la sesión, las validaciones, el ingreso, los componentes del cultivo, los switches, el asistente con
pronóstico, consejo de lugar y lo aprendido, la comparación de modelos, Aplicaciones y avisos por cultivo, el panel
general, la ilustración del cultivo (formas, especies, lugar, actuadores, clima y conexión), la Simulación en dos
partes, la creación real o virtual, la guía de conexión con la CA y los requisitos de SEO y PWA.

## 8. Production Operations

| Tarea               | Cómo                                                                                              |
|---------------------|---------------------------------------------------------------------------------------------------|
| Imagen              | `ghcr.io/smartpottech/smartpot-web`: nginx sin privilegios, solo lectura con `tmpfs`              |
| Despliegue          | Cada cambio en `main` pasa por el CI, publica la imagen y pide el despliegue central de `.github` |
| Íconos              | `pnpm icons` regenera los PNG de la PWA desde los SVG                                             |
| Caché del navegador | Una versión nueva del service worker reemplaza el app shell al recargar                           |
