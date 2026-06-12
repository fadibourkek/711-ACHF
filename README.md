# 🛡️ 711 ACHF — Discord Server Management Platform

A full-featured multi-tenant SaaS platform for Discord server management, featuring:
- **Moderation Engine** (warn/ban/kick/mute/jail with case tracking)
- **Automation Rules Engine** (IF condition → THEN action builder)
- **Ticket System** (support/appeal/report with SLA tracking)
- **Analytics Dashboard** (real-time charts and stats)
- **AI Module** (Premium: moderation suggestions, ticket replies)
- **Music Module** (Pro: YouTube queue per server)
- **Subscription System** (Free/Pro/Premium via Stripe)

---

## 📁 Project Structure

```
711-achf/
├── backend/          # Node.js + Express REST API + WebSocket
│   └── src/
│       ├── auth/         # Discord OAuth2 + JWT
│       ├── moderation/   # Cases, warnings, punishments
│       ├── tickets/      # Ticket system + SLA
│       ├── automation/   # Rules engine + automod
│       ├── analytics/    # Stats + event tracking
│       ├── subscriptions/ # Stripe integration
│       ├── music/        # Music session management
│       ├── ai/           # OpenAI integration
│       └── database/     # PostgreSQL + Redis
├── bot/              # Discord.js bot service
│   └── src/
│       ├── commands/     # Slash commands
│       ├── events/       # Discord event handlers
│       └── handlers/     # Automod, rules, music
├── dashboard/        # Next.js 14 web dashboard
│   └── src/
│       ├── app/          # App Router pages
│       └── components/   # React components
├── infra/            # Docker, Nginx, scripts
│   ├── docker/       # init.sql, docker configs
│   └── nginx/        # Reverse proxy config
└── docker-compose.yml
```

---

## 🚀 Quick Start

### Prerequisites
- Docker & Docker Compose
- Node.js 20+
- A Discord Application (bot token + OAuth2 credentials)

### 1. Clone & Configure

```bash
git clone <your-repo>
cd 711-achf
cp .env.example .env
```

### 2. Fill in `.env`

```env
# Discord (Required)
DISCORD_CLIENT_ID=1512569464065888286
DISCORD_CLIENT_SECRET=your_discord_client_secret
DISCORD_BOT_TOKEN=your_bot_token
DISCORD_REDIRECT_URI=http://localhost:3000/auth/callback

# JWT (generate with: openssl rand -hex 32)
JWT_SECRET=your_32_char_secret_here
JWT_REFRESH_SECRET=another_32_char_secret

# Stripe (optional for payments)
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
```

### 3. Discord Bot Setup

1. Go to [Discord Developer Portal](https://discord.com/developers/applications)
2. Select your application (ID: `1512569464065888286`)
3. Under **Bot**: Enable `MESSAGE CONTENT INTENT`, `SERVER MEMBERS INTENT`, `PRESENCE INTENT`
4. Under **OAuth2 → Redirects**: Add `http://localhost:3000/auth/callback`
5. Copy your bot token to `.env`

### 4. Start with Docker

```bash
# Development
docker-compose -f docker-compose.dev.yml up

# Production
docker-compose up -d
```

Services start on:
- **Dashboard**: http://localhost:3000
- **Backend API**: http://localhost:3001
- **PostgreSQL**: localhost:5432
- **Redis**: localhost:6379

### 5. Manual Setup (without Docker)

```bash
# Install all dependencies
npm install

# Start PostgreSQL + Redis (via Docker)
docker-compose up postgres redis -d

# Run database migrations
cd backend && npm run migration:run

# Start backend
cd backend && npm run dev

# Start bot (new terminal)
cd bot && npm run dev

# Start dashboard (new terminal)
cd dashboard && npm run dev
```

---

## 🗄️ Database Schema

Key tables:
| Table | Purpose |
|-------|---------|
| `users` | Discord accounts |
| `servers` | Discord guilds (tenants) |
| `server_members` | Per-server user roles |
| `moderation_cases` | All mod actions with full audit |
| `warnings` | Warning records with severity |
| `punishment_thresholds` | Auto-escalation rules |
| `automation_rules` | IF→THEN rule engine |
| `automod_config` | Per-server automod settings |
| `tickets` | Support ticket records |
| `ticket_messages` | Ticket conversation threads |
| `subscriptions` | Stripe subscription records |
| `analytics_events` | Raw event stream |
| `analytics_daily` | Aggregated daily stats |
| `audit_logs` | Full system audit trail |
| `music_sessions` | Active music queues |

---

## 🔌 API Reference

### Auth
| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/auth/discord` | Start Discord OAuth flow |
| GET | `/api/auth/discord/callback` | OAuth callback |
| GET | `/api/auth/me` | Current user profile |
| POST | `/api/auth/refresh` | Refresh access token |
| POST | `/api/auth/logout` | Invalidate session |

### Moderation
| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/moderation/servers/:id/cases` | List cases (paginated) |
| POST | `/api/moderation/servers/:id/warn` | Issue warning |
| POST | `/api/moderation/servers/:id/ban` | Ban user |
| POST | `/api/moderation/servers/:id/kick` | Kick user |
| POST | `/api/moderation/servers/:id/mute` | Timeout user |
| POST | `/api/moderation/servers/:id/jail` | Jail (role isolate) user |
| GET | `/api/moderation/servers/:id/users/:uid/history` | User mod history |
| GET/POST | `/api/moderation/servers/:id/thresholds` | Auto-punish thresholds |

### Tickets
| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/tickets/servers/:id/tickets` | List tickets |
| POST | `/api/tickets/servers/:id/tickets` | Create ticket |
| GET | `/api/tickets/:ticketId` | Get ticket + messages |
| POST | `/api/tickets/:ticketId/reply` | Add reply |
| PATCH | `/api/tickets/:ticketId/status` | Update status |
| PATCH | `/api/tickets/:ticketId/assign` | Assign staff |

### Automation
| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/automation/servers/:id/rules` | List rules |
| POST | `/api/automation/servers/:id/rules` | Create rule |
| PUT | `/api/automation/servers/:id/rules/:rid` | Update rule |
| DELETE | `/api/automation/servers/:id/rules/:rid` | Delete rule |
| GET/PUT | `/api/automation/servers/:id/automod` | Automod config |

### Analytics
| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/analytics/servers/:id/overview` | Dashboard stats |
| GET | `/api/analytics/servers/:id/activity` | Activity chart (7/30/90d) |
| GET | `/api/analytics/servers/:id/moderation-stats` | Mod breakdown |
| GET | `/api/analytics/servers/:id/top-moderators` | Top mods |

### Subscriptions
| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/subscriptions/plans` | Available plans |
| POST | `/api/subscriptions/checkout` | Create Stripe checkout |
| POST | `/api/subscriptions/portal` | Stripe billing portal |
| POST | `/api/subscriptions/webhook/stripe` | Stripe webhook |

---

## 🤖 Bot Slash Commands

| Command | Description | Permission |
|---------|-------------|------------|
| `/warn <user> <reason>` | Warn a member | Moderate Members |
| `/ban <user> <reason> [duration]` | Ban a member | Ban Members |
| `/kick <user> <reason>` | Kick a member | Kick Members |
| `/mute <user> <minutes> <reason>` | Timeout a member | Moderate Members |
| `/history <user>` | View mod history | Moderate Members |
| `/ticket <subject> [category]` | Create a ticket | Everyone |

---

## 🆓 Everything is Free

All features are unlocked for every server — no plans, no tiers, no credit card.

| Feature | Status |
|---------|--------|
| Unlimited automation rules | ✅ Free |
| Full moderation (warn/ban/kick/mute/jail) | ✅ Free |
| Advanced AutoMod | ✅ Free |
| Ticket system + SLA tracking | ✅ Free |
| Full analytics dashboard | ✅ Free |
| Music module | ✅ Free |
| AI module (requires OPENAI_API_KEY) | ✅ Free |
| Unlimited log entries | ✅ Free |
| API access | ✅ Free |
| Real-time WebSocket dashboard | ✅ Free |

---

## 🔒 Security

- **JWT** access tokens (7d) + refresh tokens (30d) stored in Redis
- **Rate limiting** per IP (global) and per endpoint (auth)
- **Role-based access**: Owner > Admin > Moderator > Member
- **Plan gating**: Feature middleware checks subscription tier
- **Bot secret**: Internal bot↔backend communication protected
- **Webhook verification**: Stripe signature validation
- **Audit logs**: All actions logged with actor, target, changes
- **Input validation**: Zod schemas on all API inputs

---

## 🛠️ Development

### Adding a new feature module:

1. **Backend**: Add routes in `backend/src/<module>/<module>.routes.ts`
2. **Register**: Import in `backend/src/main.ts`
3. **Bot**: Add handler in `bot/src/handlers/`
4. **Dashboard**: Add page in `dashboard/src/app/dashboard/[serverId]/<module>/`

### Running tests:

```bash
cd backend && npm test
```

### Environment variables reference:

See `.env.example` for full list with descriptions.

---

## 📦 Tech Stack

| Layer | Technology |
|-------|-----------|
| Backend API | Node.js, Express, TypeScript |
| Bot | Discord.js v14, TypeScript |
| Dashboard | Next.js 14, React, Tailwind CSS |
| Database | PostgreSQL 16 |
| Cache/Queue | Redis 7 |
| Real-time | Socket.io (WebSocket) |
| Auth | Discord OAuth2, JWT |
| Payments | Stripe |
| AI | OpenAI GPT-4 |
| Infrastructure | Docker, Nginx |
| Animation | Framer Motion |
| Charts | Recharts |

---

## 📄 License

MIT License — see LICENSE file.

---

Built with ❤️ for the Discord community.
