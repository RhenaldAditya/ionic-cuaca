# Weather Forecast App (Ionic Cuaca)

A modern, responsive weather forecast mobile web application built with Ionic Framework 8, Vue 3, TypeScript, and Vite. Displays real-time and hourly weather forecasts for Jakarta utilizing the Open-Meteo REST API.

## Features

- Real-time hourly weather forecasting for Jakarta (latitude -6.2, longitude 106.8)
- Dynamic temperature unit parsing and display (°C)
- Localized Indonesian timestamp formatting (`id-ID`) for measurement dates and hours
- Manual data refresh action via navigation bar button with visual feedback
- Smooth loading states with native Ionic crescent spinners
- Resilient error handling with user-friendly error banners
- Mobile-first responsive UI built with Ionic design system components
- Automatic Dark/Light mode theming matching device system preferences
- Cross-platform ready architecture with Capacitor 7

## Key Specifications & Constraints

- Weather data source: Open-Meteo Weather Forecast API (`https://api.open-meteo.com/v1/forecast`)
- Coordinates: Jakarta (-6.2° S, 106.8° E) with automatic timezone resolution
- Hourly measurement data mapped into typed structures (`ForecastItem` & `RawWeatherData`)
- Network fallback: graceful error catching with UI alert messages
- Null-safe temperature rendering (`null` values handled with fallback display)

## Installation

### Prerequisites

- Node.js 18 or later
- npm or yarn

### Setup Steps

1. Navigate to the project directory:

```bash
cd ionic-cuaca
```

2. Install dependencies:

```bash
npm install
```

## Running the Project

### Development Server

Start the development server with Vite hot module reloading:

```bash
npm run dev
```

To automatically open in your default browser:

```bash
npm run dev -- --open
```

The application will be available at `http://localhost:5173` (default Vite port).

Alternatively, using the Ionic CLI:

```bash
npx ionic serve
```

### Production Build

Build and bundle the application for production:

```bash
npm run build
```

The optimized web assets will be generated in the `dist/` directory.

### Preview Production Build

Preview the production build locally:

```bash
npm run preview
```

### Run Tests

Execute unit tests with Vitest:

```bash
npm run test:unit
```

Execute end-to-end tests with Cypress:

```bash
npm run test:e2e
```

Run linter checks:

```bash
npm run lint
```

## Project Structure

```
ionic-cuaca/
├── index.html                 # Main HTML entry point
├── package.json               # Project scripts and dependencies
├── tsconfig.json              # TypeScript root configuration
├── tsconfig.node.json         # TypeScript configuration for Node/Vite
├── vite.config.ts             # Vite build and plugin configurations
├── capacitor.config.ts        # Capacitor mobile runtime configuration
├── ionic.config.json          # Ionic CLI project configuration
├── cypress.config.ts          # Cypress E2E testing configuration
├── public/                    # Static public assets
├── src/
│   ├── main.ts                # Application bootstrap and Ionic CSS registration
│   ├── App.vue                # Root Ionic Vue component with router outlet
│   ├── vite-env.d.ts          # Vite client type definitions
│   ├── router/
│   │   └── index.ts           # Vue Router route definitions
│   ├── theme/
│   │   └── variables.css      # Ionic CSS custom properties and color palettes
│   └── views/
│       └── HomePage.vue       # Main weather view, API fetching, and UI templates
└── tests/
    ├── e2e/                   # Cypress end-to-end test suites
    └── unit/                  # Vitest unit test suites
```

## Architecture

The application follows a reactive, component-driven data flow powered by Vue 3 Composition API and Ionic UI components:

```
[ Open-Meteo REST API ]
         │ (HTTP GET)
         ▼
[ fetchWeatherData() in HomePage.vue ]
         │
    ┌────┴──────────────────────────┐
    ▼                               ▼
(Raw JSON Response)       (Catch Error Block)
    │                               │
    ▼                               ▼
[ Data Mapping / Formatting ]     [ error Ref State ]
    │                               │
    ▼                               ▼
[ forecasts & unit State ]        [ Error Banner UI ]
    │
    ▼
[ Ionic Template: IonList & IonItem ]
```

### Core Layers

**HomePage.vue** - Weather view & orchestrator
- Manages reactive state (`forecasts`, `unit`, `loading`, `error`)
- Fetches weather telemetry from the Open-Meteo REST API
- Parses raw hourly timestamp strings to localized Indonesian locale (`id-ID`)
- Renders mobile-friendly list items with `IonPage`, `IonHeader`, `IonToolbar`, `IonList`, and `IonItem`

**main.ts** - Bootstrap & theming entry point
- Initializes Vue application instance
- Registers `IonicVue` plugin and router
- Injects standard Ionic typography, structure, utilities, and system dark mode CSS

**router/index.ts** - Navigation setup
- Configures client-side routing with `@ionic/vue-router`
- Defines default redirect to `/home`

**capacitor.config.ts** - Native mobile configuration
- Configures application ID (`io.ionic.starter`) and target web directory for Android / iOS compilation

## Technologies Used

- **Framework**: [Ionic Vue 8](https://ionicframework.com/docs/vue/overview) & [Vue 3](https://vuejs.org/) (Composition API with `<script setup>`)
- **Language**: [TypeScript](https://www.typescriptlang.org/) (strict mode)
- **Build Tool / Dev Server**: [Vite 5](https://vitejs.dev/)
- **Mobile Runtime Bridge**: [Capacitor 7](https://capacitorjs.com/)
- **Icons**: [Ionicons 7](https://ionic.io/ionicons)
- **External Data Source**: [Open-Meteo API](https://open-meteo.com/)
- **Testing**: [Vitest](https://vitest.dev/) (Unit) & [Cypress](https://www.cypress.io/) (E2E)
- **Code Quality**: [ESLint](https://eslint.org/) with Vue & TypeScript rules

## Development Notes

### Code Style

- Single-File Components (`.vue`) utilizing `<script setup lang="ts">`
- Interfaces defined with PascalCase (`ForecastItem`, `RawWeatherData`)
- CamelCase naming convention for variables, refs, and functions (`fetchWeatherData`, `processedData`)
- Scoped CSS (`<style scoped>`) for view-specific layout adjustments

### Testing Strategy

- Unit tests run under Vitest with `@vue/test-utils` for testing component mounting and rendering
- End-to-end UI tests run with Cypress against live browser sessions
- Type validation checks automated via `vue-tsc` during production build

### Mobile Testing Tip

When testing in desktop browsers:
1. Open Developer Tools (**F12** or **Ctrl + Shift + I**)
2. Toggle device emulation mode (**Ctrl + Shift + M**)
3. Select a mobile preset (e.g., *iPhone 14*, *Pixel 7*) to preview the native mobile layout

## Accessibility & UI Standards

- Semantic Ionic elements (`<ion-page>`, `<ion-content>`, `<ion-list>`, `<ion-item>`)
- Accessible touch targets for buttons and interactive controls
- High-contrast visual cues for loading spinners and error states
- Adaptive color palettes honoring device dark mode settings (`@ionic/vue/css/palettes/dark.system.css`)

## Browser Compatibility

- Google Chrome / Chromium-based browsers (Edge, Opera, Brave) 91+
- Mozilla Firefox 89+
- Apple Safari (iOS & macOS) 14+
- Modern mobile WebViews (Android WebView & WKWebView)

## Troubleshooting

### Failed to Fetch API Data

If an error banner "Gagal mengambil data dari API" appears:
- Verify that your device has an active internet connection
- Ensure `https://api.open-meteo.com` is accessible and not blocked by firewall/proxy
- Click the refresh button in the toolbar to retry

### Missing Dependencies

If build or serve fails with module not found errors:
- Ensure you have executed `npm install` inside the project directory
- If errors persist, delete `node_modules` and run `npm install` again

### Port Already in Use

If port 5173 is already occupied:
- Vite will automatically attempt the next available port (e.g., 5174)
- You can manually specify a custom port: `npx vite --port 3000`

## Author & Academic Information

- **Student Name**: Rhenald Aditya BP
- **Student ID (NIM)**: 048907153
- **Project**: Prakiraan Cuaca Jakarta (Ionic Vue Mobile Application)

## License

This project is created for educational and academic coursework purposes.
