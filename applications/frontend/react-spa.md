# React Single Page Application (SPA) Template

## Purpose
This template provides a comprehensive prompt for building modern React single-page applications with best practices, proper state management, and scalable architecture.

## Use Case
Use this template when you need to create a new React-based frontend application or modernize an existing one. It covers component structure, state management, routing, and API integration.

## Template

```
Create a React single-page application with the following requirements:

## Application Overview
[Describe the purpose and main features of your application]

## Technical Requirements
- React 18+ with TypeScript
- State management: [Redux Toolkit / Zustand / Context API / Jotai]
- Routing: React Router v6
- API client: [Axios / React Query / TanStack Query]
- UI Framework: [Material-UI / Tailwind CSS / Chakra UI / Ant Design]
- Build tool: [Vite / Create React App / Next.js]

## Architecture Principles
1. Follow component-driven development
2. Implement proper separation of concerns (components, hooks, services, utils)
3. Use custom hooks for reusable logic
4. Implement error boundaries for error handling
5. Apply lazy loading for route-based code splitting
6. Use TypeScript strict mode for type safety

## Project Structure
```
src/
├── components/         # Reusable UI components
│   ├── common/        # Shared components (Button, Input, etc.)
│   └── features/      # Feature-specific components
├── hooks/             # Custom React hooks
├── pages/             # Route-level components
├── services/          # API services and data fetching
├── store/             # State management (if using Redux/Zustand)
├── utils/             # Utility functions
├── types/             # TypeScript type definitions
├── constants/         # Application constants
└── App.tsx            # Root component
```

## Key Features to Implement
1. Authentication and authorization flow
2. Protected routes
3. Form validation using [React Hook Form / Formik]
4. Loading states and skeleton screens
5. Error handling and user feedback
6. Responsive design for mobile and desktop
7. Accessibility (WCAG 2.1 AA compliance)
8. Performance optimization (memoization, virtualization)

## Best Practices
- Use functional components with hooks
- Implement proper PropTypes or TypeScript interfaces
- Follow the single responsibility principle
- Write unit tests with React Testing Library
- Use ESLint and Prettier for code consistency
- Implement CI/CD pipelines for automated testing and deployment
- Document components with Storybook (optional)

## Testing Strategy
- Unit tests for components and hooks (Jest + React Testing Library)
- Integration tests for user flows
- E2E tests for critical paths (Playwright / Cypress)
- Aim for >80% code coverage

## Performance Considerations
- Implement code splitting at route level
- Use React.memo for expensive components
- Optimize re-renders with useCallback and useMemo
- Lazy load images and heavy dependencies
- Monitor bundle size and performance metrics

## Security Guidelines
- Sanitize user inputs
- Implement CSRF protection
- Use secure HTTP-only cookies for tokens
- Validate data on both client and server
- Follow OWASP security guidelines
```

## Example Prompt

```
Create a React SPA for a task management application with the following features:

1. User authentication (login/register)
2. Dashboard showing task lists
3. CRUD operations for tasks
4. Task filtering and sorting
5. Drag-and-drop task prioritization

Use React 18 with TypeScript, Redux Toolkit for state management, React Router for navigation, and Tailwind CSS for styling. Implement proper error handling, loading states, and follow the component structure outlined above. Include unit tests for key components.
```

## References
- [React Documentation](https://react.dev/)
- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)
- [Redux Toolkit](https://redux-toolkit.js.org/)
- [React Testing Library](https://testing-library.com/react)
- [Web.dev Best Practices](https://web.dev/)
