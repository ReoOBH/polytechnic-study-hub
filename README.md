# Polytechnic Study Hub — Student Setup

**Mac & Windows | No ZIP, coding, or project download needed.**

You need **Docker Desktop** and the **read-only token** sent privately by the owner.

## First Time (5 Steps)

### 1. Open Docker Desktop

Install [Docker Desktop](https://www.docker.com/products/docker-desktop/) if you haven't already, then **start it**.

- **Mac:** Open **Terminal**.
- **Windows:** Open **PowerShell**.

Use the same commands on both.

### 2. Log in to download the app

Copy and run this command **exactly as shown**:

```bash
docker login ghcr.io -u ReoOBH
```

When you see `Password:`, **paste the token sent by the owner** and press **Enter**.

- Use the provided token, **not your GitHub password**.
- Nothing will appear as you paste. This is normal.
- You should see **`Login Succeeded`**.

**This is only for downloading the app. You will create your own Study Hub account in Step 4.**

### 3. Install and start the app

**Copy this entire command** into Terminal / PowerShell and press **Enter**:

```bash
docker run -d --name studyhub-student --restart unless-stopped -p 127.0.0.1:8081:8080 -v polytechnic-study-hub-student:/data -e NODE_ENV=production -e PORT=8080 -e HOST=0.0.0.0 -e DATABASE_PATH=/data/polytechnic-study-hub.sqlite ghcr.io/reoobh/polytechnic-study-hub:v5.2.2-private-midterm-20261031
```

Docker will download and start the app. The first download may take a few minutes. A long ID means the container started.

### 4. Create YOUR Study Hub account

Run this command **once**:

```bash
docker exec -it studyhub-student node scripts/provision-student.mjs
```

Follow the questions on screen:

1. **Username:** Choose your own (3–32 characters). Use lowercase letters, numbers, `.`, `_`, or `-`. Example: `student_01`.
2. **Password:** Choose 12–128 characters. Include at least **three** of these: lowercase English letters, uppercase English letters, numbers, or symbols. Do not use one character repeated by itself. Your typing is hidden.
3. **Confirm password:** Enter the same password again.
4. **Recovery codes:** The app will show **10 codes**. Save them somewhere private. They are shown only once and help you reset a forgotten password.

**Your Study Hub username and password are your own. They are not the GitHub download credentials.**

### 5. Open the website

Open **http://localhost:8081** and sign in with the Study Hub account you just created.

You're ready to study!

---

## Next Time — How to Open the App

1. Start **Docker Desktop**.
2. Open **Terminal** (Mac) or **PowerShell** (Windows).
3. Run:

   ```bash
   docker start studyhub-student
   ```

4. Open **http://localhost:8081**.

**Or:** In Docker Desktop → **Containers**, find `studyhub-student` and press **Start ▶**.

If the app is already running, just open the website. **Do not repeat the install or account-creation steps.** Your progress is saved.

## How to Update

**Wait until the owner sends a new immutable image digest.** Keep Docker Desktop open. Updating does not create another account. The backup contains private account and study data, so keep it on your computer.

### Mac — Terminal

Replace the digest placeholder with the exact value sent by the owner. Run the first block while the current app is still running:

```bash
set -e
IMAGE='ghcr.io/reoobh/polytechnic-study-hub@sha256:PASTE_RELEASE_DIGEST'
OLD_IMAGE=$(docker inspect --format '{{.Config.Image}}' studyhub-student)
DATA_VOLUME=$(docker inspect --format '{{range .Mounts}}{{if eq .Destination "/data"}}{{.Name}}{{end}}{{end}}' studyhub-student)
[ -n "$OLD_IMAGE" ] || { echo "Current image not found. Stop and contact the owner."; exit 1; }
[ -n "$DATA_VOLUME" ] || { echo "Study data volume not found. Stop and contact the owner."; exit 1; }
echo "Using existing data volume: $DATA_VOLUME"
STAMP=$(date +%Y%m%d-%H%M%S)
BACKUP="$HOME/StudyHub-Backups/$STAMP"
mkdir -m 700 -p "$BACKUP"
docker exec studyhub-student node scripts/backup-auth-data.mjs "/tmp/studyhub-backup-$STAMP"
docker cp "studyhub-student:/tmp/studyhub-backup-$STAMP/." "$BACKUP/"
docker run --rm --mount "type=bind,source=$BACKUP,target=/backup,readonly" "$OLD_IMAGE" node scripts/restore-auth-data.mjs --check /backup
docker pull "$IMAGE"
docker buildx imagetools inspect "$IMAGE" # Digest must match the owner’s release notice.
```

Before continuing, confirm the displayed **Digest** matches the owner’s release notice. Then recreate the container with the same data volume and settings:

```bash
docker stop studyhub-student
docker rm studyhub-student
docker run -d --name studyhub-student --restart unless-stopped -p 127.0.0.1:8081:8080 --mount "type=volume,source=$DATA_VOLUME,target=/data" -e NODE_ENV=production -e PORT=8080 -e HOST=0.0.0.0 -e DATABASE_PATH=/data/polytechnic-study-hub.sqlite "$IMAGE"
for i in {1..30}; do HEALTH=$(docker inspect --format '{{.State.Health.Status}}' studyhub-student 2>/dev/null); [ "$HEALTH" = healthy ] && break; sleep 2; done
[ "$HEALTH" = healthy ] || { echo "App did not become healthy. Keep the backup and contact the owner."; exit 1; }
curl -fsS http://localhost:8081/healthz
```

### Windows — PowerShell

Replace the digest placeholder with the exact value sent by the owner. Run the first block while the current app is still running:

```powershell
$Image = 'ghcr.io/reoobh/polytechnic-study-hub@sha256:PASTE_RELEASE_DIGEST'
$OldImage = docker inspect --format '{{.Config.Image}}' studyhub-student
$DataVolume = docker inspect --format '{{range .Mounts}}{{if eq .Destination "/data"}}{{.Name}}{{end}}{{end}}' studyhub-student
if (-not $OldImage) { throw 'Current image not found. Stop and contact the owner.' }
if (-not $DataVolume) { throw 'Study data volume not found. Stop and contact the owner.' }
$Stamp = Get-Date -Format yyyyMMdd-HHmmss
$Backup = Join-Path $HOME "StudyHub-Backups\$Stamp"
New-Item -ItemType Directory -Path $Backup -Force | Out-Null
docker exec studyhub-student node scripts/backup-auth-data.mjs "/tmp/studyhub-backup-$Stamp"
if ($LASTEXITCODE -ne 0) { throw 'Backup failed. Do not update.' }
docker cp "studyhub-student:/tmp/studyhub-backup-$Stamp/." "$Backup/"
if ($LASTEXITCODE -ne 0) { throw 'Backup copy failed. Do not update.' }
docker run --rm --mount "type=bind,source=$Backup,target=/backup,readonly" $OldImage node scripts/restore-auth-data.mjs --check /backup
if ($LASTEXITCODE -ne 0) { throw 'Backup verification failed. Do not update.' }
docker pull $Image
if ($LASTEXITCODE -ne 0) { throw 'Image download failed. The current app is still running.' }
docker buildx imagetools inspect $Image
if ($LASTEXITCODE -ne 0) { throw 'Could not verify the image. Do not update.' }
```

Confirm the displayed **Digest** matches the owner’s release notice. Then recreate the container:

```powershell
docker stop studyhub-student
if ($LASTEXITCODE -ne 0) { throw 'Could not stop the app. Do not continue.' }
docker rm studyhub-student
if ($LASTEXITCODE -ne 0) { throw 'Could not replace the old container. Keep the data volume.' }
docker run -d --name studyhub-student --restart unless-stopped -p 127.0.0.1:8081:8080 --mount "type=volume,source=$DataVolume,target=/data" -e NODE_ENV=production -e PORT=8080 -e HOST=0.0.0.0 -e DATABASE_PATH=/data/polytechnic-study-hub.sqlite $Image
if ($LASTEXITCODE -ne 0) { throw 'Container start failed. Keep the backup and contact the owner.' }
$Health = ''
for ($i = 0; $i -lt 30; $i++) { $Health = docker inspect --format '{{.State.Health.Status}}' studyhub-student 2>$null; if ($Health -eq 'healthy') { break }; Start-Sleep -Seconds 2 }
if ($Health -ne 'healthy') { throw 'App did not become healthy. Keep the backup and contact the owner.' }
curl.exe -fsS http://localhost:8081/healthz
if ($LASTEXITCODE -ne 0) { throw 'Health check failed. Keep the backup and contact the owner.' }
```

Sign in with your existing account and check your saved progress. If health or data checks fail, do not provision another account or delete any volume. Keep the backup and contact the owner for rollback help. The update keeps the existing `/data` volume; do not use `docker volume rm`, `docker volume prune`, or `docker rm -v`.

*The update sequence was tested on macOS with disposable Docker data. Native Windows PowerShell acceptance is still pending.*

## Forgot Your Study Hub Password?

With Docker Desktop running, use:

```bash
docker exec -it studyhub-student node scripts/recover-student-account.mjs
```

Enter your Study Hub username, one unused recovery code, and a new password that follows the 12–128 character and three-character-type rules above. Save the new recovery codes.

## Having Trouble?

- **`unauthorized` / `denied`:** Check the token the owner sent you, then repeat Step 2.
- **Container already exists:** The app is already installed. Use `docker start studyhub-student` instead of Step 3.
- **Website won't open:** Make sure Docker Desktop is running; then start the container.
- **Port 8081 is in use:** Close the other app using that port, or ask the owner for help.

**Important:** Never delete the Docker volume mounted at `/data` by `studyhub-student` — it stores your account and study progress. New installations normally name it `polytechnic-study-hub-student`. Do not share your token, password, recovery codes, or the private study materials.

*Mac setup has been tested. Native Windows testing is still pending.*
