# Vue.js Single Page Application Template

## Purpose
This template guides the creation of modern Vue.js 3 applications using the Composition API, TypeScript, and Vue ecosystem best practices.

## Use Case
Use when building Vue.js applications with modern patterns, composables, and proper state management using Pinia or Vuex.

## Template

```
Create a Vue.js 3 application with the following requirements:

## Application Overview
[Describe your application's purpose and core functionality]

## Technical Stack
- Vue 3 with Composition API
- TypeScript for type safety
- State management: Pinia (recommended) or Vuex 4
- Router: Vue Router 4
- Build tool: Vite
- UI Framework: [Vuetify / Element Plus / Naive UI / PrimeVue]
- HTTP Client: Axios with composables

## Architecture Principles
1. Use Composition API for better code organization
2. Create reusable composables for shared logic
3. Implement proper component communication patterns
4. Use TypeScript strict mode
5. Follow Vue 3 style guide and best practices
6. Implement lazy loading for routes

## Project Structure
```
src/
├── components/          # Reusable components
│   ├── common/         # Shared UI components
│   └── features/       # Feature-specific components
├── composables/        # Reusable composition functions
├── views/              # Page-level components
├── stores/             # Pinia stores
├── router/             # Vue Router configuration
├── services/           # API services
├── utils/              # Utility functions
├── types/              # TypeScript types and interfaces
├── assets/             # Static assets
└── App.vue             # Root component
```

## Key Features to Implement
1. Authentication flow with route guards
2. Reactive data management with Pinia
3. Form handling with validation (VeeValidate)
4. Error handling with error boundaries
5. Loading states and transitions
6. Responsive design
7. Accessibility features (a11y)
8. Internationalization (Vue I18n)

## Best Practices
- Use `<script setup>` syntax for cleaner code
- Implement proper TypeScript types for props and emits
- Create composables for shared logic
- Use provide/inject for dependency injection
- Write unit tests with Vitest and Vue Test Utils
- Implement proper error handling
- Use Suspense for async components
- Follow Vue 3 naming conventions

## Testing Strategy
- Unit tests with Vitest and Vue Test Utils
- Component testing for complex interactions
- E2E tests with Playwright
- Test composables independently

## Performance Optimization
- Use v-once for static content
- Implement virtual scrolling for large lists
- Lazy load components and routes
- Use defineAsyncComponent for heavy components
- Optimize computed properties
- Monitor bundle size

## Security
- Sanitize v-html content
- Validate user inputs
- Implement proper authentication
- Use environment variables for sensitive data
- Follow security best practices
```

## Example Prompt

```
Create a Vue 3 e-commerce SPA with:

1. Product catalog with search and filters
2. Shopping cart functionality
3. User authentication
4. Order management
5. Admin dashboard

Use Vue 3 Composition API with TypeScript, Pinia for state management, and Tailwind CSS for styling. Implement proper form validation, error handling, and loading states. Include unit tests for stores and components.
```

## References
- [Vue 3 Documentation](https://vuejs.org/)
- [Pinia Documentation](https://pinia.vuejs.org/)
- [Vue Router](https://router.vuejs.org/)
- [Vite](https://vitejs.dev/)
- [Vue Test Utils](https://test-utils.vuejs.org/)
