# Self-Hosted Buzz with Custom Domain + Mobile Pairing

This skill walks you through setting up a self-hosted Buzz relay on your own machine using a custom domain and getting mobile pairing to work.

---

## Stage 0: Prepare Your Computer (Install Docker)

### Goal of this stage
We need to install Docker because Buzz runs inside Docker containers. Without Docker, we cannot run the relay.

### What you need before starting this stage
- A working computer (Mac, Windows, or Linux)
- An internet connection
- Ability to open a web browser and Terminal / Command Prompt

### Step 0.1: Check if Docker is already installed

**Why we do this:**  
We want to avoid installing Docker twice if it is already on your computer.

**What to do:**

1. Open **Terminal** (on Mac: press `Cmd + Space`, type "Terminal", and press Enter).

2. Type this command and press Enter:

   ```bash
   docker --version
   ```

**What you should see:**

- If you see something like `Docker version 27.0.0`, Docker is already installed. You can skip to Stage 1.
- If you see `command not found`, Docker is not installed. Continue to the next step.

**Danger:**  
If you see any other error, copy the full message and ask for help before continuing.

---

### Step 0.2: Download OrbStack (Docker for Mac)

**Why we do this:**  
OrbStack is currently the easiest way to install Docker on a Mac.

**What to do:**

1. Open your web browser.
2. Go to this address:

   ```
   https://orbstack.dev
   ```

3. Click the big **Download** button.

4. After the download finishes, open the downloaded file (it will be a `.dmg` file).

5. Drag the **OrbStack** icon into your **Applications** folder.

**How to know it worked:**  
You should now see OrbStack in your Applications folder.

---

### Step 0.3: Install and start OrbStack

**Why we do this:**  
We need to run OrbStack so Docker becomes available on your computer.

**What to do:**

1. Open **OrbStack** from your Applications folder (double-click it).

2. Follow the on-screen instructions to complete the installation.

3. When OrbStack finishes installing, it should be running (you may see its icon in the menu bar at the top of your screen).

**How to know it worked:**  
Open Terminal and run:

```bash
docker --version
```

You should now see a version number.

---

## Stage 1: Download the Buzz Desktop App

### Goal of this stage
We need the Buzz Desktop app to create our identity and to pair our phone later.

### What you need before starting this stage
- Docker is installed and working (from Stage 0)
- An internet connection

### Step 1.1: Go to the official download page

**Why we do this:**  
We need to download the correct version of the Buzz Desktop app.

**What to do:**

1. Open your web browser.
2. Go to this address:

   ```
   https://github.com/block/buzz/releases
   ```

**How to know you are in the right place:**  
You should see a list of releases with names like `desktop-v0.5.20`.

---

### Step 1.2: Download the correct version

**Why we do this:**  
Different computers need different versions of the app.

**What to do:**

1. Find the latest release that starts with `desktop-` (example: `desktop-v0.5.20`).

2. Download the correct file for your computer:

   - **Mac with Apple Silicon** (M1, M2, M3, M4 chips): Download the file ending in `_aarch64.dmg`
   - **Mac with Intel chip**: Download the file ending in `_x64.dmg`
   - **Windows**: Download the file ending in `_x64-setup.exe`

**Recommended version:** `desktop-v0.5.20`

**Danger:**  
Do not download server or mobile versions. Only download the **desktop** version.

---

### Step 1.3: Install the Buzz Desktop app

**Why we do this:**  
We need to install the app so we can use it.

**What to do:**

- **On Mac:** Open the downloaded `.dmg` file and drag the Buzz icon into your **Applications** folder.
- **On Windows:** Double-click the downloaded `.exe` file and follow the installer.

**How to know it worked:**  
You should see the Buzz app in your Applications folder (Mac) or Start Menu (Windows).

---

## Stage 2: Download the Buzz Source Code

### Goal of this stage
We need the Buzz source code because it contains the Docker files we will use to run the relay.

### What you need before starting this stage
- Docker is working
- Buzz Desktop app is installed

### Step 2.1: Create a folder for the source code

**Why we do this:**  
We want to keep the Buzz code in an organized place.

**What to do:**

Open **Terminal** and run:

```bash
mkdir -p ~/src
```

This creates a folder called `src` in your home directory.

---

### Step 2.2: Download the source code

**Why we do this:**  
We need to copy the official Buzz code from GitHub onto our computer.

**What to do:**

In Terminal, run these commands one by one:

```bash
cd ~/src
git clone https://github.com/block/buzz.git
```

**How to know it worked:**

Run this command:

```bash
ls ~/src/buzz
```

You should see a list of folders and files, including `deploy` and `README.md`.

**Danger:**  
If you see an error or the folder is empty, run the `git clone` command again.

---

## Stage 3: Set Up Your Domain (DNS)

### Goal of this stage
We need to make `buzz.yourdomain.com` point to your computer so people can reach your relay from the internet.

### What you need before starting this stage
- You own a domain name (example: `yourdomain.com`)

### Step 3.1: Find your public IP address

**Why we do this:**  
We need to know the address of your computer on the internet.

**What to do:**

In Terminal, run:

```bash
curl -s https://ipv4.icanhazip.com
```

Copy the number that appears.

---

### Step 3.2: Create a DNS record

**Why we do this:**  
We need to tell the internet that `buzz.yourdomain.com` belongs to your computer.

**What to do:**

1. Log into the website where you manage your domain.
2. Go to the DNS settings.
3. Create a new record with these values:

   - **Type**: `A`
   - **Name**: `buzz`
   - **Value**: Paste the IP address from Step 3.1

**How to know it worked:**  
After some time (5–60 minutes), visiting `https://dnschecker.org` and searching for `buzz.yourdomain.com` should show your IP address in most locations.

---

## Stage 4: Install the Relay Locally

### Goal of this stage
We want to run the Buzz relay on your computer for the first time.

### What you need before starting this stage
- Docker is working
- Buzz source code is downloaded
- You have a text editor (TextEdit on Mac is fine)

### Step 4.1: Go to the correct folder

**Why we do this:**  
We need to work in the folder that contains the Docker files.

**What to do:**

In Terminal, run:

```bash
cd ~/src/buzz/deploy/compose
```

---

### Step 4.2: Create your settings file

**Why we do this:**  
We need a file called `.env` that contains all our passwords and settings.

**What to do:**

In Terminal, run:

```bash
cp .env.example .env
```

This copies the example settings into a new file called `.env`.

---

### Step 4.3: Open the settings file

**Why we do this:**  
We need to edit the `.env` file.

**What to do:**

In Terminal, run:

```bash
open .env
```

This will open the file in TextEdit.

---

### Step 4.4: Fill in the basic settings

**Why we do this:**  
We need to tell Buzz what domain and ports to use while testing locally.

**What to do:**

In the `.env` file, find and change these lines to the following values:

```env
BUZZ_DOMAIN=127.0.0.1
RELAY_URL=ws://127.0.0.1:3000
BUZZ_MEDIA_BASE_URL=http://127.0.0.1:9000/media
BUZZ_CORS_ORIGINS=http://127.0.0.1:3000,http://127.0.0.1:9000
```

**How to save the file:**  
Press `Cmd + S` on Mac (or `Ctrl + S` on Windows), then close the file.

---

### Step 4.5: Generate passwords

**Why we do this:**  
We need secure passwords for the database and other services. Buzz does not provide these — we must create them ourselves.

**What to do:**

In Terminal, run:

```bash
for name in POSTGRES_PASSWORD REDIS_PASSWORD BUZZ_S3_ACCESS_KEY BUZZ_S3_SECRET_KEY; do
  sed -i.bak "s|^${name}=.*|${name}=$(openssl rand -hex 32)|" .env
done
rm -f .env.bak
```

---

### Step 4.6: Generate two important secrets

**Why we do this:**  
We need two long secret keys for the relay to work securely.

**What to do:**

Run these two commands and copy the results:

```bash
openssl rand -hex 32
openssl rand -hex 32
```

Paste the first result into `BUZZ_RELAY_PRIVATE_KEY` and the second into `BUZZ_GIT_HOOK_HMAC_SECRET` in the `.env` file.

**How to save:** Press `Cmd + S`.

---

### Step 4.7: Start the relay

**Why we do this:**  
We want to run the Buzz relay for the first time.

**What to do:**

In Terminal, run these two commands:

```bash
./run.sh config
./run.sh start
```

**How to check you did it right:**

```bash
curl -fsS http://127.0.0.1:3000/_liveness
```

You should see `ok` or `healthy`.

---

## Stage 5: Make It Public with HTTPS

### Goal of this stage
We want people to reach your relay using your real domain name with a secure connection.

### What you need before starting this stage
- The relay is running locally
- Your domain DNS is working

### Step 5.1: Update your domain settings

**Why we do this:**  
We need to change the settings from the test address (`127.0.0.1`) to your real domain.

**What to do:**

Open the `.env` file:

```bash
open ~/src/buzz/deploy/compose/.env
```

Change these lines to your real domain:

```env
BUZZ_DOMAIN=buzz.yourdomain.com
RELAY_URL=wss://buzz.yourdomain.com
BUZZ_MEDIA_BASE_URL=https://buzz.yourdomain.com/media
BUZZ_CORS_ORIGINS=https://buzz.yourdomain.com
```

**How to save:** Press `Cmd + S`.

---

### Step 5.2: Start the relay with HTTPS

**Why we do this:**  
We want to enable secure access over the internet.

**What to do:**

In Terminal, run:

```bash
cd ~/src/buzz/deploy/compose
BUZZ_COMPOSE_TLS=true ./run.sh start
```

**How to check you did it right:**

Open `https://buzz.yourdomain.com` in a browser.  
You should see a long page of text **and** a padlock icon.

---

## Stage 6: Fix Mobile Pairing

### Goal of this stage
The official setup is missing one important service. We need to add it so phone pairing works.

### What you need before starting this stage
- The relay is running with HTTPS
- Your domain is working

### Step 6.1: Add the pair service to compose.yml

**Why we do this:**  
We need to add the missing `buzz-pair-relay` service.

**What to do:**

Open the compose file:

```bash
open ~/src/buzz/deploy/compose/compose.yml
```

Find the `relay:` service and add this block **right after it** (before `postgres:`):

```yaml
  pair:
    image: ${BUZZ_IMAGE:-ghcr.io/block/buzz:main}
    entrypoint: ["/usr/local/bin/buzz-pair-relay"]
    environment:
      - BUZZ_PAIR_RELAY_BIND_ADDR=0.0.0.0:5000
    restart: unless-stopped
    networks:
      - buzz-net
```

**How to save:** Press `Cmd + S`.

---

### Step 6.2: Add the pairing URL to .env

**Why we do this:**  
We need to tell the relay where the pairing service is located.

**What to do:**

Open the `.env` file:

```bash
open ~/src/buzz/deploy/compose/.env
```

Add this line anywhere in the file:

```env
BUZZ_PAIRING_RELAY_URL=wss://buzz.yourdomain.com/pair
```

**How to save:** Press `Cmd + S`.

---

### Step 6.3: Create the Caddyfile

**Why we do this:**  
We need to tell Caddy how to route traffic for the `/pair` path.

**What to do:**

In Terminal, run:

```bash
cat > ~/src/buzz/deploy/compose/Caddyfile << "EOF"
{$BUZZ_DOMAIN} {
  encode zstd gzip
  reverse_proxy /pair* pair:5000
  reverse_proxy relay:3000
}
EOF
```

---

### Step 6.4: Start everything with the Caddy configuration

**Why we do this:**  
We need to start the relay with the new Caddyfile that includes the pairing route.

**What to do:**

In Terminal, run:

```bash
docker compose -f compose.yml -f compose.caddy.yml up -d
```

**How to check you did it right:**

```bash
curl -I https://buzz.yourdomain.com/pair
```

You should get `HTTP/2 400` or `HTTP/2 101` (not 404).

---

## Stage 7: Connect Your Phone

### Goal of this stage
We want to pair the Buzz mobile app with our self-hosted relay.

### What you need before starting this stage
- The relay is running with the pair service
- The `/pair` endpoint returns 400 or 101

### Step 7.1: Start pairing from Desktop

**Why we do this:**  
We need to generate a QR code to pair the phone.

**What to do:**

1. Open the **Buzz Desktop** app.
2. Go to **Settings → Mobile → Pair Mobile Device**.

---

### Step 7.2: Scan the QR code with your phone

**Why we do this:**  
We need to connect the phone to the relay.

**What to do:**

1. Open the **Buzz mobile app** on your phone.
2. Choose **Scan QR Code**.
3. Scan the code shown on your computer.
4. Confirm the 6-digit code on both devices.

**How to check you did it right:**  
Your phone should now show the same channels as your desktop.

---

## Troubleshooting

### Still getting 404 on `/pair`

```bash
cat > ~/src/buzz/deploy/compose/Caddyfile << "EOF"
{$BUZZ_DOMAIN} {
  encode zstd gzip
  reverse_proxy /pair* pair:5000
  reverse_proxy relay:3000
}
EOF

docker compose -f compose.yml -f compose.caddy.yml restart caddy
```

### “no such service: caddy”

Always start with both files:

```bash
docker compose -f compose.yml -f compose.caddy.yml up -d
```

### Certificate error in browser

Check DNS:

```bash
curl -I https://buzz.yourdomain.com
```

Check Caddy logs:

```bash
docker logs buzz-prod-caddy-1 --tail 30 | grep -E 'acme|certificate|challenge'
```

### Want to reset everything

```bash
cd ~/src/buzz/deploy/compose
docker compose down -v
./run.sh start
```

---

**Congratulations!** You have completed the full journey.

Thank you for going on this adventure with me. 🎉
