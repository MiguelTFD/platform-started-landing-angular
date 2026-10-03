# platform-started-landing-angular

A static landing page built as a Single Page Application (SPA) with Angular 18. It follows strict high-performance, AOT (Ahead-Of-Time) compilation, and zero-bloat principles. Unnecessary abstractions and monolithic UI libraries are explicitly excluded, and there is no Server-Side Rendering (SSR).

## Table of Contents

- [Tech Stack](#tech-stack)
- [Getting Started](#getting-started)
- [NPM Scripts](#npm-scripts)
- [Project Structure](#project-structure)
- [Folder Structure Explanation](#folder-structure-explanation)
- [Code Generation](#code-generation)
- [Styling with Tailwind CSS](#styling-with-tailwind-css)
- [Coding Conventions](#coding-conventions)
- [Build and Deployment](#build-and-deployment)
- [Testing](#testing)
- [Troubleshooting](#troubleshooting)
- [Version Control Guidelines](#version-control-guidelines)

## Tech Stack

| Area               | Technology                                                                          |
| ------------------ | ----------------------------------------------------------------------------------- |
| Framework          | Angular 18 (standalone architecture, **no NgModules**)                              |
| Language           | TypeScript 5.5 (strict mode)                                                        |
| Styling            | Tailwind CSS 3.4, PostCSS, Autoprefixer (plain CSS, **no preprocessors like SCSS**) |
| State & reactivity | RxJS, Angular Signals                                                               |
| Build              | Angular `application` builder (esbuild) with the Vite dev server                    |
| Unit tests         | Karma + Jasmine                                                                     |
| Formatting         | Prettier                                                                            |

Explicitly excluded: SSR, Angular Material and any other monolithic UI library.

## Getting Started

### Prerequisites

- Node.js `^18.19.1 || ^20.11.1 || >=22.0.0` (required by Angular 18)
- npm (or your preferred package manager)
- Angular CLI 18:

```bash
npm install -g @angular/cli@18
ng version
```

### Install and Run

```bash
npm install
npm start          # same as: ng serve
```

The app will be available at `http://localhost:4200/` and reloads automatically when you change any source file (HMR through the Vite dev server).

**Useful flags:**

```bash
ng serve -o                  # open the browser automatically
ng serve --port 4300         # use a custom port
ng serve --host 0.0.0.0      # expose the server on your local network
```

## NPM Scripts

| Script          | Command                                        | Description                                                |
| --------------- | ---------------------------------------------- | ---------------------------------------------------------- |
| `npm start`     | `ng serve`                                     | Development server (development configuration by default)  |
| `npm run build` | `ng build`                                     | Production build (production is the default configuration) |
| `npm run watch` | `ng build --watch --configuration development` | Rebuild on file changes, unoptimized with source maps      |
| `npm test`      | `ng test`                                      | Unit tests with Karma and Jasmine                          |

## Project Structure

This project follows a strict separation of concerns using a standalone Angular architecture. All application code lives under `src/app/`.

```
.
├── public/                     <-- Static assets served directly (robots.txt, favicon, images/)
├── src/
│   ├── app/
│   │   ├── core/               <-- Technical infrastructure (Singletons)
│   │   │   ├── http/           <-- Interceptors and API client logic
│   │   │   └── services/       <-- Global state and cross-cutting services (e.g., SeoService)
│   │   │
│   │   ├── pages/              <-- Smart Components (Routed Views, lazy loaded)
│   │   │   ├── about/
│   │   │   ├── contact/
│   │   │   ├── home/
│   │   │   │   ├── components/ <-- Organisms strictly private to the Home view (e.g., Hero)
│   │   │   │   └── home.component.ts
│   │   │   └── our-services/
│   │   │
│   │   ├── shared/             <-- Reusable UI (used by 2+ pages or by the app shell)
│   │   │   ├── layouts/        <-- Page wrappers (App Shell components)
│   │   │   │   ├── footer/
│   │   │   │   └── navbar/
│   │   │   └── ui/             <-- Dumb components (Highly reusable, no state mutations)
│   │   │
│   │   ├── app.component.ts    <-- Root App Shell (template contains <router-outlet>)
│   │   ├── app.config.ts       <-- Global dependency injection providers (Router, HttpClient)
│   │   └── app.routes.ts       <-- Lazy-loaded routing table
│   │
│   ├── index.html              <-- HTML entry point (static meta tags for SEO)
│   ├── main.ts                 <-- Bootstrap execution entry point
│   └── styles.css              <-- Global stylesheet orchestrator (@tailwind directives)
│
├── angular.json                <-- CLI and esbuild workspace configuration
├── tailwind.config.js          <-- Tailwind JIT compiler config
└── tsconfig.json               <-- TypeScript compiler rules and path aliases (@/*)
```

## Folder Structure Explanation

### 1. `core/` (Technical Infrastructure)

Contains cross-cutting concerns that apply globally to the SPA. **No UI components belong here.**

- **`http/`**: HTTP interceptors (e.g., injecting auth tokens if an API is consumed) and base API configuration.
- **`services/`**: Root-provided singleton services. Global `Signal` state (such as a dark/light mode toggle) and SEO metadata handling live here.

### 2. `pages/` (Smart Components & Feature Boundaries)

Each directory represents a distinct, navigable page in the application.

- Components here are "smart components". They may inject services from `core/` to fetch data and orchestrate the view.
- **Lazy loading**: every page is loaded asynchronously with `loadComponent` in `app.routes.ts`.
- **Private components**: if a page (like `home/`) has a complex section that is _never_ used anywhere else (e.g., a large Hero banner), it must live in `pages/home/components/`. Do not pollute `shared/` with single-use organisms.

### 3. `shared/` (Reusable UI & Layouts)

Contains purely visual, stateless elements designed for maximum reusability. A component moves here only when two or more pages (or the app shell) use it.

- **`layouts/`**: structural components that wrap the application, such as `NavbarComponent` and `FooterComponent`.
- **`ui/`**: "dumb components". They receive data via inputs, emit events via outputs, and contain **no** business logic or HTTP calls.

> Simple buttons or badges must be built with Tailwind utility classes, not by creating redundant Angular components.

Dependency direction: `pages → shared`, `pages → core` and `shared → core` are allowed. `shared` and `core` must never import from `pages`.

### 4. `styles.css` (Global Styles)

Because the project relies on Tailwind CSS, the global CSS footprint is minimal and lives in a single file.

- **`src/styles.css`**: contains the `@tailwind base`, `components` and `utilities` directives.
- Custom CSS is strictly limited. Global overrides (e.g., applying the Gruvbox background to the `body` tag) use Tailwind's `@apply` inside an `@layer base` block.
- If the file grows too large, split it into partials and import them from `styles.css`.

### 5. `public/` (Static Assets)

Everything in `public/` is copied as-is to the root of the build output. Put `robots.txt`, the favicon and images here (e.g., `public/images/logo.svg`) and reference them with absolute paths (`/images/logo.svg`).

### 6. Root Configuration Files

- **`tailwind.config.js`**: defines the `content` globs used to tree-shake unused CSS.
- **`app.routes.ts`**: the central navigation table. Enforces code-splitting by importing components only when their route is visited.
- **`tsconfig.json`**: strict mode (`"strict": true`) plus the `@/*` path alias that maps to `src/app/*`, which avoids relative-path hell:

```json
{
  "compilerOptions": {
    "strict": true,
    "paths": { "@/*": ["./src/app/*"] }
  }
}
```

## Code Generation

The Angular CLI generates files under `src/app/`, which matches this project's structure. In Angular 18 components are **standalone by default**.

```bash
# Routed page (smart component)
ng g c pages/about

# Private component of a page
ng g c pages/home/components/hero

# Dumb reusable component
ng g c shared/ui/card

# Layout component
ng g c shared/layouts/navbar

# Global services and interceptors
ng g s core/services/seo
ng g interceptor core/http/auth
```

> Each component creates a `.ts`, `.html`, `.css` and `.spec.ts` file in its own folder.

**Common options:**

```bash
--skip-tests               # do not create the .spec.ts file
--inline-template          # keep the template inside the .ts file
--dry-run                  # preview the files without writing them
```

## Styling with Tailwind CSS

`src/styles.css` is registered in `angular.json` for both the build and test targets and holds the Tailwind directives:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

`tailwind.config.js` must scan every template and component file:

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ["./src/**/*.{html,ts}"],
  theme: { extend: {} },
  plugins: [],
};
```

Rules:

- Prefer utility classes in templates over custom CSS.
- Use `@apply` only inside `@layer base` for global element overrides.
- Component styles are capped by the production budgets (see [Build and Deployment](#build-and-deployment)).

## Coding Conventions

- Components are standalone. Do not create `NgModule`s.
- Selector prefix: `app-` (e.g., `app-navbar`).
- File naming: `name.component.ts`, `name.service.ts`, and so on, using kebab-case.
- Every page is lazy loaded in `app.routes.ts`:

```ts
export const routes: Routes = [
  {
    path: "",
    loadComponent: () => import("./pages/home/home.component").then((m) => m.HomeComponent),
  },
];
```

- Prefer Angular Signals for local and global UI state, and RxJS for asynchronous streams.
- Import across folders with the `@/*` alias instead of long relative paths:

```ts
import { SeoService } from "@/core/services/seo.service";
```

- Keep unit tests next to the code they test (`*.spec.ts`), as the CLI generates them.

## Build and Deployment

### Production Build

```bash
npm run build
```

> The production configuration is the default. It generates static HTML/JS/CSS in:

```
dist/platform-started-landing-angular/browser
```

**Other options:**

```bash
ng build --configuration development      # unoptimized build with source maps
ng build --base-href /landing/            # when served from a sub-path
```

### Build Budgets

The production build enforces size limits defined in `angular.json`:

| Budget              | Warning | Error |
| ------------------- | ------- | ----- |
| Initial bundle      | 500 kB  | 1 MB  |
| Any component style | 2 kB    | 4 kB  |

> If a build fails because of a budget, fix the cause (lazy load, remove unused code, use Tailwind utilities) instead of raising the limit.

### Serving with Nginx

Because this is an SPA, every unknown route must fall back to `index.html`:

```nginx
server {
    listen 80;
    server_name example.com;

    root /var/www/platform-started-landing-angular/browser;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # Cache static assets with hashed filenames
    location ~* \.(?:js|css|woff2?|png|jpg|jpeg|svg|ico)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

> Do not apply long-term caching to `index.html` itself, so new deployments are picked up immediately.

### SEO Without SSR

Since there is no SSR, meta tags are set on the client. `SeoService` (in `core/services/`) uses Angular's `Title` and `Meta` services to update the title and description on every route change. Keep in mind that crawlers that do not execute JavaScript will only see the static tags in `src/index.html`, so keep those accurate.

## Testing

```bash
npm test                                           # watch mode
ng test --watch=false --browsers=ChromeHeadless    # single run (CI)
```

> Linting is not configured in this project.

## Troubleshooting

```bash
ng cache clean                                         # clear the Angular build cache
rm -rf node_modules package-lock.json && npm install   # clean reinstall
```

## Version Control Guidelines

- Every new feature or fix must be implemented in a separate branch.
- Direct commits to the `main` branch are strictly forbidden.
- Commit messages must follow the conventional commits format (`type: description`) and be written in the imperative mood.

Example:

```bash
git commit -m "feat: add lazy loading to the our-services page"
```

After committing, push your branch to the remote repository:

```bash
git push origin <branch-name>
```

> Open a Pull Request on GitHub and assign the project owner as a reviewer.
