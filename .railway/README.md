# Twenty CRM auf Railway deployen

Dieses Dokument beschreibt, wie du Twenty CRM auf Railway deployen kannst.

This document describes how to deploy Twenty CRM on Railway.

---

## 🚀 Quick Start (Deutsch)

### Option 1: Community-Vorlage nutzen (Empfohlen)

Die schnellste Methode ist die vorgefertigte Railway-Vorlage:

1. **Klicke auf den Deploy-Button:**
   ```
   https://railway.com/deploy/nAL3hA
   ```

2. **Railway erstellt automatisch:**
   - ✅ Twenty Server (Haupt-Applikation auf Port 3000)
   - ✅ Twenty Worker (Background Job Processor)
   - ✅ PostgreSQL 16 Datenbank
   - ✅ Redis Cache & Session Store

3. **Konfiguriere die Environment Variables** (siehe unten)

4. **Warte auf das Deployment** (ca. 5-10 Minuten beim ersten Mal)

5. **Öffne deine App:** Die URL findest du im Railway Dashboard unter "Settings" → "Public Domain"

---

### Option 2: Manuelles Setup

Falls du mehr Kontrolle möchtest:

#### Schritt 1: Services erstellen

Erstelle in deinem Railway-Projekt folgende Services:

**1. PostgreSQL Datenbank**
- Gehe zu "New" → "Database" → "Add PostgreSQL"
- Railway generiert automatisch `DATABASE_URL`

**2. Redis**
- Gehe zu "New" → "Database" → "Add Redis"
- Railway generiert automatisch `REDIS_URL`

**3. Twenty Server**
- Gehe zu "New" → "GitHub Repo" → Wähle dein Twenty CRM Fork
- **Dockerfile Path:** `packages/twenty-docker/twenty/Dockerfile`
- **Root Directory:** `/` (wichtig!)

**4. Twenty Worker**
- Klone den "Twenty Server" Service
- Ändere nur den Start Command (siehe unten)

#### Schritt 2: Service-Konfiguration

**Twenty Server:**
```bash
# Build Settings
Dockerfile Path: packages/twenty-docker/twenty/Dockerfile
Root Directory: /

# Deploy Settings
Port: 3000
Health Check Path: /healthz
Start Command: node dist/src/main (Standard, wird vom Dockerfile übernommen)
```

**Twenty Worker:**
```bash
# Build Settings (identisch zum Server)
Dockerfile Path: packages/twenty-docker/twenty/Dockerfile
Root Directory: /

# Deploy Settings
Start Command: yarn worker:prod

# Environment Variables (zusätzlich)
DISABLE_DB_MIGRATIONS=true
DISABLE_CRON_JOBS_REGISTRATION=true
```

#### Schritt 3: Environment Variables setzen

Siehe `env-checklist.md` für die vollständige Liste.

**Minimale Konfiguration für beide Services (Server + Worker):**

```bash
# Core
SERVER_URL=https://your-app.railway.app
APP_SECRET=<generiere-mit-openssl-rand-base64-32>

# Database (automatisch von Railway gesetzt wenn PostgreSQL Plugin genutzt)
PG_DATABASE_URL=${{Postgres.DATABASE_URL}}

# Redis (automatisch von Railway gesetzt wenn Redis Plugin genutzt)
REDIS_URL=${{Redis.REDIS_URL}}

# Storage
STORAGE_TYPE=local
STORAGE_LOCAL_PATH=/app/packages/twenty-server/.local-storage

# Frontend Build
REACT_APP_SERVER_BASE_URL=https://your-app.railway.app
```

**Für Production empfohlen:**
- Setze `STORAGE_TYPE=s3` und konfiguriere S3-kompatiblen Storage
- Aktiviere SSL (Railway macht das automatisch)
- Setze starkes `APP_SECRET` (mindestens 32 Zeichen)

---

## 🌍 Quick Start (English)

### Option 1: Use Community Template (Recommended)

The fastest method is using the pre-built Railway template:

1. **Click the deploy button:**
   ```
   https://railway.com/deploy/nAL3hA
   ```

2. **Railway automatically creates:**
   - ✅ Twenty Server (Main application on port 3000)
   - ✅ Twenty Worker (Background job processor)
   - ✅ PostgreSQL 16 Database
   - ✅ Redis Cache & Session Store

3. **Configure environment variables** (see below)

4. **Wait for deployment** (approx. 5-10 minutes first time)

5. **Open your app:** Find the URL in Railway Dashboard under "Settings" → "Public Domain"

---

### Option 2: Manual Setup

If you want more control:

#### Step 1: Create Services

Create the following services in your Railway project:

**1. PostgreSQL Database**
- Go to "New" → "Database" → "Add PostgreSQL"
- Railway automatically generates `DATABASE_URL`

**2. Redis**
- Go to "New" → "Database" → "Add Redis"
- Railway automatically generates `REDIS_URL`

**3. Twenty Server**
- Go to "New" → "GitHub Repo" → Select your Twenty CRM fork
- **Dockerfile Path:** `packages/twenty-docker/twenty/Dockerfile`
- **Root Directory:** `/` (important!)

**4. Twenty Worker**
- Clone the "Twenty Server" service
- Only change the start command (see below)

#### Step 2: Service Configuration

**Twenty Server:**
```bash
# Build Settings
Dockerfile Path: packages/twenty-docker/twenty/Dockerfile
Root Directory: /

# Deploy Settings
Port: 3000
Health Check Path: /healthz
Start Command: node dist/src/main (default, inherited from Dockerfile)
```

**Twenty Worker:**
```bash
# Build Settings (identical to server)
Dockerfile Path: packages/twenty-docker/twenty/Dockerfile
Root Directory: /

# Deploy Settings
Start Command: yarn worker:prod

# Environment Variables (additional)
DISABLE_DB_MIGRATIONS=true
DISABLE_CRON_JOBS_REGISTRATION=true
```

#### Step 3: Set Environment Variables

See `env-checklist.md` for the complete list.

**Minimum configuration for both services (Server + Worker):**

```bash
# Core
SERVER_URL=https://your-app.railway.app
APP_SECRET=<generate-with-openssl-rand-base64-32>

# Database (automatically set by Railway when using PostgreSQL plugin)
PG_DATABASE_URL=${{Postgres.DATABASE_URL}}

# Redis (automatically set by Railway when using Redis plugin)
REDIS_URL=${{Redis.REDIS_URL}}

# Storage
STORAGE_TYPE=local
STORAGE_LOCAL_PATH=/app/packages/twenty-server/.local-storage

# Frontend Build
REACT_APP_SERVER_BASE_URL=https://your-app.railway.app
```

**Recommended for Production:**
- Set `STORAGE_TYPE=s3` and configure S3-compatible storage
- Enable SSL (Railway does this automatically)
- Set strong `APP_SECRET` (minimum 32 characters)

---

## 🔧 Wichtige Hinweise / Important Notes

### Automatische Migrationen
Das Dockerfile führt automatisch Datenbank-Migrationen beim Start aus. Dies geschieht nur beim **Server**-Service, nicht beim Worker (daher `DISABLE_DB_MIGRATIONS=true`).

The Dockerfile automatically runs database migrations on startup. This only happens on the **Server** service, not the Worker (hence `DISABLE_DB_MIGRATIONS=true`).

### Ephemeres Dateisystem
Railway verwendet ein ephemeres Dateisystem. Für Production solltest du `STORAGE_TYPE=s3` nutzen und einen S3-kompatiblen Object Storage konfigurieren (z.B. AWS S3, Cloudflare R2, Backblaze B2).

Railway uses an ephemeral filesystem. For production, use `STORAGE_TYPE=s3` and configure S3-compatible object storage (e.g., AWS S3, Cloudflare R2, Backblaze B2).

### SSL/HTTPS
Railway bietet automatisch HTTPS für deine Public Domain. Verwende immer `https://` in `SERVER_URL` und `REACT_APP_SERVER_BASE_URL`.

Railway automatically provides HTTPS for your public domain. Always use `https://` in `SERVER_URL` and `REACT_APP_SERVER_BASE_URL`.

### Health Checks
Twenty CRM bietet einen Health-Check-Endpoint unter `/healthz`. Railway kann diesen nutzen, um die Service-Gesundheit zu überwachen.

Twenty CRM provides a health check endpoint at `/healthz`. Railway can use this to monitor service health.

---

## 🐛 Troubleshooting

### Problem: "Failed to connect to database"

**Lösung / Solution:**
- Überprüfe, ob `PG_DATABASE_URL` korrekt gesetzt ist
- Stelle sicher, dass der PostgreSQL-Service läuft
- Check if `PG_DATABASE_URL` is correctly set
- Ensure the PostgreSQL service is running

### Problem: "Redis connection failed"

**Lösung / Solution:**
- Überprüfe `REDIS_URL`
- Stelle sicher, dass der Redis-Service läuft
- Check `REDIS_URL`
- Ensure the Redis service is running

### Problem: Build schlägt fehl / Build fails

**Lösung / Solution:**
- Stelle sicher, dass der **Root Directory** auf `/` gesetzt ist (nicht auf `packages/twenty-docker`)
- Überprüfe den **Dockerfile Path**: `packages/twenty-docker/twenty/Dockerfile`
- Make sure **Root Directory** is set to `/` (not `packages/twenty-docker`)
- Check **Dockerfile Path**: `packages/twenty-docker/twenty/Dockerfile`

### Problem: Worker-Service startet nicht

**Lösung / Solution:**
- Überprüfe den Start Command: `yarn worker:prod`
- Stelle sicher, dass `DISABLE_DB_MIGRATIONS=true` gesetzt ist
- Check start command: `yarn worker:prod`
- Ensure `DISABLE_DB_MIGRATIONS=true` is set

### Problem: Frontend lädt nicht

**Lösung / Solution:**
- Überprüfe `REACT_APP_SERVER_BASE_URL` - muss die Railway Public Domain sein
- Stelle sicher, dass `SERVER_URL` identisch ist
- Check `REACT_APP_SERVER_BASE_URL` - must be the Railway public domain
- Ensure `SERVER_URL` is identical

---

## 📚 Weitere Ressourcen / Additional Resources

- [Twenty CRM Self-Hosting Docs](../packages/twenty-docs/developers/self-hosting/docker-compose.mdx)
- [Railway Documentation](https://docs.railway.app/)
- [Twenty CRM GitHub](https://github.com/twentyhq/twenty)
- [Twenty CRM Community Discord](https://discord.gg/twenty)

---

## 🔐 Sicherheit / Security

**Wichtig / Important:**

1. **Ändere** `APP_SECRET` sofort nach dem Deployment
2. **Nutze** starke Passwörter für PostgreSQL
3. **Aktiviere** nur die Auth-Provider, die du benötigst
4. **Setze** `STORAGE_TYPE=s3` für Production
5. **Überprüfe** regelmäßig die Logs auf ungewöhnliche Aktivitäten

1. **Change** `APP_SECRET` immediately after deployment
2. **Use** strong passwords for PostgreSQL
3. **Enable** only the auth providers you need
4. **Set** `STORAGE_TYPE=s3` for production
5. **Check** logs regularly for unusual activity

---

## 📝 Nächste Schritte / Next Steps

Nach erfolgreichem Deployment:

1. ✅ Öffne deine Twenty CRM Instanz
2. ✅ Erstelle deinen ersten Admin-Account
3. ✅ Konfiguriere zusätzliche Settings im Admin Panel (wenn `IS_CONFIG_VARIABLES_IN_DB_ENABLED=true`)
4. ✅ Richte Auth-Provider ein (Google, Microsoft, etc.)
5. ✅ Konfiguriere Email-Versand (SMTP)
6. ✅ Lade dein Team ein!

After successful deployment:

1. ✅ Open your Twenty CRM instance
2. ✅ Create your first admin account
3. ✅ Configure additional settings in the admin panel (if `IS_CONFIG_VARIABLES_IN_DB_ENABLED=true`)
4. ✅ Set up auth providers (Google, Microsoft, etc.)
5. ✅ Configure email sending (SMTP)
6. ✅ Invite your team!
