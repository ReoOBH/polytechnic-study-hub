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

**Important:** Never delete the Docker volume `polytechnic-study-hub-student` — it stores your account and study progress. Do not share your token, password, recovery codes, or the private study materials.

*Mac setup has been tested. Native Windows testing is still pending.*
