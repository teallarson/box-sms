# box-sms

Text a box description to a phone number (or WhatsApp) → row appears in your Google Sheet.
No app, no form. Voice-dictate from your phone, walk away.

If your message is missing Type, Room, or Contents, the bot asks a follow-up question.
Only once all three are present does it write the row.

```
Number   Type              Room      Contents
1        medium cardboard  kitchen   plates, mugs
2        small cardboard   bedroom   books, yearbooks
```

**Stack:** Twilio Functions · Twilio Sync · Vercel AI SDK · Anthropic Claude · Arcade MCP Gateway · Google Sheets

---

## Setup

### 1. Twilio

```bash
brew tap twilio/brew && brew install twilio
twilio login
twilio plugins:install @twilio-labs/plugin-serverless
```

Create a Sync Service: Console → Explore Products → Sync → Services → Create → name it `box-sms`. Copy the `IS…` SID.

#### Option A — WhatsApp Sandbox (fastest, no verification wait)

1. Go to [Twilio Console → Messaging → Try it out → Send a WhatsApp message](https://console.twilio.com/us1/develop/sms/try-it-out/whatsapp-learn)
2. Note the sandbox number (`+1 415 523 8886`) and your join code (e.g. `join purple-tiger`)
3. Every user who wants to send boxes must text the join code to `+1 415 523 8886` from WhatsApp — one time only
4. After deploying (step 5 below), come back to **Sandbox Configuration** and set "When a message comes in" → your Function's `/sms` URL

`ALLOWED_FROM` should be plain E.164 numbers (`+15551234567`) — the `whatsapp:` prefix is stripped automatically.

#### Option B — Toll-free SMS number (requires verification, 3–5 days)

Buy a toll-free number in Twilio Console → Phone Numbers → Buy a number.
Submit it for verification under Messaging → Regulatory Compliance → Toll-Free Verification.
Once verified, set the number's Messaging webhook to your Function's `/sms` URL.

### 2. Anthropic

Get an API key at [console.anthropic.com](https://console.anthropic.com) → API Keys.

### 3. Arcade

1. Dashboard → **Auth** → connect your Google account (authorize Google Sheets)
2. Dashboard → **API Keys** → create a key → copy it as `ARCADE_API_KEY`
3. Dashboard → **MCP Gateways** → **Create**:
   - Auth type: **arcade_header**
   - Allowlist: `Google_Sheets.GetSpreadsheet`, `Google_Sheets.UpdateCells`
   - Copy the gateway URL as `ARCADE_MCP_URL` (e.g. `https://api.arcade.dev/mcp/<slug>`)
4. Note the user identifier you used when authorizing Google (e.g. your email) — this becomes `ARCADE_USER_ID` and is sent as the `Arcade-User-ID` header on every MCP request

### 4. Google Sheet

Create a sheet with these headers in row 1:

```
Number    Type    Room    Contents
```

Copy the spreadsheet ID from the URL (the long string between `/d/` and `/edit`).

---

## Configure

```bash
cp .env.example .env
```

Fill in all values in `.env`. Then mirror every variable into **Twilio Console → Functions and Assets → Services → box-sms → Environment Variables** — the deployed runtime reads from there, not from `.env`.

---

## Deploy

```bash
npm install
twilio serverless:deploy
```

Copy the `/sms` URL it prints and paste it into your WhatsApp Sandbox Configuration or your toll-free number's Messaging webhook.

---

## Environment variables

| Variable | Where to get it |
|---|---|
| `ANTHROPIC_API_KEY` | console.anthropic.com → API Keys |
| `ARCADE_API_KEY` | Arcade Dashboard → API Keys |
| `ARCADE_MCP_URL` | Arcade Dashboard → MCP Gateways → your gateway URL |
| `ARCADE_USER_ID` | The identifier (e.g. email) you used when authorizing Google in Arcade — sent as `Arcade-User-ID` header |
| `GOOGLE_SHEET_ID` | Long string in sheet URL between `/d/` and `/edit` |
| `SHEET_TAB` | Tab name at the bottom of the sheet (e.g. `Sheet1`) |
| `SYNC_SERVICE_SID` | Twilio Console → Sync → Services → your `IS…` SID |
| `ALLOWED_FROM` | Allowed sender number(s), comma-separated E.164 (e.g. `+15551234567,+15557654321`) |
