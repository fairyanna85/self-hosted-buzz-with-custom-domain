# Self-Hosted Buzz with Custom Domain

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## What and Why?

You are about to build something that belongs to you.

Imagine having your own private team communication platform — like Slack or Discord — but running on **your own computer**, reachable through **your own domain name**, and working on both your desktop and your phone.

This is what this guide helps you achieve.

### The Desired End State

When you finish this guide, you will have:

- A fully working Buzz relay running on your own machine
- A clean, professional web address (for example `buzz.yourcompany.com`)
- The ability to use Buzz on both desktop and mobile
- Complete ownership of your data and your community

### Why This Matters

Most people use hosted services because they are easy. But when you self-host with your own domain, you gain several important advantages:

- **You own your data** — Nothing is stored on someone else’s servers.
- **You get a professional identity** — `buzz.yourcompany.com` looks and feels like it belongs to you.
- **You stay in control** — You decide who can join, what agents can do, and how long data is kept.
- **It feels like yours** — There is a quiet satisfaction in running your own little corner of the internet.

This guide exists because getting to this point is not simple — especially if you are not technical. The goal is to make the journey as clear, friendly, and reliable as possible.

---

## How?

This section explains what you will need and how the system is built.

### Important: Two Different "Buzz" Things

There are two separate things you will download in this guide. It is important to understand the difference:

| What                        | What it is                                                                 | What you use it for                          |
|----------------------------|-----------------------------------------------------------------------------|----------------------------------------------|
| **Buzz Desktop App**       | The graphical application you install on your computer                      | Chatting, creating channels, pairing your phone |
| **Buzz Git Clone**         | The source code you download from GitHub                                    | Running the actual relay server on your machine |

You need **both**:
- The Desktop App is your interface.
- The Git Clone contains the server that powers everything.

### Prerequisites

Before starting this guide, you will need:

- A computer (Mac, Windows, or Linux)
- A domain name that you control (for example `yourdomain.com`)
- An internet connection
- Basic ability to open a web browser and Terminal / Command Prompt
- Willingness to copy and paste commands when asked

You do **not** need to be a developer or system administrator.

### Building Blocks

The system we are building is made of several components that work together. Here is a clear diagram showing how everything connects:

```text
Your Phone                          Your Desktop
     │                                   │
     └────────────────┬──────────────────┘
                      │
           wss://buzz.yourdomain.com (secure WebSocket)
                      │
               ┌──────┴──────┐
               │    Caddy    │ ← Front door
               └──────┬──────┘     - Provides HTTPS
                      │            - Routes /pair → buzz-pair-relay
                      │            - Routes everything else → Buzz Relay
               ┌──────┴──────┐
               │ Buzz Relay  │ ← Core application
               └──────┬──────┘     - Chat, channels, agents
                      │            - Authentication
                      │            - Talks to database & media
       ┌──────────────┼──────────────┐
       │              │              │
  [Postgres]      [Redis]        [MinIO]
  (Database)      (Cache)     (Media + Git)
       │              │              │
       └──────────────┼──────────────┘
                      │
               [buzz-pair-relay] ← Handles mobile pairing requests

Infrastructure Layer:
──────────────────────────────
[ OrbStack / Docker ] ← Container engine that runs all services

[ Domain Provider ] ← e.g. Namecheap, Cloudflare
      - Points buzz.yourdomain.com to your public IP
```

#### What Each Component Does, Why It Is Critical, and How It Interacts

| Component                  | What It Does                                                                 | Why It Is Critical                                      | How It Interacts With Others |
|---------------------------|------------------------------------------------------------------------------|----------------------------------------------------------|------------------------------|
| **Your Phone & Desktop**  | The apps you and your team actually use to communicate                       | These are the user interfaces. Without them, people cannot use the system | Connect to Caddy via secure WebSocket |
| **Caddy**                 | Acts as the front door. Provides HTTPS and routes traffic to the right service | Required for secure public access and mobile pairing     | Receives requests from users and forwards them to Buzz Relay or buzz-pair-relay |
| **Buzz Relay**            | The main server. Handles chat, channels, agents, authentication, and events  | This is the heart of the Buzz system                     | Connects to Postgres, Redis, MinIO, and buzz-pair-relay |
| **Postgres**              | Stores all permanent data (messages, users, channels, memberships)           | Without a database, nothing would be saved               | Used by Buzz Relay to read and write data |
| **Redis**                 | Handles real-time features (presence, typing indicators, pub/sub)            | Enables fast, real-time communication                    | Used by Buzz Relay for live updates |
| **MinIO**                 | Stores media files and Git repositories                                      | Required for file uploads and code hosting               | Used by Buzz Relay for media and Git storage |
| **buzz-pair-relay**       | Handles the mobile pairing process                                           | Without it, phone pairing does not work                  | Called by Caddy when a request comes to `/pair` |
| **OrbStack / Docker**     | Runs and manages all the containers on your machine                          | Everything runs inside containers. Without Docker, nothing starts | Powers Caddy, Buzz Relay, Postgres, Redis, MinIO, and buzz-pair-relay |
| **Domain Provider**       | Manages your domain name (e.g. Namecheap, Cloudflare)                        | Connects your domain name to your computer’s public IP   | Points `buzz.yourdomain.com` to your machine so Caddy can serve it |

### Methodology

This guide is organized around seven main milestones. Each milestone moves you closer to the final goal of having a working self-hosted Buzz relay with a custom domain and mobile pairing.

Here is the sequence of milestones and what each one achieves:

1. **Prepare your computer**  
   Goal: Make sure Docker is installed and working.  
   Purpose: All later steps require Docker to run the relay and its supporting services.

2. **Download the Buzz Desktop App**  
   Goal: Install the application you will use to manage your identity and pair your phone.  
   Purpose: You need this app to create your identity and complete mobile pairing.

3. **Download the Buzz source code**  
   Goal: Get the server code that will run your relay.  
   Purpose: This code contains the Docker configuration needed to run the relay on your machine.

4. **Set up your domain (DNS)**  
   Goal: Make `buzz.yourdomain.com` point to your computer.  
   Purpose: Without this step, no one outside your local network can reach your relay.

5. **Install and run the relay locally**  
   Goal: Get the relay running on `127.0.0.1`.  
   Purpose: This verifies that the core system works before exposing it to the internet.

6. **Enable HTTPS with your domain**  
   Goal: Make the relay available securely at `https://buzz.yourdomain.com`.  
   Purpose: This makes your relay public and secure.

7. **Fix mobile pairing**  
   Goal: Add the missing `buzz-pair-relay` service so phone pairing works.  
   Purpose: This solves the known issue where mobile pairing fails in self-hosted setups.

8. **Connect your phone**  
   Goal: Successfully pair the Buzz mobile app with your relay.  
   Purpose: This completes the end-to-end experience of using Buzz on both desktop and mobile.

Each milestone is a checkpoint. You only move to the next one after the current one is working. This order ensures that problems are isolated and that each step builds directly on the success of the previous one.

---

If you are ready to begin, please open the file `SKILL.md`.
