# SRS: Portfolio de proyectos de Andrea Servín

Especificación de requisitos de software · Versión 0.1 (borrador) · 29 de octubre de 2026 · Autora: Ing. Andrea Servín

## 1. Introducción

### 1.1 Propósito

Este documento define los requisitos del **Portfolio de proyectos**, una aplicación web que reúne los proyectos de software de Andrea Servín con su descripción, tecnologías, capturas, demo y código. Sirve de referencia para diseñar, construir y probar el sistema, y para decidir qué entra y qué no en cada versión.

### 1.2 Alcance

El sistema es un sitio web estático, sin servidor propio, publicado en GitHub Pages en `https://andrea-servinm.github.io/portfolio/`. Complementa la página de presentación (`https://andrea-servinm.github.io/`), cuyo botón “Ver más proyectos” apuntará a este sitio.

**Dentro del alcance:** listado de proyectos, filtros, búsqueda, páginas de detalle, tema claro y oscuro, varios idiomas y despliegue automático.

**Fuera del alcance:** servidor, base de datos, panel de administración, cuentas de usuario, blog, formulario de contacto con servidor, analítica, SEO (se trabaja en la página de presentación). Los datos vienen siempre de archivos JSON.

### 1.3 Objetivos

- Mostrar el trabajo de forma ordenada a reclutadores y clientes potenciales.
- Demostrar dominio de React, TypeScript, enrutamiento, pruebas, accesibilidad y despliegue automatizado.
- Permitir agregar un proyecto editando un solo archivo de datos.

### 1.4 Definiciones y siglas

| Término | Significado |
| --- | --- |
| SRS | Software Requirements Specification, este documento |
| SPA | Aplicación de una sola página, que navega sin recargar |
| Prerender | Generar el HTML de cada ruta durante el build, para servirlo como archivo estático |
| Slug | Identificador de un proyecto en la URL, por ejemplo `portfolio` |
| WCAG | Pautas de accesibilidad web del W3C |
| MoSCoW | Prioridades: Must (imprescindible), Should (importante), Could (deseable) |
| LCP | Largest Contentful Paint, tiempo hasta mostrar el contenido principal |

### 1.5 Referencias

- Página de presentación: https://andrea-servinm.github.io
- ISO/IEC/IEEE 29148, como base de la estructura de este documento
- Documentación de React Router v7 (modo framework, SPA y prerender)
- WCAG 2.1, nivel AA

## 2. Descripción general

### 2.1 Perspectiva del producto

El sitio es independiente y forma parte de un recorrido de tres pasos: la presentación (quién soy), el portfolio (qué he construido) y, por último, cada demo y repositorio (cómo funciona). No depende de ningún servicio propio: todo el contenido se genera en el build a partir de archivos del repositorio.

```
Presentación  ──►  Portfolio (este sistema)  ──►  Demo / Código de cada proyecto
(landing)          listado · filtros · detalle       GitHub Pages / GitHub
```

### 2.2 Usuarios

| Usuario | Qué busca | Conocimiento técnico |
| --- | --- | --- |
| Reclutador/a | Ver rápido el stack y la calidad del trabajo | Medio |
| Cliente potencial | Entender qué tipo de software puede recibir y cómo contactar | Bajo |
| Colega desarrollador/a | Revisar decisiones técnicas y acceder al código | Alto |
| Andrea (administradora) | Agregar y editar proyectos con poco esfuerzo | Alto |

### 2.3 Funciones principales

1. Listado de proyectos con filtros por tecnología, categoría y estado, y búsqueda de texto.
2. Página de detalle por proyecto, con problema, rol, decisiones técnicas y enlaces.
3. Selector de tema claro, oscuro o del sistema.
4. Contenido administrado desde un archivo de datos validado.
5. Selector de idioma, por defecto Español.
6. Publicación automática en cada cambio.

### 2.4 Restricciones

- Sin servidor ni base de datos: todos los datos viven en archivos JSON del repositorio; costo de hosting cero.
- Stack: TypeScript, React, Vite y React Router v7, en modo framework con `ssr: false` y prerender de las rutas.
- Publicación en GitHub Pages, con ruta base `/portfolio/`.
- Diseño coherente con la presentación: mismas variables de color, tipografías (Bricolage Grotesque e Instrument Sans) y modo oscuro.
- Proyecto desarrollado y mantenido por una sola persona.

### 2.5 Supuestos y dependencias

- GitHub Pages y GitHub Actions están disponibles y gratuitos para el repositorio.
- Las demos y repositorios enlazados siguen publicados.

## 3. Requisitos funcionales

| ID | Requisito | Prioridad |
| --- | --- | --- |
| RF-01 | El sistema muestra el listado de proyectos como tarjetas con título, resumen, etiquetas de tecnología, estado y enlaces. | Must |
| RF-02 | El orden por defecto es del más reciente al más antiguo según la fecha del proyecto; los proyectos destacados pueden fijarse primero. | Must |
| RF-03 | Se puede filtrar por tecnología (selección múltiple), categoría y estado. | Must |
| RF-04 | Existe un buscador de texto sobre título, resumen y etiquetas, sin distinguir mayúsculas ni acentos. | Must |
| RF-05 | Los filtros y el buscador se combinan; se muestra el número de resultados, un mensaje cuando no hay coincidencias y un botón para limpiar. | Must |
| RF-06 | El estado de filtros y búsqueda se refleja en la URL mediante parámetros de consulta, para poder compartir la vista. | Should |
| RF-07 | Cada proyecto tiene una página en `/proyecto/:slug` con problema, rol, decisiones técnicas, tecnologías y enlaces a demo y código. | Must |
| RF-08 | La página de detalle ofrece navegación al proyecto anterior y al siguiente. | Should |
| RF-9 | Una ruta inexistente muestra una página 404 con enlace de regreso. | Must |
| RF-10 | Hay un selector de tema claro, oscuro y sistema, y la preferencia se guarda entre visitas. | Must |
| RF-11 | Hay un selector de idiomas, por defecto Español. | Must |
| RF-12 | Encabezado y pie incluyen el enlace de regreso a la presentación y los enlaces a GitHub, LinkedIn y correo. | Must |
| RF-13 | Cada página define su propio título, visible en la pestaña del navegador. | Must |
| RF-14 | Agregar un proyecto requiere editar solo `projects.json`, sin tocar componentes. | Must |
| RF-15 | El JSON se valida contra un esquema durante el build; si es inválido, el build falla con un mensaje que indica el proyecto y el campo. | Must |
| RF-16 | Mostrar datos del repositorio desde la API pública de GitHub (estrellas, lenguajes, última actualización), con alternativa si la consulta falla. | Could |

## 4. Requisitos de datos

Los proyectos se almacenan en `src/data/projects.json`. Los tipos de TypeScript se derivan del mismo esquema de validación, de modo que el dato, el tipo y la validación nunca se contradicen.

| Campo | Tipo | Obligatorio | Regla |
| --- | --- | --- | --- |
| `slug` | texto | Sí | Único, en minúsculas y con guiones |
| `title` | texto | Sí | Hasta 80 caracteres |
| `summary` | texto | Sí | Hasta 200 caracteres |
| `description` | texto | No | Descripción larga para la página de detalle |
| `role` | texto | No | Qué hizo la autora en el proyecto |
| `problem` | texto | No | Qué problema resuelve |
| `decisions` | lista de textos | No | Decisiones técnicas relevantes |
| `tags` | lista de textos | Sí | Al menos una tecnología |
| `category` | opción | Sí | `web`, `bot`, `landing`, `api` u `otro` |
| `status` | opción | Sí | `publicado`, `en-desarrollo` o `archivado` |
| `date` | texto AAAA-MM | Sí | Define el orden por defecto |
| `featured` | verdadero o falso | No | Si es verdadero, se fija primero |
| `links.demo` | URL | No | Debe ser https |
| `links.repo` | URL | No | Debe ser https |

## 5. Requisitos no funcionales

| ID | Categoría | Requisito | Cómo se mide |
| --- | --- | --- | --- |
| RNF-01 | Rendimiento | Puntaje mínimo de 90 en Rendimiento, Accesibilidad, Buenas prácticas | Lighthouse en móvil |
| RNF-02 | Rendimiento | LCP menor a 2,5 s y JavaScript inicial menor a 200 KB comprimido | Lighthouse y reporte del build |
| RNF-03 | Accesibilidad | Cumplir WCAG 2.1 AA: uso completo con teclado, foco visible, contraste mínimo 4,5:1 y enlace para saltar al contenido | Auditoría con axe y prueba manual |
| RNF-04 | Accesibilidad | Respetar la preferencia de movimiento reducido | Prueba manual |
| RNF-05 | Responsivo | Funcionar sin desplazamiento horizontal desde 320 px hasta 1920 px | Pruebas en navegador |
| RNF-06 | Compatibilidad | Últimas dos versiones de Chrome, Firefox, Safari y Edge | Pruebas manuales |
| RNF-07 | Mantenibilidad | TypeScript en modo estricto, ESLint y Prettier sin errores | Verificación en CI |
| RNF-8 | Calidad | Cobertura mínima de 70 % en la lógica de filtros, búsqueda y ordenamiento | Reporte de Vitest |
| RNF-9 | Seguridad y privacidad | Sin cookies de seguimiento ni datos personales; enlaces externos con `rel=noopener noreferrer`; dependencias auditadas | `npm audit` y Dependabot |
| RNF-10 | Despliegue | Publicación automática al subir a `main`; el build falla si fallan lint, pruebas o validación del JSON | GitHub Actions |
| RNF-11 | Idioma | Contenido de la interfaz en español (es-PY) por defecto | Revisión editorial |

## 6. Interfaces externas

### 6.1 Interfaz de usuario

| Ruta | Pantalla |
| --- | --- |
| `/` | Listado con filtros y buscador |
| `/proyecto/:slug` | Detalle del proyecto |
| `*` | Página 404 |

### 6.2 Interfaces de software

| Servicio | Uso | Obligatorio |
| --- | --- | --- |
| GitHub Pages | Hosting estático | Sí |
| GitHub Actions | Build y despliegue | Sí |
| Google Fonts | Tipografías | Sí, con tipografías de respaldo |
| API REST pública de GitHub | Datos de repositorios (RF-16) | No |

## 7. Casos de uso y criterios de aceptación

### 7.1 Casos de uso

- **CU-01. Filtrar por tecnología:** un reclutador entra al portfolio, selecciona la etiqueta React y ve solo los proyectos que la usan.
- **CU-02. Revisar un proyecto y contactar:** un cliente abre el detalle de un proyecto, lee el problema que resolvía, prueba la demo y vuelve a la presentación para escribir a Andrea.
- **CU-03. Agregar un proyecto:** Andrea añade un objeto a `projects.json`, sube el cambio y, en pocos minutos, el proyecto aparece en el sitio.

### 7.2 Criterios de aceptación

- **RF-03 y RF-05:** dado que hay proyectos con distintas etiquetas, cuando se seleccionan dos etiquetas y se escribe un texto, entonces solo se muestran los proyectos que cumplen todas las condiciones y se indica cuántos son.
- **RF-05:** cuando ningún proyecto coincide, entonces aparece un mensaje claro y un botón que restablece los filtros.
- **RF-07:** dado un slug existente, cuando se abre su dirección directamente o se recarga la página, entonces se muestra el detalle sin errores.
- **RF-14:** dado un nuevo objeto válido en `projects.json`, cuando se hace el build, entonces el proyecto aparece en el listado y tiene su página sin modificar ningún componente.
- **RF-15:** dado un objeto con un campo faltante o un slug repetido, cuando se hace el build, entonces el proceso falla e indica qué proyecto y qué campo tienen el problema.

## 8. Arquitectura y datos

### 8.1 Visión general

El sitio se construye con Vite y React Router v7 en modo framework. Con `ssr: false` no hay servidor en ejecución, y con la lista `prerender` se genera un HTML por cada ruta, incluidas las de cada proyecto, que se leen del JSON. El resultado es una carpeta de archivos estáticos que GitHub Pages sirve sin configuración adicional.

```
projects.json ──► validación (esquema) ──► build (Vite + React Router)
                                              │
                                              ▼
                                   HTML + JS + CSS estáticos
                                              │
                                              ▼
                                        GitHub Pages
```

### 8.2 Estructura de carpetas propuesta

```
portfolio/
├── app/
│   ├── routes/            # home, proyecto.$slug, 404
│   ├── components/        # ProjectCard, Filters, SearchBox, ThemeToggle, LanguageToggle
│   ├── lib/               # filtrar, buscar, ordenar (con pruebas)
│   ├── data/
│   │   ├── projects.json
│   │   └── schema.ts      # esquema de validación y tipos
│   └── styles/tokens.css  # variables de color y tipografía
├── react-router.config.ts
└── .github/workflows/deploy.yml
```

### 8.3 Datos: siempre JSON

Todos los datos del sitio viven en archivos JSON dentro del repositorio, y así seguirán: no habrá servidor, base de datos ni API en ninguna versión. El JSON se versiona con git, se valida en el build y no cuesta nada de alojar. El contenido crece por proyectos agregados, no por usuarios, así que un archivo basta.

## 9. Plan por fases y trazabilidad

| Fase | Entregable | Requisitos que cubre |
| --- | --- | --- |
| F1. Base | Proyecto Vite y React Router con TypeScript, variables de diseño, esquema y `projects.json`, selector de tema e idioma, encabezado y pie | RF-10, RF-11, RF-12, RF-14, RF-15 |
| F2. Listado | Tarjetas, filtros, buscador, orden y parámetros en la URL | RF-01 a RF-06 |
| F3. Detalle | Páginas de proyecto, navegación anterior y siguiente, 404 y metadatos | RF-07 a RF-9, RF-13 |
| F4. Calidad | Pruebas, auditoría de accesibilidad, Lighthouse | RNF-01 a RNF-10 |
| F5. Publicación | GitHub Actions, ruta base, README con capturas, enlace desde la presentación | RNF-11, RNF-12 |
| F6. Opcional | Datos desde la API de GitHub | RF-16 |

## 10. Riesgos

| Riesgo | Impacto | Mitigación |
| --- | --- | --- |
| Rutas que fallan al recargar en GitHub Pages | Alto | Prerender de todas las rutas y página 404 de respaldo; prueba en el despliegue de F5 |
| Ruta base incorrecta que rompe estilos e imágenes | Medio | Configurar la base una sola vez y probar el build local con esa ruta |
| Contenido desactualizado o enlaces rotos | Medio | Revisión periódica y comprobación de enlaces en CI |
| Ampliar el alcance antes de terminar la versión 1 | Medio | Respetar este documento y mover las ideas nuevas a la sección 11 |


