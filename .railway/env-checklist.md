# Environment Variables Checkliste für Railway

Vollständige Liste aller Environment Variables für Twenty CRM auf Railway.

Complete list of all environment variables for Twenty CRM on Railway.

---

## ✅ Erforderliche Variablen / Required Variables

Diese Variablen **müssen** gesetzt werden, damit Twenty CRM funktioniert.

These variables **must** be set for Twenty CRM to work.

### 🔧 Core Configuration

| Variable | Beschreibung / Description | Beispielwert / Example | Wo setzen? / Where to set? |
|----------|---------------------------|----------------------|---------------------------|
| `SERVER_URL` | Öffentliche URL deiner Railway App / Public URL of your Railway app | `https://your-app.railway.app` | Server + Worker |
| `APP_SECRET` | Geheimer Schlüssel für Verschlüsselung (min. 32 Zeichen) / Secret key for encryption (min. 32 chars) | Generiere mit / Generate with: `openssl rand -base64 32` | Server + Worker |
| `PG_DATABASE_URL` | PostgreSQL Verbindungs-URL / PostgreSQL connection URL | `${{Postgres.DATABASE_URL}}` (automatisch / automatic) | Server + Worker |
| `REDIS_URL` | Redis Verbindungs-URL / Redis connection URL | `${{Redis.REDIS_URL}}` (automatisch / automatic) | Server + Worker |

### 🎨 Frontend Build

| Variable | Beschreibung / Description | Beispielwert / Example | Wo setzen? / Where to set? |
|----------|---------------------------|----------------------|---------------------------|
| `REACT_APP_SERVER_BASE_URL` | Backend URL für Frontend / Backend URL for frontend | `https://your-app.railway.app` (identisch zu SERVER_URL) | Server (Build-time!) |

### 💾 Storage Configuration

| Variable | Beschreibung / Description | Beispielwert / Example | Wo setzen? / Where to set? |
|----------|---------------------------|----------------------|---------------------------|
| `STORAGE_TYPE` | Speicher-Typ / Storage type | `local` (Development)<br>`s3` (Production empfohlen / recommended) | Server + Worker |
| `STORAGE_LOCAL_PATH` | Lokaler Speicherpfad (nur wenn `STORAGE_TYPE=local`) / Local storage path | `/app/packages/twenty-server/.local-storage` | Server + Worker |

---

## 🔐 S3 Storage (Empfohlen für Production / Recommended for Production)

Wenn `STORAGE_TYPE=s3`, setze folgende Variablen / If `STORAGE_TYPE=s3`, set these variables:

| Variable | Beschreibung / Description | Beispielwert / Example |
|----------|---------------------------|----------------------|
| `STORAGE_S3_REGION` | AWS Region oder kompatible Region / AWS Region or compatible region | `us-east-1`, `eu-central-1`, `auto` |
| `STORAGE_S3_NAME` | S3 Bucket Name | `my-twenty-storage` |
| `STORAGE_S3_ENDPOINT` | S3 Endpoint URL (optional für AWS, erforderlich für kompatible Services) | AWS: `https://s3.amazonaws.com`<br>Cloudflare R2: `https://[account-id].r2.cloudflarestorage.com`<br>Backblaze B2: `https://s3.[region].backblazeb2.com` |
| `STORAGE_S3_ACCESS_KEY_ID` | Access Key ID | `AKIAIOSFODNN7EXAMPLE` |
| `STORAGE_S3_SECRET_ACCESS_KEY` | Secret Access Key | `wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY` |

---

## ⚙️ Service-spezifische Variablen / Service-specific Variables

### Twenty Worker (zusätzlich / additional)

| Variable | Beschreibung / Description | Wert / Value |
|----------|---------------------------|--------------|
| `DISABLE_DB_MIGRATIONS` | Deaktiviert Migrations beim Worker-Start / Disable migrations on worker start | `true` |
| `DISABLE_CRON_JOBS_REGISTRATION` | Deaktiviert Cron Job Registrierung beim Worker / Disable cron job registration on worker | `true` |

---

## 🔓 Optionale Variablen / Optional Variables

Diese Variablen können auch über das Admin Panel konfiguriert werden, wenn `IS_CONFIG_VARIABLES_IN_DB_ENABLED=true` (Standard).

These variables can also be configured via the Admin Panel if `IS_CONFIG_VARIABLES_IN_DB_ENABLED=true` (default).

### 🔐 Google Authentication

| Variable | Beschreibung / Description | Beispiel / Example |
|----------|---------------------------|-------------------|
| `AUTH_GOOGLE_ENABLED` | Google Auth aktivieren / Enable Google auth | `true` |
| `AUTH_GOOGLE_CLIENT_ID` | Google OAuth Client ID | `123456789-abc.apps.googleusercontent.com` |
| `AUTH_GOOGLE_CLIENT_SECRET` | Google OAuth Client Secret | `GOCSPX-abc123...` |
| `AUTH_GOOGLE_CALLBACK_URL` | OAuth Callback URL | `https://your-app.railway.app/auth/google/redirect` |
| `AUTH_GOOGLE_APIS_CALLBACK_URL` | Google APIs Callback URL | `https://your-app.railway.app/auth/google-apis/redirect` |

### 🔐 Microsoft Authentication

| Variable | Beschreibung / Description | Beispiel / Example |
|----------|---------------------------|-------------------|
| `AUTH_MICROSOFT_ENABLED` | Microsoft Auth aktivieren / Enable Microsoft auth | `true` |
| `AUTH_MICROSOFT_CLIENT_ID` | Microsoft OAuth Client ID | `12345678-1234-1234-1234-123456789abc` |
| `AUTH_MICROSOFT_CLIENT_SECRET` | Microsoft OAuth Client Secret | `abc~123...` |
| `AUTH_MICROSOFT_CALLBACK_URL` | OAuth Callback URL | `https://your-app.railway.app/auth/microsoft/redirect` |
| `AUTH_MICROSOFT_APIS_CALLBACK_URL` | Microsoft APIs Callback URL | `https://your-app.railway.app/auth/microsoft-apis/redirect` |
| `AUTH_MICROSOFT_TENANT_ID` | Microsoft Tenant ID (optional) | `common` oder spezifische Tenant ID |

### 📧 Email Configuration

| Variable | Beschreibung / Description | Beispiel / Example |
|----------|---------------------------|-------------------|
| `EMAIL_FROM_ADDRESS` | Absender-Email / Sender email | `noreply@yourdomain.com` |
| `EMAIL_FROM_NAME` | Absender-Name / Sender name | `Twenty CRM` |
| `EMAIL_SYSTEM_ADDRESS` | System-Email Adresse / System email address | `system@yourdomain.com` |
| `EMAIL_DRIVER` | Email Treiber / Email driver | `smtp` |
| `EMAIL_SMTP_HOST` | SMTP Server / SMTP server | `smtp.gmail.com` |
| `EMAIL_SMTP_PORT` | SMTP Port | `465` (SSL) oder `587` (TLS) |
| `EMAIL_SMTP_USER` | SMTP Benutzername / SMTP username | `your-email@gmail.com` |
| `EMAIL_SMTP_PASSWORD` | SMTP Passwort / SMTP password | `your-app-password` |

### 📱 Messaging & Calendar Integrations

| Variable | Beschreibung / Description | Wert / Value |
|----------|---------------------------|--------------|
| `MESSAGING_PROVIDER_GMAIL_ENABLED` | Gmail Messaging aktivieren / Enable Gmail messaging | `true` oder `false` |
| `CALENDAR_PROVIDER_GOOGLE_ENABLED` | Google Calendar aktivieren / Enable Google Calendar | `true` oder `false` |
| `MESSAGING_PROVIDER_MICROSOFT_ENABLED` | Microsoft Messaging aktivieren / Enable Microsoft messaging | `true` oder `false` |
| `CALENDAR_PROVIDER_MICROSOFT_ENABLED` | Microsoft Calendar aktivieren / Enable Microsoft Calendar | `true` oder `false` |

### 📊 Analytics & Telemetry

| Variable | Beschreibung / Description | Wert / Value |
|----------|---------------------------|--------------|
| `TELEMETRY_ENABLED` | Telemetrie aktivieren / Enable telemetry | `true` (Standard / default) |
| `TELEMETRY_ANONYMIZATION_ENABLED` | Anonymisierte Telemetrie / Anonymized telemetry | `true` |
| `ANALYTICS_ENABLED` | Analytics aktivieren (ClickHouse erforderlich) / Enable analytics | `false` (Standard, da ClickHouse nicht standardmäßig vorhanden) |

### 🌐 API & CORS

| Variable | Beschreibung / Description | Beispiel / Example |
|----------|---------------------------|-------------------|
| `API_RATE_LIMITING_TTL` | Rate Limiting Time-to-Live (Sekunden) / Rate limiting TTL (seconds) | `60` |
| `API_RATE_LIMITING_LIMIT` | Rate Limiting Max Requests / Rate limiting max requests | `100` |
| `FRONT_BASE_URL` | Frontend Base URL (falls separiert) / Frontend base URL (if separated) | `https://your-app.railway.app` |

### 🔧 Advanced Configuration

| Variable | Beschreibung / Description | Wert / Value |
|----------|---------------------------|--------------|
| `IS_CONFIG_VARIABLES_IN_DB_ENABLED` | Ermöglicht Konfiguration über Admin Panel / Allows configuration via admin panel | `true` (Standard / default) |
| `PORT` | Server Port (Railway setzt dies automatisch) / Server port (Railway sets automatically) | `3000` |
| `NODE_ENV` | Node Environment | `production` |
| `LOG_LEVEL` | Log Level | `error`, `warn`, `info`, `debug` |
| `DEBUG_MODE` | Debug Modus aktivieren / Enable debug mode | `false` |

---

## 🎯 Empfohlene Minimal-Konfiguration / Recommended Minimal Configuration

Für einen **schnellen Start** (Development/Testing):

For a **quick start** (Development/Testing):

```bash
# Core (Required)
SERVER_URL=https://your-app.railway.app
APP_SECRET=[generiere-32-zeichen-secret]
PG_DATABASE_URL=${{Postgres.DATABASE_URL}}
REDIS_URL=${{Redis.REDIS_URL}}

# Frontend
REACT_APP_SERVER_BASE_URL=https://your-app.railway.app

# Storage (Lokal für Testing)
STORAGE_TYPE=local
STORAGE_LOCAL_PATH=/app/packages/twenty-server/.local-storage

# Worker Service (zusätzlich)
DISABLE_DB_MIGRATIONS=true
DISABLE_CRON_JOBS_REGISTRATION=true
```

---

## 🚀 Empfohlene Production-Konfiguration / Recommended Production Configuration

Für **Production-Einsatz**:

For **production use**:

```bash
# Core (Required)
SERVER_URL=https://your-app.railway.app
APP_SECRET=[starkes-geheimes-secret-min-32-zeichen]
PG_DATABASE_URL=${{Postgres.DATABASE_URL}}
REDIS_URL=${{Redis.REDIS_URL}}

# Frontend
REACT_APP_SERVER_BASE_URL=https://your-app.railway.app

# S3 Storage (Empfohlen!)
STORAGE_TYPE=s3
STORAGE_S3_REGION=eu-central-1
STORAGE_S3_NAME=your-bucket-name
STORAGE_S3_ENDPOINT=https://s3.amazonaws.com  # oder Cloudflare R2, etc.
STORAGE_S3_ACCESS_KEY_ID=[your-access-key]
STORAGE_S3_SECRET_ACCESS_KEY=[your-secret-key]

# Email (SMTP)
EMAIL_FROM_ADDRESS=noreply@yourdomain.com
EMAIL_FROM_NAME=Twenty CRM
EMAIL_DRIVER=smtp
EMAIL_SMTP_HOST=smtp.gmail.com
EMAIL_SMTP_PORT=465
EMAIL_SMTP_USER=your-email@gmail.com
EMAIL_SMTP_PASSWORD=[app-password]

# Auth (Optional - Google)
AUTH_GOOGLE_ENABLED=true
AUTH_GOOGLE_CLIENT_ID=[your-client-id]
AUTH_GOOGLE_CLIENT_SECRET=[your-client-secret]
AUTH_GOOGLE_CALLBACK_URL=https://your-app.railway.app/auth/google/redirect

# Worker Service (zusätzlich)
DISABLE_DB_MIGRATIONS=true
DISABLE_CRON_JOBS_REGISTRATION=true

# Security
LOG_LEVEL=warn
DEBUG_MODE=false
```

---

## 📝 Wie setze ich Environment Variables in Railway? / How to set environment variables in Railway?

### Methode 1: Railway Dashboard

1. Öffne dein Railway-Projekt / Open your Railway project
2. Wähle den Service (Server oder Worker) / Select the service (Server or Worker)
3. Klicke auf "Variables" Tab
4. Klicke "New Variable"
5. Gib Name und Wert ein / Enter name and value
6. Speichern / Save

### Methode 2: Railway CLI

```bash
# Login
railway login

# Link project
railway link

# Set variable
railway variables set SERVER_URL=https://your-app.railway.app

# Set multiple variables from file
railway variables set --file .env.railway
```

### Methode 3: Reference andere Services

Railway ermöglicht Service-Referenzen / Railway allows service references:

```bash
# PostgreSQL
PG_DATABASE_URL=${{Postgres.DATABASE_URL}}

# Redis
REDIS_URL=${{Redis.REDIS_URL}}

# Custom Service Reference
SOME_VAR=${{ServiceName.VARIABLE_NAME}}
```

---

## 🔍 Variablen-Validierung / Variable Validation

### APP_SECRET generieren / Generate APP_SECRET

**Linux/Mac:**
```bash
openssl rand -base64 32
```

**Windows (PowerShell):**
```powershell
[Convert]::ToBase64String((1..32 | ForEach-Object { Get-Random -Minimum 0 -Maximum 256 }))
```

**Online (falls kein OpenSSL verfügbar):**
- https://generate-secret.now.sh/32

### URLs validieren / Validate URLs

Stelle sicher, dass alle URLs identisch sind / Make sure all URLs are identical:
- `SERVER_URL` = `REACT_APP_SERVER_BASE_URL`
- Beide müssen `https://` verwenden (Railway macht dies automatisch)
- Beide both must use `https://` (Railway does this automatically)

### Callback URLs Format

```bash
# Google
AUTH_GOOGLE_CALLBACK_URL=https://your-app.railway.app/auth/google/redirect
AUTH_GOOGLE_APIS_CALLBACK_URL=https://your-app.railway.app/auth/google-apis/redirect

# Microsoft
AUTH_MICROSOFT_CALLBACK_URL=https://your-app.railway.app/auth/microsoft/redirect
AUTH_MICROSOFT_APIS_CALLBACK_URL=https://your-app.railway.app/auth/microsoft-apis/redirect
```

---

## ⚠️ Häufige Fehler / Common Mistakes

### ❌ Fehler: Unterschiedliche URLs

```bash
# FALSCH / WRONG
SERVER_URL=https://app1.railway.app
REACT_APP_SERVER_BASE_URL=https://app2.railway.app

# RICHTIG / CORRECT
SERVER_URL=https://your-app.railway.app
REACT_APP_SERVER_BASE_URL=https://your-app.railway.app
```

### ❌ Fehler: Schwaches APP_SECRET

```bash
# FALSCH / WRONG - zu kurz
APP_SECRET=mysecret

# RICHTIG / CORRECT - mindestens 32 Zeichen
APP_SECRET=Xy9mK2nP5qR8sT1vW4xZ7aC0dE3fG6hJ9kL2mN5p
```

### ❌ Fehler: Falscher Storage Path

```bash
# FALSCH / WRONG
STORAGE_LOCAL_PATH=/storage

# RICHTIG / CORRECT
STORAGE_LOCAL_PATH=/app/packages/twenty-server/.local-storage
```

### ❌ Fehler: Worker führt Migrations aus

```bash
# Worker Service braucht diese beiden Variablen!
# Worker service needs these two variables!
DISABLE_DB_MIGRATIONS=true
DISABLE_CRON_JOBS_REGISTRATION=true
```

---

## 📚 Weitere Informationen / Additional Information

- [Twenty CRM Environment Setup Docs](../packages/twenty-docs/developers/self-hosting/setup.mdx)
- [Railway Environment Variables Docs](https://docs.railway.app/develop/variables)
- [Docker Environment Example](../packages/twenty-docker/.env.example)

---

**Tipp:** Nutze Railway's "Shared Variables" Feature, um Variablen zwischen Server und Worker zu teilen!

**Tip:** Use Railway's "Shared Variables" feature to share variables between Server and Worker services!
