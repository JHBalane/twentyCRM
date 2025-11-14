# Twenty CRM - Development Guide

Dieses Dokument beschreibt, wie du Twenty CRM lokal entwickelst und neue Features hinzufügst.

This document describes how to develop Twenty CRM locally and add new features.

---

## 🚀 Quick Start (Deutsch)

### Voraussetzungen / Prerequisites

**System-Anforderungen:**
- **Node.js:** 24.5.0 oder höher
- **Yarn:** 4.x (wird automatisch via Corepack installiert)
- **PostgreSQL:** 16.x
- **Redis:** Latest

**Empfohlene Tools:**
- Docker Desktop (für lokale Datenbank & Redis)
- Git
- VS Code oder WebStorm/IntelliJ IDEA

---

### 1. Repository Setup

```bash
# Repository klonen
git clone https://github.com/twentyhq/twenty.git
cd twenty

# Oder dein Fork
git clone https://github.com/DEIN-USERNAME/twenty.git
cd twenty

# Yarn aktivieren (Corepack)
corepack enable

# Dependencies installieren (kann ein paar Minuten dauern)
yarn install
```

---

### 2. Datenbank & Redis starten

**Option A: Mit Docker (empfohlen)**

```bash
# Im Root-Verzeichnis
cd packages/twenty-docker
docker-compose up -d db redis

# Überprüfen, ob Services laufen
docker-compose ps
```

**Option B: Lokal installiert**

Falls du PostgreSQL und Redis bereits lokal installiert hast:

```bash
# PostgreSQL starten (macOS mit Homebrew)
brew services start postgresql@16

# Redis starten
brew services start redis

# Datenbank erstellen
createdb twenty
```

---

### 3. Environment Variables konfigurieren

```bash
# Erstelle .env im Root-Verzeichnis
cp packages/twenty-server/.env.example .env

# Oder manuell erstellen:
touch .env
```

**Minimale .env für lokales Development:**

```bash
# Core
SERVER_URL=http://localhost:3000
APP_SECRET=replace-me-with-a-random-string-min-32-chars

# Database (wenn Docker verwendet)
PG_DATABASE_URL=postgresql://postgres:postgres@localhost:5432/twenty

# Oder separate Variablen
PG_DATABASE_HOST=localhost
PG_DATABASE_PORT=5432
PG_DATABASE_USER=postgres
PG_DATABASE_PASSWORD=postgres
PG_DATABASE_NAME=twenty

# Redis (wenn Docker verwendet)
REDIS_URL=redis://localhost:6379

# Storage (lokal)
STORAGE_TYPE=local
STORAGE_LOCAL_PATH=.local-storage

# Frontend
REACT_APP_SERVER_BASE_URL=http://localhost:3000

# Development
NODE_ENV=development
DEBUG_MODE=true
LOG_LEVEL=debug
```

**APP_SECRET generieren:**
```bash
openssl rand -base64 32
```

---

### 4. Datenbank initialisieren

```bash
# Datenbank migrations ausführen
npx nx database:reset twenty-server

# Oder schrittweise:
npx nx run twenty-server:typeorm migration:run
npx nx run twenty-server:command workspace:sync-metadata
```

---

### 5. Development Server starten

**Option A: Alles auf einmal (Frontend + Backend + Worker)**

```bash
yarn start
```

Dies startet:
- **Frontend:** http://localhost:3001 (Vite Dev Server)
- **Backend:** http://localhost:3000 (NestJS API)
- **Worker:** Background Job Processor

**Option B: Services einzeln starten**

```bash
# Terminal 1: Backend
npx nx start twenty-server

# Terminal 2: Frontend
npx nx start twenty-front

# Terminal 3: Worker (optional für Background Jobs)
npx nx run twenty-server:worker
```

---

### 6. App öffnen & ersten Account erstellen

1. Öffne http://localhost:3001 im Browser
2. Klicke auf "Continue with Email"
3. Nutze die vorgefüllten Test-Credentials oder erstelle einen neuen Account
4. Fertig! 🎉

---

## 🛠️ Development Workflow

### Code-Änderungen machen

Twenty CRM nutzt ein **Nx Monorepo**. Die wichtigsten Packages sind:

```
packages/
├── twenty-front/          # React Frontend
├── twenty-server/         # NestJS Backend
├── twenty-ui/             # Shared UI Components
└── twenty-shared/         # Common Types & Utils
```

**Frontend-Änderungen:**
```bash
# Starte nur Frontend Dev Server
npx nx start twenty-front

# Komponenten in Storybook entwickeln
npx nx storybook:dev twenty-front

# Frontend Tests
npx nx test twenty-front

# Linting
npx nx lint twenty-front --fix
```

**Backend-Änderungen:**
```bash
# Starte Backend
npx nx start twenty-server

# Backend Tests
npx nx test twenty-server

# Integration Tests (mit DB Reset)
npx nx run twenty-server:test:integration:with-db-reset

# Linting
npx nx lint twenty-server --fix
```

---

### Neue Datenbank-Migration erstellen

Wenn du das Datenmodell änderst:

```bash
# Migration generieren
npx nx run twenty-server:typeorm migration:generate \
  src/database/typeorm/core/migrations/common/[DEIN-NAME] \
  -d src/database/typeorm/core/core.datasource.ts

# Migration ausführen
npx nx run twenty-server:typeorm migration:run

# Metadata synchronisieren
npx nx run twenty-server:command workspace:sync-metadata
```

**Beispiel:**
```bash
npx nx run twenty-server:typeorm migration:generate \
  src/database/typeorm/core/migrations/common/AddUserPreferences \
  -d src/database/typeorm/core/core.datasource.ts
```

---

### GraphQL Schema Änderungen

Wenn du GraphQL Types änderst:

```bash
# GraphQL Types für Frontend generieren
npx nx run twenty-front:graphql:generate

# Startet Backend neu, um Schema-Änderungen zu laden
npx nx start twenty-server
```

---

## 🧪 Testing

### Unit Tests

```bash
# Frontend Tests
npx nx test twenty-front

# Backend Tests
npx nx test twenty-server

# Specific file
npx nx test twenty-front --testFile=src/components/MyComponent.test.tsx

# Watch mode
npx nx test twenty-front --watch
```

### Integration Tests

```bash
# Backend Integration Tests mit DB Reset
npx nx run twenty-server:test:integration:with-db-reset

# Ohne DB Reset (schneller)
npx nx run twenty-server:test:integration
```

### E2E Tests

```bash
# Playwright E2E Tests
cd packages/twenty-e2e-testing
npx playwright test

# Mit UI
npx playwright test --ui

# Specific test
npx playwright test tests/login.spec.ts
```

### Storybook Tests

```bash
# Storybook bauen
npx nx storybook:build twenty-front

# Storybook Tests ausführen
npx nx storybook:serve-and-test:static twenty-front
```

---

## 🎨 Code Style & Best Practices

### Wichtige Regeln (aus CLAUDE.md)

1. **Nur Funktionale Komponenten** (keine Klassenkomponenten)
   ```typescript
   // ✅ Gut
   export const MyComponent = () => { ... }

   // ❌ Schlecht
   export class MyComponent extends React.Component { ... }
   ```

2. **Nur Named Exports** (keine Default Exports)
   ```typescript
   // ✅ Gut
   export const MyComponent = () => { ... }

   // ❌ Schlecht
   export default MyComponent;
   ```

3. **Types statt Interfaces** (außer bei Third-Party Extensions)
   ```typescript
   // ✅ Gut
   export type User = { ... }

   // ❌ Schlecht (außer bei extend von Third-Party)
   export interface User { ... }
   ```

4. **String Literals statt Enums** (außer GraphQL Enums)
   ```typescript
   // ✅ Gut
   type Status = 'active' | 'inactive' | 'pending';

   // ❌ Schlecht
   enum Status { Active, Inactive, Pending }
   ```

5. **Kein 'any' Typ erlaubt**
   ```typescript
   // ✅ Gut
   const data: unknown = ...;
   const user: User = ...;

   // ❌ Schlecht
   const data: any = ...;
   ```

6. **Event Handlers bevorzugt statt useEffect** für State Updates
   ```typescript
   // ✅ Gut
   const handleClick = () => setCount(count + 1);

   // ❌ Vermeiden wenn möglich
   useEffect(() => setCount(count + 1), [dependency]);
   ```

---

### Styling (Emotion)

Twenty nutzt **Emotion** für Styling:

```typescript
import styled from '@emotion/styled';

export const StyledButton = styled.button`
  padding: ${({ theme }) => theme.spacing(2)};
  color: ${({ theme }) => theme.font.color.primary};
  background: ${({ theme }) => theme.background.primary};

  &:hover {
    background: ${({ theme }) => theme.background.secondary};
  }
`;
```

---

### State Management (Recoil)

Twenty nutzt **Recoil** für globales State Management:

```typescript
// atoms/userState.ts
import { atom } from 'recoil';

export const userState = atom<User | null>({
  key: 'userState',
  default: null,
});

// In Component
import { useRecoilState } from 'recoil';
import { userState } from '@/atoms/userState';

export const MyComponent = () => {
  const [user, setUser] = useRecoilState(userState);
  // ...
};
```

---

### Internationalization (Lingui)

```typescript
import { useLingui } from '@lingui/react';
import { t } from '@lingui/macro';

export const MyComponent = () => {
  const { i18n } = useLingui();

  return <div>{t`Hello World`}</div>;
};
```

---

## 📦 Nützliche Nx Commands

### Build

```bash
# Frontend bauen
npx nx build twenty-front

# Backend bauen
npx nx build twenty-server

# Alle Packages bauen
npx nx run-many --target=build --all
```

### Linting & Formatting

```bash
# Linting
npx nx lint twenty-front
npx nx lint twenty-server

# Auto-fix
npx nx lint twenty-front --fix
npx nx lint twenty-server --fix

# Format code
npx nx fmt twenty-front
npx nx fmt twenty-server
```

### Type Checking

```bash
npx nx typecheck twenty-front
npx nx typecheck twenty-server
```

### Dependency Graph

```bash
# Zeigt Abhängigkeiten zwischen Packages
npx nx graph
```

---

## 🐛 Troubleshooting

### Problem: "Cannot find module" oder Build-Fehler

**Lösung:**
```bash
# Dependencies neu installieren
rm -rf node_modules
yarn install

# Nx Cache löschen
npx nx reset
```

---

### Problem: Datenbank-Verbindungsfehler

**Lösung:**
```bash
# Überprüfe ob PostgreSQL läuft
docker-compose ps  # Wenn Docker verwendet

# Oder direkt
psql -U postgres -h localhost -c "SELECT version();"

# Überprüfe PG_DATABASE_URL in .env
echo $PG_DATABASE_URL
```

---

### Problem: Redis-Verbindungsfehler

**Lösung:**
```bash
# Überprüfe ob Redis läuft
docker-compose ps  # Wenn Docker verwendet

# Oder direkt
redis-cli ping  # Sollte "PONG" zurückgeben

# Überprüfe REDIS_URL in .env
echo $REDIS_URL
```

---

### Problem: Port already in use

**Lösung:**
```bash
# Finde Prozess auf Port 3000 (macOS/Linux)
lsof -ti:3000

# Töte Prozess
kill -9 $(lsof -ti:3000)

# Windows
netstat -ano | findstr :3000
taskkill /PID <PID> /F
```

---

### Problem: GraphQL Schema out of sync

**Lösung:**
```bash
# Backend neu starten
npx nx start twenty-server

# GraphQL Types neu generieren
npx nx run twenty-front:graphql:generate
```

---

### Problem: "migrations need to run"

**Lösung:**
```bash
# Datenbank zurücksetzen (ACHTUNG: Löscht alle Daten!)
npx nx database:reset twenty-server

# Oder nur neue Migrations ausführen
npx nx run twenty-server:typeorm migration:run
```

---

## 🔀 Git Workflow

### Branch Strategie

```bash
# Feature Branch erstellen
git checkout -b feature/your-feature-name

# Oder Bug Fix
git checkout -b fix/your-bug-fix

# Von main aktuell halten
git checkout main
git pull origin main
git checkout feature/your-feature-name
git rebase main
```

---

### Commit Messages

Twenty folgt **Conventional Commits**:

```bash
# Format
<type>(<scope>): <subject>

# Beispiele
feat(frontend): add dark mode toggle
fix(backend): resolve database connection issue
docs(readme): update installation instructions
refactor(api): simplify user controller logic
test(integration): add tests for auth flow
```

**Types:**
- `feat`: Neues Feature
- `fix`: Bug Fix
- `docs`: Dokumentation
- `refactor`: Code-Refactoring
- `test`: Tests hinzufügen/ändern
- `chore`: Build/Config Änderungen

---

### Pre-commit Checks

Vor dem Commit:

```bash
# Linting
npx nx lint twenty-front --fix
npx nx lint twenty-server --fix

# Tests
npx nx test twenty-front
npx nx test twenty-server

# Type checking
npx nx typecheck twenty-front
npx nx typecheck twenty-server
```

---

## 🚀 Von Local zu Railway deployen

### 1. Änderungen testen

```bash
# Lokale Tests
yarn test

# Build testen
npx nx build twenty-front
npx nx build twenty-server

# Docker Build testen (optional)
docker build -f packages/twenty-docker/twenty/Dockerfile -t twenty:test .
docker run -p 3000:3000 twenty:test
```

---

### 2. Code pushen

```bash
# Zu deinem Fork pushen
git add .
git commit -m "feat: your feature description"
git push origin feature/your-feature-name
```

---

### 3. Railway Auto-Deploy

Falls du **Railway mit GitHub verbunden** hast:

1. **Automatisches Deployment:** Railway deployt automatisch bei jedem Push zu `main`
2. **Preview Deployments:** Branches erstellen automatisch Preview Environments
3. **Environment Variables:** Stelle sicher, dass alle Variablen in Railway gesetzt sind

---

### 4. Manuelles Railway Deployment

```bash
# Railway CLI installieren
npm i -g @railway/cli

# Login
railway login

# Projekt linken
railway link

# Deploy
railway up

# Logs anschauen
railway logs
```

---

## 📚 Weitere Ressourcen

### Dokumentation

- **Twenty Docs:** https://twenty.com/developers
- **NestJS Docs:** https://docs.nestjs.com/
- **React Docs:** https://react.dev/
- **Recoil Docs:** https://recoiljs.org/
- **Nx Docs:** https://nx.dev/

### Community

- **GitHub:** https://github.com/twentyhq/twenty
- **Discord:** https://discord.gg/twenty
- **Twitter:** https://twitter.com/twentycrm

---

## 💡 Tipps für neue Entwickler

### Frontend Development

1. **Nutze Storybook** für UI-Komponenten
   ```bash
   npx nx storybook:dev twenty-front
   ```

2. **Hot Reload:** Vite bietet super schnelles Hot Reloading
3. **DevTools:** React DevTools + Recoil DevTools installieren
4. **UI Library:** Nutze `twenty-ui` Components wo möglich

---

### Backend Development

1. **GraphQL Playground:** http://localhost:3000/graphql
2. **API Docs:** NestJS generiert automatisch Swagger-Docs
3. **Database GUI:** Nutze TablePlus, pgAdmin oder DBeaver
4. **Redis GUI:** RedisInsight für Redis-Debugging

---

### Performance

1. **Nx Cache:** Nutze Nx-Cache für schnellere Builds
   ```bash
   npx nx build twenty-front  # Zweiter Run ist viel schneller
   ```

2. **Selective Testing:** Teste nur betroffene Projekte
   ```bash
   npx nx affected:test
   ```

3. **Build Parallelisierung:**
   ```bash
   npx nx run-many --target=build --all --parallel=3
   ```

---

## ✅ Checklist für neues Feature

- [ ] Feature Branch erstellt (`feature/my-feature`)
- [ ] Code geschrieben (Frontend und/oder Backend)
- [ ] Types/Interfaces definiert (keine `any` Types)
- [ ] Unit Tests geschrieben
- [ ] Integration Tests (falls Backend)
- [ ] Linting passes (`npx nx lint --fix`)
- [ ] Type checking passes (`npx nx typecheck`)
- [ ] Storybook Stories (falls UI Component)
- [ ] GraphQL Types generiert (falls Schema geändert)
- [ ] Database Migration (falls Schema geändert)
- [ ] Dokumentation aktualisiert
- [ ] Commit Message folgt Convention
- [ ] Lokal getestet
- [ ] Bereit für Pull Request / Railway Deploy

---

**Viel Erfolg beim Entwickeln! 🚀**

Bei Fragen: Twenty Discord Community oder GitHub Issues.
