# System Patterns

## System Architecture

### Three-Layer Desktop Application

```
precision-resume/
├── Frontend Layer (Next.js)
│   ├── React Components (UI)
│   ├── State Management (React hooks, Context API)
│   ├── Styling (Tailwind CSS)
│   └── Routing (Next.js App Router)
│
├── Desktop Layer (Tauri v2)
│   ├── Native Window Management
│   ├── System Integration
│   ├── Rust Commands (Backend Logic)
│   └── Hybrid Inference Toggle (Local/Cloud Mode)
│
└── Data Layer
    ├── JSON Resume Schema (Universal Data Portability)
    ├── Local File System Storage
    └── AppData directory storage
```

### File Structure

```
precision-resume/
├── src/              # Next.js app directory
│   └── app/         # App Router pages and layouts
├── src-tauri/        # Rust backend and Tauri configuration
│   ├── src/          # Rust source files
│   ├── Cargo.toml     # Rust dependencies
│   └── tauri.conf.json # Tauri configuration
├── public/           # Static assets
├── memory-bank/     # Project documentation
└── [config files]   # package.json, next.config.ts, etc.
```

## Key Technical Decisions

### 1. Local-First Privacy Architecture
- **Decision:** All data stored locally on user's machine with hybrid inference options
- **Implementation:** 
  - Local Mode: Ollama/Llama 3 for total privacy
  - Cloud Mode: Groq/OpenAI for high-speed reasoning
  - Toggle UI switch for mode selection
- **Benefits:** Complete data sovereignty, zero cloud leakage, flexible performance
- **Trade-off:** Local mode requires more computational resources

### 2. Universal Data Portability (JSON Resume)
- **Decision:** Adopt open-source JSON Resume schema for all internal storage
- **Rationale:** Ensures career data remains portable and structured
- **Benefits:** No vendor lock-in, exportable Master Profile, data interoperability
- **Implementation:** All resume data structures follow JSON Resume specification

### 3. Tauri v2 Framework Choice
- **Decision:** Use Tauri v2 over Electron
- **Rationale:** Smaller bundle size, native performance, Rust safety guarantees
- **Benefits:** Fast startup, low memory footprint, native Windows integration
- **Trade-off:** Requires Rust knowledge for backend logic

### 4. Next.js with Static Export
- **Decision:** Configure Next.js for static site generation
- **Rationale:** Required for Tauri bundling, simplifies deployment
- **Benefits:** Faster build times, smaller bundles, no server runtime overhead
- **Constraint:** Must use `output: 'export'` and `images.unoptimized: true`

### 5. TypeScript Throughout
- **Decision:** Use TypeScript for all new code
- **Rationale:** Type safety, better developer experience, reduced runtime bugs
- **Benefits:** IntelliSense support, early error detection, self-documenting code

## Design Patterns in Use

### Component Architecture
- **Pattern:** Atomic Design principles
  - **Atoms:** Basic UI elements (buttons, inputs, labels)
  - **Molecules:** Composed components (forms, cards, search bars)
  - **Organisms:** Complex sections (layouts, feature modules)
- **Benefits:** Reusability, maintainability, clear component hierarchy
- **Guideline:** Keep components small and focused to avoid context drift

### State Management
- **Pattern:** React Hooks + Context API
  - **Local State:** useState/useReducer for component-level state
  - **Global State:** Context providers for shared application state (e.g., resume data, inference mode)
  - **Persistence:** Local file system via Tauri commands
- **Benefits:** Simple, built-in, no additional dependencies

### Data Flow
- **Pattern:** Unidirectional data flow with bidirectional communication
  - **Frontend → Backend:** Tauri invoke commands (synchronous/async Rust functions)
  - **Backend → Frontend:** Tauri event emission (push updates to UI)
  - **Storage:** Direct read/write to JSON files via Tauri commands

## Component Relationships

### Frontend Layer Structure
```
src/app/
├── layout.tsx          # Root layout (global styles, providers)
├── page.tsx            # Home page component
├── globals.css         # Global styles
└── [routes]/           # Future route-specific pages and layouts
```

### Backend Layer Structure
```
src-tauri/src/
├── main.rs             # Entry point, window configuration
├── lib.rs              # Tauri commands, event handlers
└── build.rs            # Build configuration, asset bundling
```

### Data Layer Integration
- **Storage Location:** User's AppData directory
- **File Format:** JSON following JSON Resume schema
- **Access Pattern:** Via Tauri commands (not direct file access)

## Integration Points

### Next.js ↔ Tauri Communication

**Frontend to Rust (Invoke Commands):**
```typescript
import { invoke } from '@tauri-apps/api/core';

// Call a Rust command with parameters
const result = await invoke('command_name', { 
  param: value 
});
```

**Rust to Frontend (Event Emission):**
```rust
// Emit an event to frontend
tauri::Event::emit(&window, "event-name", payload)?;
```

**Frontend Event Listener:**
```typescript
import { listen } from '@tauri-apps/api/event';

// Listen for events from Rust
const unlisten = await listen('event-name', (event) => {
  console.log('Received:', event.payload);
});

// Cleanup when done
unlisten();
```

### Hybrid Inference Pattern
```typescript
// Toggle between local and cloud inference
const inferenceMode = isInferenceLocal ? 'ollama' : 'groq';

if (inferenceMode === 'ollama') {
  // Call local Ollama endpoint
  const result = await invoke('local_inference', { prompt });
} else {
  // Call cloud API (Groq/OpenAI)
  const result = await invoke('cloud_inference', { prompt, provider: 'groq' });
}
```

### JSON Resume Operations Pattern
```typescript
// Read resume data
const resumeData = await invoke('read_resume', { 
  path: 'resume.json' 
});

// Write resume data
await invoke('save_resume', { 
  path: 'resume.json', 
  data: resumeData 
});

// Export to different formats
const pdf = await invoke('export_pdf', { resumeData });
const html = await invoke('export_html', { resumeData });
```

## Development Workflow Patterns

### Local Development
1. **Start:** `pnpm tauri:dev` (starts both Next.js dev server and Tauri window)
2. **Frontend Changes:** Hot-reload automatically in browser/Tauri window
3. **Rust Changes:** Trigger automatic recompilation (slower than frontend)
4. **Debugging:** Browser DevTools available for frontend debugging

### Production Build
1. **Run:** `pnpm tauri:build`
2. **Next.js:** Builds to static files (output export)
3. **Rust:** Compiles backend code
4. **Bundle:** Combines everything into Windows executable (.exe)
5. **Installer:** Creates .msi installer for distribution

### Code Organization Best Practices
- **Components:** One component per file, export default
- **Tauri Commands:** Group related commands in lib.rs
- **State:** Keep local state in components, lift to Context when shared
- **Types:** Define interfaces for all data structures (follow JSON Resume schema)
- **Error Handling:** Use try/catch for async operations, display user-friendly errors
- **Privacy:** Never send sensitive career data to external services without user consent

## Critical Implementation Paths

### Adding a New Feature
1. Define data model (TypeScript interface following JSON Resume schema)
2. Create Tauri commands for backend logic (if needed)
3. Build React components for UI
4. Connect UI to Tauri commands
5. Add state management (local or Context)
6. Persist data to JSON files (if needed)
7. Implement hybrid inference toggle support (if AI feature)

### Implementing Hybrid Inference
1. Create UI toggle component for Local/Cloud mode selection
2. Implement local inference Tauri command (Ollama integration)
3. Implement cloud inference Tauri command (Groq/OpenAI integration)
4. Create unified interface in frontend that routes to appropriate backend
5. Add error handling for both modes
6. Store user preference in local settings

### Modifying Resume Data Schema
1. Update TypeScript interfaces (ensure JSON Resume compatibility)
2. Update Tauri commands to handle new schema
3. Migration strategy for existing data (if any)
4. Update UI components to use new fields
5. Test export/import functionality

## Privacy and Security Patterns

### Data Sovereignty
- All career data stored locally on user's machine
- API keys and credentials never logged or sent to external services
- User controls when and if data is sent to cloud inference
- Clear visual indicators for current inference mode (Local vs Cloud)

### ATS "Robot View" Validation
- Implement local PDF parsing engine
- Reverse-engineer generated PDFs to show ATS interpretation
- Identify formatting "breakers" (columns, icons, tables)
- Ensure 100% data fidelity during submission

### Semantic Scoring
- Real-time gap analysis (0-100 score)
- Keyword match rate calculation
- Formatting compliance checking
- Low-latency inference for immediate feedback

## Technologies Used

### Frontend Stack
- **Framework:** Next.js 15.0.0
- **UI Library:** React 19.0.0
- **Language:** TypeScript 5.0.0
- **Styling:** Tailwind CSS 3.4.0
- **Router:** Next.js App Router

### Desktop Wrapper
- **Framework:** Tauri v2.0.0
- **Backend Language:** Rust
- **Plugins:** @tauri-apps/plugin-shell

### Inference (Planned)
- **Local Mode:** Ollama + Llama 3
- **Cloud Mode:** Groq / OpenAI

### Data Standards
- **Schema:** JSON Resume (open-source standard)
- **Format:** JSON

## Development Setup

### Environment Requirements
- **Node.js:** v20.0.0 or higher
- **Package Manager:** pnpm
- **Rust/Cargo:** 1.92.0 or higher (for Tauri)
- **Build Tools:** Appropriate platform build tools (e.g., Visual Studio C++ Build Tools on Windows)
- **IDE:** Visual Studio Code (recommended)

### Installation Steps
1. Clone repository
2. Install dependencies: `pnpm install`
3. Start development: `pnpm tauri:dev`

## Technical Constraints

### Data Architecture
- **Local-First:** All data stored locally on user's machine
- **No Backend Server:** Self-contained application (no external server)
- **Storage:** JSON files following JSON Resume schema
- **Privacy:** Zero cloud leakage by default (cloud mode opt-in)

### Deployment Requirements
- **Zero Dependencies:** Installer requires no pre-requisite software from user
- **Single File:** Delivered as .msi installer
- **Build Output:** Cross-platform Windows .msi and .exe installers

### Development Guidelines
- **TypeScript:** Use for all new code (type safety, better DX)
- **Component Size:** Keep components small and modular
- **Code Style:** Direct, functional implementations (no filler code)
- **Privacy First:** Default to local processing, explicit user consent for cloud

## Dependencies

### Frontend (package.json)
- next: ^15.0.0
- react: ^19.0.0
- react-dom: ^19.0.0
- typescript: ^5.0.0
- tailwindcss: ^3.4.0
- @tauri-apps/api: ^2.0.0
- @tauri-apps/plugin-shell: ^2.0.0

### Backend (Cargo.toml)
- tauri (with default features)
- serde (JSON serialization)
- tokio (async runtime)

## Tool Usage Patterns

### Running Development Server
```bash
pnpm tauri:dev
```
This starts both Next.js dev server and Tauri window simultaneously.

### Building for Production
```bash
pnpm tauri:build
```
This creates a production build and packages it into an installer.

### Linting
```bash
pnpm lint
```
Runs ESLint to check code quality.
