# Portfolio de proyectos · Andrea Servin

Sitio web que reúne mis proyectos de software, con su descripción, tecnologías, demo y código. Se puede filtrar por tecnología, categoría y estado, y buscar por texto. Cada proyecto tiene su propia página de detalle.

**Sitio:** https://andrea-servinm.github.io/portfolio/
**Presentación:** https://andrea-servinm.github.io

> **Estado:** en desarrollo. Este repositorio sigue el documento de requisitos (SRS) del proyecto y avanza por fases. El detalle está en la sección [Hoja de ruta](#hoja-de-ruta).

## Qué hace

- Listado de proyectos en tarjetas, del más reciente al más antiguo.
- Filtros por tecnología, categoría y estado, combinables con un buscador de texto.
- Filtros reflejados en la URL, para compartir una vista concreta.
- Página de detalle por proyecto: problema, rol, decisiones técnicas, tecnologías y enlaces.
- Tema claro, oscuro o del sistema, con la preferencia guardada.
- Idioma a seleccionar, por defecto Español.
- Accesible (WCAG 2.1 AA), responsivo y con HTML prerenderizado por ruta para SEO.

## Tecnologías

- **TypeScript**, **React** y **Vite**
- **React Router** para las rutas
- **JSON** como única fuente de datos: sin servidor ni base de datos
- **Vitest** y **Testing Library** para las pruebas
- **GitHub Actions** y **GitHub Pages** para el despliegue

## Cómo funcionan los datos

Todos los proyectos viven en un solo archivo, `src/data/projects.json`. Los tipos de TypeScript y la validación salen del mismo esquema, así que el dato, el tipo y la validación no se contradicen. Si el JSON es inválido, el build falla e indica qué proyecto y qué campo tienen el problema.

Cada proyecto tiene esta forma:

| Campo | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `slug` | texto | Sí | Identificador único en la URL, en minúsculas y con guiones |
| `title` | texto | Sí | Hasta 80 caracteres |
| `summary` | texto | Sí | Hasta 200 caracteres |
| `description` | texto | No | Descripción larga para la página de detalle |
| `role` | texto | No | Qué hice en el proyecto |
| `problem` | texto | No | Qué problema resuelve |
| `decisions` | lista de textos | No | Decisiones técnicas relevantes |
| `tags` | lista de textos | Sí | Al menos una tecnología |
| `category` | opción | Sí | `web`, `bot`, `landing`, `api` u `otro` |
| `status` | opción | Sí | `publicado`, `en-desarrollo` o `archivado` |
| `date` | texto `AAAA-MM` | Sí | Define el orden por defecto |
| `featured` | booleano | No | Si es verdadero, el proyecto se fija primero |
| `links.demo` | URL https | No | Enlace a la demo |
| `links.repo` | URL https | No | Enlace al repositorio |

## Cómo agregar un proyecto

1. Se agrega un objeto nuevo a `src/data/projects.json`.
3. Se sube a la rama `dev` y, cuando esté listo, se une a `main` con un pull request (ver [Flujo de trabajo con git](#flujo-de-trabajo-con-git)). El sitio se actualiza solo en uno o dos minutos.

No hace falta tocar ningún componente.

## Estructura

```
portfolio/
├── src/
│   ├── routes/            # listado, detalle de proyecto y 404
│   ├── components/        # tarjeta, filtros, buscador, selector de tema e idioma
│   ├── lib/               # filtrar, buscar y ordenar (con pruebas)
│   ├── data/
│   │   ├── projects.json  # los proyectos
│   │   └── schema.ts      # esquema de validación y tipos
│   └── styles/tokens.css  # colores y tipografías
├── .github/workflows/     # despliegue automático
├── SRS.md
└── README.md
```
## Comandos utilizados

| Comando | Qué hace |
|---|---|
| `npm run dev` | Servidor de desarrollo con recarga automática |
| `npm run build` | Valida los datos y genera el sitio estático |
| `npm run preview` | Sirve el resultado del build para revisarlo |
| `npm test` | Ejecuta las pruebas |
| `npm run lint` | Revisa el código con ESLint |

## Flujo de trabajo con git

| Rama | Para qué sirve |
|---|---|
| `main` | Lo que está publicado. Siempre debe funcionar, y cada cambio que llega aquí se despliega solo. |
| `dev` | Rama de trabajo diaria. Reúne los cambios y se prueba antes de publicar. |

No hay una rama `test` aparte: las pruebas, la revisión de código y el build corren automáticamente en cada pull request, y eso hace el trabajo de una rama de pruebas sin mantener otra copia del proyecto.

**Cómo se trabaja**

1. Empieza siempre desde `dev` actualizada.
2. Haz commits pequeños y con mensaje claro.
3. Une la rama de funcionalidad a `dev` con un pull request. Si las comprobaciones pasan, se integra.
4. Cuando `dev` está estable, abre un pull request de `dev` hacia `main`. Al unirlo, GitHub Actions publica el sitio.
5. Al terminar cada fase de la hoja de ruta, marca la versión con una etiqueta: `git tag v0.1.0 && git push --tags`.

**Mensajes de commit**

Se usa el formato de [Conventional Commits](https://www.conventionalcommits.org/es/): `tipo: descripción corta`, con los tipos `feat`, `fix`, `docs`, `style`, `refactor`, `test` y `chore`.

```
feat: agregar filtros por tecnología
fix: corregir la ruta base de las imágenes
docs: actualizar el README
```

**Protección de `main`**

En Settings → Branches se activa una regla para `main` que exige pasar por pull request y que las comprobaciones (lint, pruebas y build) estén en verde. Así nada se publica sin haber sido revisado por la automatización.

## Despliegue

El sitio se publica en GitHub Pages con GitHub Actions.

- **En cada pull request** hacia `dev` o `main`: se revisa el código, se ejecutan las pruebas, se valida el JSON y se construye el sitio, pero no se publica nada.
- **Al llegar un cambio a `main`:** se repiten esos pasos y, si todo pasa, el sitio se publica.

Si algún paso falla, no se publica nada.

Como el sitio vive en `/portfolio/`, la ruta base está configurada en el proyecto, y las rutas de los enlaces se construyen a partir de ella.

## Hoja de ruta

- [ ] **Fase 1. Base:** proyecto, variables de diseño, esquema y datos, selector de tema e idiomas, encabezado y pie
- [ ] **Fase 2. Listado:** tarjetas, filtros, buscador, orden y filtros en la URL
- [ ] **Fase 3. Detalle:** páginas de proyecto, navegación anterior y siguiente, 404 y metadatos
- [ ] **Fase 4. Calidad:** pruebas, accesibilidad, Lighthouse
- [ ] **Fase 5. Publicación:** despliegue automático y enlace desde la presentación
- [ ] **Fase 6. Opcional:** datos del repositorio desde la API pública de GitHub

## Contacto

- Correo: andreaservin.ing@gmail.com
- LinkedIn: https://www.linkedin.com/in/andreaservin-ing
- GitHub: https://github.com/andrea-servinm
