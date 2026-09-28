# AI Builder - Architecture Documentation

## System Overview

```
┌─────────────────────────────────────────────────────────┐
│                   User Browser                          │
└────────────────────┬────────────────────────────────────┘
                     │
                     │ HTTP/REST
                     ↓
┌─────────────────────────────────────────────────────────┐
│             Next.js Frontend (Port 3000)                │
│  ┌──────────────────────────────────────────────────┐  │
│  │  React Flow - Visual Builder Canvas             │  │
│  │  - Node Editor                                   │  │
│  │  - Component Library                            │  │
│  │  - Real-time Preview                            │  │
│  └──────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────┐  │
│  │  State Management (Zustand)                     │  │
│  │  - Workflow State                               │  │
│  │  - UI State                                     │  │
│  └──────────────────────────────────────────────────┘  │
└────────────────────┬────────────────────────────────────┘
                     │
                     │ REST API
                     ↓
┌─────────────────────────────────────────────────────────┐
│          Express.js API Server (Port 3001)              │
│  ┌──────────────────────────────────────────────────┐  │
│  │  Authentication Routes                          │  │
│  │  - Login / Register                             │  │
│  │  - JWT Token Management                        │  │
│  └──────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────┐  │
│  │  Workflow Routes                                │  │
│  │  - CRUD Operations                              │  │
│  │  - Publish/Deploy                              │  │
│  └──────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────┐  │
│  │  Component Routes                               │  │
│  │  - Component Library Management                 │  │
│  │  - Schema Validation                            │  │
│  └──────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────┐  │
│  │  Export Routes                                  │  │
│  │  - Generate Standalone Apps                     │  │
│  │  - Docker Image Creation                        │  │
│  │  - Package Export                              │  │
│  └──────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────┐  │
│  │  AI Integration Service                         │  │
│  │  - OpenAI API Calls                             │  │
│  │  - Anthropic API Calls                          │  │
│  │  - Provider Management                         │  │
│  └──────────────────────────────────────────────────┘  │
└────────────────────┬────────────────────────────────────┘
                     │
                     │ Prisma ORM
                     ↓
┌─────────────────────────────────────────────────────────┐
│         PostgreSQL Database (Port 5432)                 │
│  ┌──────────────────────────────────────────────────┐  │
│  │  Users Table                                    │  │
│  │  - Authentication Data                         │  │
│  │  - User Preferences                            │  │
│  └──────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────┐  │
│  │  Workflows Table                                │  │
│  │  - Workflow Metadata                           │  │
│  │  - Publication Status                          │  │
│  └──────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────┐  │
│  │  Nodes & Edges Tables                          │  │
│  │  - Canvas Structure                            │  │
│  │  - Component Connections                      │  │
│  └──────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────┐  │
│  │  Components Table                               │  │
│  │  - Component Definitions                       │  │
│  │  - Configuration Schemas                       │  │
│  └──────────────────────────────────────────────────┘  │
│  ┌──────────────────────────────────────────────────┐  │
│  │  AppExports Table                               │  │
│  │  - Export History                              │  │
│  │  - File Storage Metadata                       │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

## Module Structure

### Frontend (`apps/web`)
- **Pages**: Next.js app router pages
- **Components**: Reusable React components
- **Hooks**: Custom React hooks for workflow logic
- **Lib**: Utilities and helpers
- **Store**: Zustand state management

### Backend (`apps/api`)
- **Routes**: Express route handlers
- **Services**: Business logic layer
- **Middleware**: Express middleware (auth, validation)
- **Utils**: Utility functions
- **Prisma**: Database schema and migrations

## Data Flow

### Creating a Workflow
1. User opens the canvas editor
2. Drags components from library
3. Components added to workflow state (Zustand)
4. User connects nodes with edges
5. On save: workflow sent to API
6. API stores nodes, edges, and components in database

### Exporting an App
1. User clicks "Export" button
2. Frontend sends workflow ID to API
3. API fetches workflow from database
4. Export service generates:
   - Node.js package (if selected)
   - Docker image (if selected)
   - ZIP archive (if selected)
5. Files uploaded to storage/generated
6. Download link returned to user

### Running a Generated App
1. User downloads exported app
2. For Node.js: Run `npm install && npm start`
3. For Docker: Run `docker build . && docker run -p 3000:3000 .`
4. Generated app executes workflow locally
5. Integrates with configured AI services

## Authentication Flow

1. User submits login credentials
2. API validates against database
3. Password verified with bcrypt
4. JWT token generated
5. Token stored in browser (httpOnly cookie)
6. Token included in all subsequent requests
7. Middleware verifies token on protected routes

## Export/Download Architecture

### Types of Exports

#### 1. Node.js Package
- Bundles workflow into executable app
- Includes all dependencies
- Can be run with `npm start`
- Supports environment variables

#### 2. Docker Container
- Generates Dockerfile
- Builds image with all dependencies
- Includes docker-compose.yml
- Ready to deploy anywhere

#### 3. ZIP Archive
- Source code package
- npm dependencies listed
- README with setup instructions
- Environment template included

### Export Process
1. Validate workflow completeness
2. Generate necessary files
3. Bundle dependencies
4. Create archive/container
5. Store with metadata
6. Generate download link (expires in 7 days)

## Security Considerations

1. **Authentication**: JWT tokens with expiration
2. **Password**: bcrypt hashing with salt
3. **API**: CORS configured, input validation with Zod
4. **Database**: Connection pooling, prepared statements
5. **Exports**: Temporary storage with expiration
6. **Secrets**: Environment variables not exposed in exports

## Scalability

### Current Architecture
- Monolithic with microservice-ready structure
- Single database for all data
- File storage for exports

### Future Scalability
- Separate microservices for export/execution
- Message queue for long-running tasks
- Distributed cache (Redis)
- S3/Cloud storage for exports
- Horizontal scaling of API servers
