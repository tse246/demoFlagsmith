# CMX: Claim Motor X - Development Blueprint

## Overview
This document outlines the architecture, guidelines, and requirements for the **CMX (Claim Motor X)** project. It serves as the primary reference for Claude Code and other AI coding assistants to generate consistent, high-quality code that follows established patterns and best practices.

**IMPORTANT FOR AI ASSISTANTS:**
- Always follow the architectural layers strictly: `Controller → Handler → Manager → Service`
- Never bypass validation requirements or architectural constraints
- All generated code must adhere to the testing and quality standards specified
- Reference this document for any architectural decisions or implementation patterns

---

## Tech Stack

### **Frontend Tech Stack**
- **Angular (Latest Version)**: Frontend framework for creating dynamic, responsive web applications.  
- **Flagsmith**: Feature flag management for controlled rollouts and A/B testing.

### **Backend Tech Stack**
- **.NET 8**: Backend framework for building scalable, high-performance APIs.  
- **SQL Server**: Relational database for secure and efficient data storage.  
- **MediateR**: Library for implementing the mediator pattern to ensure clean separation of concerns.  
- **FluentValidation**: Library for request validation using `AbstractValidator` classes.  
- **Xunit**: Unit testing framework for .NET applications.  
- **NSubstitute**: Mocking library for creating test doubles in unit tests.  
- **Flagsmith**: Feature flag management for controlled rollouts and A/B testing.  
- **EAGLE Logger**: Logging and monitoring tool for tracking system performance and debugging.  
- **N-Tier Architecture**: Modular design with distinct layers (Presentation, Business Logic, Data Access).  

---

## Directory Structure

### Backend Structure Template
```plaintext
CMX/
 ├─ src/
 │ ├─ API/
 │ │ ├─ Controllers/
 │ │ └─ Middlewares/
 │ ├─ Application/
 │ │ ├─ Handlers/
 │ │ └─ Validators/
 │ ├─ Domain/
 │ │ ├─ Entities/
 │ │ └─ Interfaces/
 │ ├─ Infrastructure/
 │ │ ├─ Services/
 │ │ └─ Persistence/
 │ └─ BuildingBlocks/
 │   ├─ Logging/
 │   ├─ FeatureFlags/
 │   └─ Common/
 └─ tests/
     ├─ UnitTests/
     └─ IntegrationTests/
```

### Frontend Structure Template
```plaintext
cmx-frontend/
 ├─ src/
 │ ├─ app/
 │ │ ├─ core/
 │ │ │ ├─ services/
 │ │ │ ├─ guards/
 │ │ │ └─ interceptors/
 │ │ ├─ shared/
 │ │ │ ├─ components/
 │ │ │ ├─ directives/
 │ │ │ └─ pipes/
 │ │ ├─ features/
 │ │ ├─ app.component.ts
 │ │ ├─ app.config.ts
 │ │ └─ app.routes.ts
 │ ├─ assets/
 │ └─ environments/
```

---

## Domain Modules

### Dispatch Module
- Manages claim dispatching to surveyors or repair shops
- Handles allocation rules and priorities  
- Ensures claim cases are routed correctly and tracked

### FNOL (First Notice of Loss) Module
Handles the initial intake of claim cases. Functionalities include:

1. **Search Policy by Vehicle Registration Number**  
   - Allow searching existing policy records using car license plate
   - Validation: must match policy records in DB

2. **Search Policy by Insured Name/Surname**  
   - Allow searching using insured's first name/last name
   - Condition: require at least partial match with DB record

3. **Search Policy by ID Card Number / Passport**  
   - Allow lookup by unique ID number or passport number
   - Validation: exact match required

4. **Search Policy by Policy Detail**  
   - Search using policy number or contract reference
   - Must return exact or close matches

5. **Search Policy by Chassis Number**  
   - Lookup using vehicle chassis number (VIN)
   - Ensure uniqueness and validation against DB

### Surveyor Module
- Assign and manage surveyors for claims
- Handle scheduling, on-site survey logging, and photo uploads
- Track surveyor performance and workload  

---

## Architectural Guidelines

### 1. Controllers
- Entry points for API requests.  
- **STRICT RULE: No business logic, validation, or conditions** allowed in controllers.  
- Controllers must only delegate processing to **Handlers** via **MediatR**.  
- Controllers cannot directly interact with Managers or Services.
- **AI ASSISTANTS:** Always generate minimal controllers that only handle routing and delegation.  

### 2. Handlers
- Process requests received from controllers via **MediatR**.  
- Coordinate with **Managers** to execute business logic.  
- **STRICT RULE: Handlers can only interact with Managers**.
- Must implement `IRequestHandler<TRequest, TResponse>` interface.
- **AI ASSISTANTS:** Generate handlers that orchestrate workflow through managers only.  

### 3. Managers
- Encapsulate business workflows and logic.  
- Ensure reusability and separation of concerns.  
- **STRICT RULE: Managers can only interact with Services**.
- Handle complex business rules and orchestration.
- **AI ASSISTANTS:** Place all business logic in managers, never in handlers or controllers.  

### 4. Services
- Handle data access, external API calls, and utilities.  
- **EXCLUSIVE RULE: Only Services can directly access the database**.  
- Services cannot interact with Managers, Handlers, or Controllers.
- Implement repository patterns and data mapping.
- **AI ASSISTANTS:** All database operations must be implemented in services only.  

### 5. Layered Structure
Strict layered approach:  
`Controller → Handler → Manager → Service`

### 6. Validation
- **MANDATORY:** Must use **FluentValidation** with `AbstractValidator` classes.  
- **FORBIDDEN:** No `if-else` validation in controllers, handlers, or managers.  
- Rules must include required fields, format checks, and ranges.
- All validation must be declarative and testable.
- **AI ASSISTANTS:** Always create separate validator classes inheriting from `AbstractValidator<T>`.  

### 7. Error Handling
- All errors handled by **Global Exception Middleware**.  
- Use **ProblemDetails (RFC 7807)** for standardized API responses.  
- Exceptions must never propagate beyond middleware.  

### 8. Logging & Observability
- Use **EAGLE Logger** for all logs.  
- All logs must include **TraceId / CorrelationId**.  
- Support structured logging (JSON).  
- Log levels: `Info`, `Warning`, `Error`.  

### 9. Health Checks
- Expose health endpoints including:  
  - Version Information.  
  - Database connectivity.  
  - External dependency status.  

### 10. Testing
- **NON-NEGOTIABLE: 90%+ unit test coverage required**.  
- **NON-NEGOTIABLE: <10% code duplication** across the project.  
- Testing responsibilities:
  - Controller → routing/authorization  
  - Handler → request flow (mock Managers)  
  - Manager → business logic (mock Services)
  - Service → DB/API interactions (mock dependencies)  
- **MANDATORY FRAMEWORKS:** Xunit + NSubstitute.
- **AI ASSISTANTS:** Generate tests for every class created, following the testing responsibilities above.  

### 11. SonarQube/SonarLint Compliance
- Code must pass SonarQube scans with no critical issues.  
- Maintainability, Security, Reliability must be high.  
- Regular scans in CI/CD pipeline.  

---

## Frontend Guidelines (Angular)

### **Modern Angular Architecture (2024+ Best Practices)**

#### 1. **Standalone Components (Angular 14+)**
- **MANDATORY:** Use standalone components instead of NgModules
- **FORBIDDEN:** Traditional feature modules (except for lazy loading boundaries)
- Components should be self-contained with their own imports
- **AI ASSISTANTS:** Generate standalone components only

#### 2. **Signals & Modern Reactive Patterns (Angular 16+)**
- **PREFERRED:** Use Angular Signals for state management
- **PREFERRED:** Use `computed()` for derived state
- **PREFERRED:** Use `effect()` for side effects
- **FALLBACK:** RxJS only when signals aren't sufficient
- **AI ASSISTANTS:** Prefer signals over traditional observables

#### 3. **Control Flow Syntax (Angular 17+)**
- **MANDATORY:** Use `@if`, `@for`, `@switch` instead of structural directives
- **FORBIDDEN:** `*ngIf`, `*ngFor`, `*ngSwitch` (legacy syntax)
- **AI ASSISTANTS:** Always use new control flow syntax

#### 4. **Component Architecture**
- **Smart Components:** Handle business logic, API calls, state management
- **Dumb Components:** Pure presentation, input/output only
- **STRICT RULE:** Dumb components cannot inject services
- **STRICT RULE:** Smart components handle all side effects

#### 5. **Dependency Injection**
- **MANDATORY:** Use `inject()` function instead of constructor injection
- Services must be provided at appropriate levels (root, component, route)
- **AI ASSISTANTS:** Always use functional injection pattern

#### 6. **State Management Strategy**
```
Small/Medium Apps: Angular Signals + Services
Large Apps: NgRx + Signals integration
Real-time: Signals + RxJS streams
```

#### 7. **Form Handling**
- **MANDATORY:** Reactive Forms only (no template-driven forms)
- **PREFERRED:** Type-safe forms with strict typing
- Use custom validators with proper error handling
- **AI ASSISTANTS:** Generate strongly-typed reactive forms

#### 8. **Performance Best Practices**
- **MANDATORY:** OnPush change detection strategy
- **MANDATORY:** TrackBy functions for *ngFor loops
- Lazy loading for feature boundaries
- Image optimization with Angular Image directive

---

## Hooks

### Pre-Hook
Executed **before build/deploy**:
1. Run static code analysis (SOLID + N-Tier compliance).  
2. Run unit tests with Xunit (90% coverage).  
3. Check code duplication (<10%).  
4. Validate external dependencies.  

### Post-Hook
Executed **after build/deploy**:
1. Display code coverage (`Code Coverage: 92%`).  
2. Display code duplication (`Code Duplication: 8%`).  
3. Log results with **EAGLE Logger**.  
4. Notify stakeholders (email/Slack).  

---

## AI Assistant Instructions

### Command Examples
Users can request projects using commands like:

**Backend Projects:**
- "Create backend project using domain modules: Dispatch and FNOL"
- "Generate CMX backend with all domain modules"
- "Create FNOL module with policy search functionality"
- "Add Surveyor module to existing CMX project"

**Frontend Projects:**
- "Create Angular frontend project using domain modules: Dispatch and FNOL"
- "Generate CMX Angular frontend with all domain modules"
- "Create FNOL Angular module with policy search forms"
- "Add Surveyor Angular module to existing CMX frontend"

**Full-Stack Projects:**
- "Create full CMX project (backend + frontend) with Dispatch and FNOL modules"
- "Generate complete CMX solution with all domain modules"

### Code Generation Rules
1. **Architecture Compliance:** Always follow the strict layer separation: Controller → Handler → Manager → Service
2. **Validation First:** Create FluentValidation validators before implementing handlers
3. **Testing Required:** Generate unit tests for every class with appropriate mocking
4. **No Shortcuts:** Never bypass architectural constraints for convenience
5. **Naming Conventions:** Use consistent naming patterns across all layers
6. **Module Generation:** When domain modules are specified, generate complete folder structure and base classes

### Quality Gates
- All generated code must compile without warnings
- Unit tests must pass with 90%+ coverage
- No code duplication above 10%
- SonarQube compliance required

### Implementation Order
When creating new features, follow this sequence:
1. Create domain entities and interfaces
2. Create FluentValidation validators
3. Create services (data layer)
4. Create managers (business logic)
5. Create handlers (orchestration)
6. Create controllers (API endpoints)
7. Create comprehensive unit tests for all layers

### Project Generation Templates

#### Backend Project Template
When user requests "Create backend project using domain modules: [ModuleNames]":

1. **Create Solution Structure:**
   ```
   CMX.sln
   src/
   ├── CMX.API/
   ├── CMX.Application/
   ├── CMX.Domain/
   ├── CMX.Infrastructure/
   └── CMX.BuildingBlocks/
   tests/
   ├── CMX.UnitTests/
   └── CMX.IntegrationTests/
   ```

2. **For Each Domain Module, Generate:**
   - Domain/Entities/[Module]/ (e.g., Policy, Claim, Surveyor entities)
   - Domain/Interfaces/[Module]/ (e.g., IRepository interfaces)  
   - Application/Handlers/[Module]/ (e.g., SearchPolicyHandler, DispatchHandler)
   - Application/Validators/[Module]/ (e.g., SearchPolicyValidator)
   - Infrastructure/Services/[Module]/ (e.g., PolicyService, DispatchService)
   - API/Controllers/[Module]/ (e.g., PolicyController, DispatchController)
   - Unit test classes for all above components

3. **Base Infrastructure:**
   - Program.cs with DI configuration
   - Global Exception Middleware
   - EAGLE Logger integration
   - Flagsmith integration
   - Health check endpoints
   - MediatR setup
   - FluentValidation setup

#### Frontend Project Template
When user requests "Create Angular frontend project using domain modules: [ModuleNames]":

1. **Create Modern Angular Structure (Standalone Components):**
   ```
   cmx-frontend/
   ├── src/
   │   ├── app/
   │   │   ├── core/
   │   │   │   ├── services/
   │   │   │   ├── guards/
   │   │   │   └── interceptors/
   │   │   ├── shared/
   │   │   │   ├── components/
   │   │   │   ├── directives/
   │   │   │   └── pipes/
   │   │   ├── features/
   │   │   │   └── [module-folders]/
   │   │   ├── app.component.ts (standalone)
   │   │   ├── app.config.ts (providers)
   │   │   └── app.routes.ts
   │   ├── assets/
   │   └── environments/
   ```

2. **For Each Domain Module, Generate:**
   - features/[module]/components/ (smart components: list, search, assignment)
   - features/[module]/components/ (dumb components: card, form, dialog UI)  
   - features/[module]/services/ (HTTP services with inject() pattern)
   - features/[module]/models/ (TypeScript interfaces: Policy, Claim, Surveyor)
   - features/[module]/[module].routes.ts (lazy loading configuration)
   - Unit test files (.spec.ts) with 90%+ coverage for all components

3. **Base Angular Setup:**
   - main.ts with bootstrapApplication()
   - app.config.ts with providers configuration
   - Core services (HTTP interceptors, guards, error handling)
   - Shared standalone components and utilities
   - Flagsmith integration service with inject()
   - Environment configurations with type safety
   - Angular Material or modern UI library integration

---

## CI/CD Integration
- Hooks must run inside **CI/CD pipeline** (GitHub Actions, GitLab CI, or Azure DevOps).  
- Reports (coverage, SonarQube) uploaded as artifacts.  
- **BUILD FAILURE CONDITIONS:**
  - Coverage <90%
  - Code duplication >10%
  - SonarQube critical issues
  - Test failures  