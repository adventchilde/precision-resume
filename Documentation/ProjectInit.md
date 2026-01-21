# Tauri v2 + Next.js + Tailwind CSS + pnpm Project Initialization Guide

## Prerequisites

Before starting, ensure the following are installed:

- **Node.js** (v18+ recommended)
  - Check: `node --version`
- **Rust** (stable toolchain)
  - Check: `rustc --version`
- **C++ Build Tools** (required for native modules)
- **pnpm** package manager
  - Check: `pnpm --version`
  - Install: `npm install -g pnpm`

## Initialization Process

### 1. Create Project Directory

```bash
cd your-projects-directory
mkdir project-name
cd project-name
```

### 2. Create package.json

Create a `package.json` file with the following structure:

```json
{
  "name": "project-name",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "tauri": "tauri",
    "tauri:dev": "tauri dev",
    "tauri:build": "tauri build"
  },
  "dependencies": {
    "@tauri-apps/api": "^2.0.0",
    "@tauri-apps/plugin-shell": "^2.0.0",
    "next": "^15.0.0",
    "react": "^19.0.0",
    "react-dom": "^19.0.0"
  },
  "devDependencies": {
    "@tauri-apps/cli": "^2.0.0",
    "@types/node": "^20.0.0",
    "@types/react": "^19.0.0",
    "@types/react-dom": "^19.0.0",
    "autoprefixer": "^10.4.0",
    "eslint": "^9.0.0",
    "eslint-config-next": "^15.0.0",
    "postcss": "^8.4.0",
    "tailwindcss": "^3.4.0",
    "typescript": "^5.0.0"
  }
}
```

**Important Notes:**
- Tauri CLI (`@tauri-apps/cli`) must be in devDependencies
- Include both `@tauri-apps/api` and `@tauri-apps/plugin-shell` in dependencies
- React 19+ is compatible with Next.js 15

### 3. Create TypeScript Configuration

Create `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "ES2017",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [
      {
        "name": "next"
      }
    ],
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

### 4. Create Next.js Configuration

Create `next.config.ts`:

```typescript
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  /* config options here */
};

export default nextConfig;
```

### 5. Create Tailwind CSS Configuration

Create `tailwind.config.ts`:

```typescript
import type { Config } from "tailwindcss";

const config: Config = {
  content: [
    "./src/pages/**/*.{js,ts,jsx,tsx,mdx}",
    "./src/components/**/*.{js,ts,jsx,tsx,mdx}",
    "./src/app/**/*.{js,ts,jsx,tsx,mdx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
};
export default config;
```

### 6. Create PostCSS Configuration

Create `postcss.config.mjs`:

```javascript
/** @type {import('postcss-load-config').Config} */
const config = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
};

export default config;
```

### 7. Create Source Files

#### Global Styles
Create `src/app/globals.css`:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

#### Root Layout
Create `src/app/layout.tsx`:

```typescript
import type { Metadata } from "next";
import "./globals.css";

export const metadata: Metadata = {
  title: "Your App Name",
  description: "Your app description",
};

export default function RootLayout({
  children,
}: Readonly<{
  children: React.ReactNode;
}>) {
  return (
    <html lang="en">
      <body className="antialiased">
        {children}
      </body>
    </html>
  );
}
```

#### Home Page
Create `src/app/page.tsx`:

```typescript
export default function Home() {
  return (
    <main className="min-h-screen bg-gray-100 flex items-center justify-center">
      <div className="bg-white p-8 rounded-lg shadow-md">
        <h1 className="text-3xl font-bold text-gray-800 mb-4">
          Your App Name
        </h1>
        <p className="text-gray-600">
          Welcome to your application
        </p>
      </div>
    </main>
  );
}
```

### 8. Install Dependencies

```bash
pnpm install
```

**Note:** You may see TypeScript errors in your editor until dependencies are installed. This is normal and expected.

### 9. Initialize Tauri

Run the Tauri initialization command with all required flags:

```bash
pnpm tauri init \
  --app-name "Your App Name" \
  --window-title "Your App Name" \
  --dev-url "http://localhost:3000" \
  --frontend-dist "../dist" \
  --before-dev-command "pnpm dev" \
  --before-build-command "pnpm build"
```

**Important:**
- `--dev-url` must match Next.js default port (3000)
- `--frontend-dist` points to Next.js build output (relative to src-tauri)
- `--before-dev-command` starts Next.js dev server
- `--before-build-command` builds Next.js for production

**Issues to Watch For:**
- The command may prompt for input if any required flags are missing
- Use all flags above to avoid interactive prompts
- Ensure you're in the project root directory

### 10. Verify Tauri Configuration

Check `src-tauri/tauri.conf.json` to ensure it has:

```json
{
  "$schema": "../node_modules/@tauri-apps/cli/config.schema.json",
  "productName": "Your App Name",
  "version": "0.1.0",
  "identifier": "com.tauri.dev",
  "build": {
    "frontendDist": "../dist",
    "devUrl": "http://localhost:3000",
    "beforeDevCommand": "pnpm dev",
    "beforeBuildCommand": "pnpm build"
  },
  "app": {
    "windows": [
      {
        "title": "Your App Name",
        "width": 800,
        "height": 600,
        "resizable": true,
        "fullscreen": false
      }
    ],
    "security": {
      "csp": null
    }
  },
  "bundle": {
    "active": true,
    "targets": "all",
    "icon": [
      "icons/32x32.png",
      "icons/128x128.png",
      "icons/128x128@2x.png",
      "icons/icon.icns",
      "icons/icon.ico"
    ]
  }
}
```

### 11. Verify Environment

Check all prerequisites:

```bash
node --version    # Should be v18+
rustc --version   # Should be stable toolchain
pnpm --version    # Should be 8+ recommended
```

### 12. Test Development Server

#### Test Next.js Only
```bash
pnpm dev
```
- Open http://localhost:3000
- Verify the page renders with Tailwind styles

#### Test Tauri Desktop App
```bash
pnpm tauri:dev
```
- This will:
  1. Start Next.js dev server
  2. Compile Rust backend
  3. Open desktop window
  4. Enable hot reload for both frontend and backend

## Available Scripts

| Command | Description |
|---------|-------------|
| `pnpm dev` | Start Next.js dev server only |
| `pnpm build` | Build Next.js for production |
| `pnpm start` | Start Next.js production server |
| `pnpm tauri` | Run Tauri CLI commands |
| `pnpm tauri:dev` | Start Tauri development mode |
| `pnpm tauri:build` | Build desktop application for distribution |

## Project Structure

```
project-name/
├── src/
│   └── app/
│       ├── layout.tsx          # Root layout
│       ├── page.tsx            # Home page
│       └── globals.css         # Tailwind directives
├── src-tauri/
│   ├── src/
│   │   ├── main.rs            # Rust entry point
│   │   └── lib.rs             # Tauri commands
│   ├── icons/                 # App icons
│   ├── capabilities/          # Permissions config
│   ├── Cargo.toml             # Rust dependencies
│   ├── build.rs               # Build configuration
│   └── tauri.conf.json        # Tauri configuration
├── package.json               # Node dependencies
├── tsconfig.json             # TypeScript config
├── next.config.ts            # Next.js config
├── tailwind.config.ts        # Tailwind config
├── postcss.config.mjs        # PostCSS config
└── node_modules/             # Dependencies (auto-generated)
```

## Common Issues and Solutions

### TypeScript Errors Before Installation
**Issue:** Editor shows "Cannot find module 'next'" errors after creating config files
**Solution:** This is expected. Run `pnpm install` to install dependencies

### Tauri Init Prompts for Input
**Issue:** `pnpm tauri init` waits for user input
**Solution:** Use all required flags as shown in step 9

### Tauri Template Not Found
**Issue:** `pnpm create tauri-app --template next-ts` fails
**Solution:** Tauri CLI doesn't have a Next.js template. Initialize Next.js manually first, then add Tauri with `pnpm tauri init`

### Build Errors
**Issue:** Tauri build fails with native module errors
**Solution:** Ensure C++ Build Tools are installed for your platform

### Hot Reload Not Working
**Issue:** Changes don't reflect in Tauri dev window
**Solution:** Ensure `beforeDevCommand` is set to `pnpm dev` in tauri.conf.json

## Best Practices

1. **Version Management:** Keep Tauri API and CLI versions in sync
2. **Path Aliases:** Use `@/*` imports for cleaner code (already configured)
3. **Tailwind:** Extend the theme in `tailwind.config.ts` as needed
4. **Security:** Review and configure CSP in `tauri.conf.json` for production
5. **Icons:** Replace default Tauri icons with your app's branding

## Next Steps

After initialization:
1. Set up your app's routing structure in `src/app/`
2. Create reusable components in `src/components/`
3. Add Tauri commands in `src-tauri/src/lib.rs`
4. Configure app window size and behavior in `tauri.conf.json`
5. Set up state management if needed (Zustand, Redux, etc.)
6. Add form validation libraries if needed (Zod, React Hook Form)

## Resources

- [Tauri Documentation](https://v2.tauri.app/start)
- [Next.js Documentation](https://nextjs.org/docs)
- [Tailwind CSS Documentation](https://tailwindcss.com/docs)
- [Rust Book](https://doc.rust-lang.org/book/)
