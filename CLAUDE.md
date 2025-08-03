# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Setup and Installation
```bash
npm run setup  # Install dependencies and build everything
```

### Development
```bash
npm start          # Run the Electron app in development mode
npm run lint       # Run ESLint for code quality checks
npm run build      # Build both renderer and web components for production
npm run build:renderer  # Build only the renderer process (Electron UI)
npm run build:web  # Build only the web dashboard
npm run watch:renderer  # Watch mode for renderer development
```

### Packaging and Distribution
```bash
npm run package    # Create development build
npm run make       # Create distributable packages
npm run build:win  # Build Windows version
npm run publish    # Build and publish release
```

### Web Dashboard (pickleglass_web/)
```bash
cd pickleglass_web
npm run dev        # Start Next.js development server
npm run build      # Build Next.js production app
npm run start      # Start Next.js production server
npm run lint       # Run Next.js linting
```

## Architecture Overview

Glass is a cross-platform Electron application that provides AI-powered real-time meeting assistance and contextual AI capabilities. The architecture follows a modular, service-oriented design with strict separation of concerns.

### Core Architectural Principles

1. **Centralized Data Logic**: All data persistence logic is handled in the Electron Main Process. UI layers (renderer and web) cannot access data sources directly.
2. **Feature-Based Modularity**: Code is organized by feature in `src/features/` to promote encapsulation.
3. **Dual-Database Repositories**: Every user data repository has two implementations (SQLite for local, Firebase for cloud) with identical interfaces.
4. **AI Provider Abstraction**: AI model interactions use a Factory Pattern - new providers can be added by creating modules that conform to the base interface.
5. **Encryption by Default**: All sensitive user data is encrypted before being persisted to Firebase.

### Application Structure

```
src/
├── index.js                 # Main Electron process entry point
├── features/               # Feature-based modules
│   ├── ask/               # AI Q&A functionality
│   ├── listen/            # Audio capture and transcription
│   ├── settings/          # Application settings
│   ├── shortcuts/         # Keyboard shortcuts
│   └── common/            # Shared services and repositories
├── bridge/               # IPC communication bridges
├── window/               # Window management
└── ui/                   # UI components and views
```

### Data Architecture

The application uses a sophisticated **Repository Factory Pattern** that dynamically switches between SQLite and Firebase:

1. **Service Layer**: Contains business logic, calls repository methods
2. **Repository Factory**: Checks authentication status and selects appropriate repository
3. **Repository Implementation**: Either SQLite (local) or Firebase (cloud)
4. **UID Injection**: Factory automatically injects user ID for Firebase operations

### Web Dashboard Architecture

The web dashboard (`pickleglass_web/`) uses a three-tier architecture:

1. **Next.js Frontend**: React-based UI
2. **Node.js Backend**: Express server for API endpoints
3. **Electron Main Process**: Ultimate authority for all local data access

Web dashboard cannot access SQLite directly - it must communicate via IPC with the main process.

### Key Services and Components

- **authService**: Handles user authentication and session management
- **modelStateService**: Manages API keys and model states
- **databaseInitializer**: Sets up and manages SQLite database
- **ollamaService**: Handles local LLM operations
- **windowManager**: Manages Electron windows and positioning
- **featureBridge**: IPC bridge for feature-specific communication

### Development Guidelines

#### File Organization
- Place feature-specific code in `src/features/[feature]/`
- Place shared code in `src/features/common/`
- Use the Service-Repository pattern for data operations
- Maintain separation between UI logic and business logic

#### Repository Pattern
- Always create both SQLite and Firebase implementations
- Use the factory pattern in `index.js` files to select repositories
- Ensure both implementations expose identical interfaces

#### AI Provider Integration
- New AI providers should be added to `src/common/ai/providers/`
- Implement the standard interface defined in the base provider
- Register the provider in the factory

#### Security
- All Firebase data must be encrypted using `createEncryptedConverter`
- API keys are managed through `modelStateService`
- User authentication is handled through `authService`

#### Code Quality
- Run `npm run lint` before committing
- Follow the existing code style and patterns
- Use TypeScript where available (web dashboard)
- Maintain clear separation of concerns

### Common Development Tasks

#### Adding a New Feature
1. Create directory in `src/features/[feature]/`
2. Implement service layer for business logic
3. Create repository implementations (SQLite + Firebase)
4. Add repository factory in `index.js`
5. Implement UI components if needed
6. Add IPC handlers in `featureBridge.js` if needed

#### Adding a New AI Provider
1. Create provider module in `src/common/ai/providers/`
2. Implement standard interface methods
3. Register provider in `factory.js`
4. Add any necessary configuration options

#### Modifying Database Schema
1. Update schema definitions in appropriate repository files
2. Implement migration logic if needed
3. Test both SQLite and Firebase implementations

### Testing and Quality Assurance

- Ensure full production build works: `npm run build`
- Test both local (SQLite) and cloud (Firebase) modes
- Verify window positioning and shortcuts work correctly
- Test audio capture and transcription functionality
- Validate encryption/decryption of sensitive data

### Build Process

The build process involves:
1. Building the Next.js web dashboard (`pickleglass_web/`)
2. Building the Electron renderer process (`src/ui/`)
3. Packaging everything with Electron Builder

The application supports both development and production builds with appropriate configuration handling.