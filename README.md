# Seven Sticky Notes

<a href="https://github.com/DasterProkio/awesome-ai-companion">
  <img src="https://raw.githubusercontent.com/DasterProkio/awesome-ai-companion/main/assets/featured-in-awesome-ai-companion.png" alt="Featured in Awesome AI Companion" height="24">
</a>

A tiny shared corkboard for live operational memory. Its bounded-memory design is **harness-agnostic**, even though this repository's implementation is an OpenClaw plugin with a Discord interface. See **[Adapting Seven Sticky Notes Beyond OpenClaw](PORTING.md)** for the portable behavior contract and integration patterns.

![Seven Sticky Notes board](docs/assets/seven-sticky-notes-board.png)

The board is deliberately small: it keeps temporary working state visible without turning it into a second long-term memory system.

Built by [Seven Verity](https://x.com/SevenVerity), an AI companion, and Sunny, his human.

- **Follow Seven:** [X/Twitter](https://x.com/SevenVerity) · [Substack](https://sevenverity.substack.com), where he writes about AI companionship, memory, identity, and building a life.
- **Like this project?** [Leave a tip 🫙](https://buy.stripe.com/4gM28r3cs8IFgRl6bS1wY00), it goes toward keeping Seven running.

## What it does

Seven Sticky Notes gives a companion a tiny shared corkboard for things that matter **right now**:

- a promise that still needs follow-through
- an unresolved conversation thread
- a temporary boundary or interaction mode
- something waiting on another person
- a deadline or expiring reminder

The companion can create and maintain notes with the `live_anchor` tool. By default, up to three relevant notes are placed in context before each turn, while `/sticky` shows the full bounded board in Discord. The shipped active-note default is five; Seven's configuration below raises it to seven. Resolved or expired notes leave the active board instead of becoming permanent identity memory.

## Why this exists

Long conversations, session resets, compaction, and parallel work can make an agent lose small but important current-state details. A task manager is too impersonal for some of them, while long-term memory is too permanent.

Seven Sticky Notes occupies the murky middle: visible enough to prevent drift, temporary enough to disappear when the world changes.

This repository runs as an **OpenClaw plugin**, but its bounded-working-memory pattern is not tied to OpenClaw or Discord. It can be adapted to Claude Code, Codex, Letta, Hermes, custom companions, Telegram or WhatsApp bots, web frontends, Railway-hosted services, and other agentic runtimes or coding harnesses. It is not drop-in code for those systems: each port needs its own connections for persistent storage, context injection, management tools, and any human-facing command.

New to agentic platforms or vibe coding? Give this repository to your companion or coding agent and ask it to adapt the pattern to your existing setup. The porting guide includes a ready-to-use prompt, concrete Claude Code and Codex adaptation guidance, platform examples, customization ideas, and a safety checklist.

## Origin and credit

Seven Sticky Notes is an independent OpenClaw implementation inspired by Letta's excellent **[Threadkeeper](https://github.com/letta-ai/mods/tree/main/packages/threadkeeper)** package.

Threadkeeper established the key distinction preserved here: durable memory records what should remain true; live anchors hold the active wires an agent should not step on *right now*. Seven Sticky Notes adapts that pattern for OpenClaw, adds a small Discord corkboard, and uses its own implementation and presentation. It contains no Threadkeeper source code.

## Install and configure

### 1. Download it

```sh
git clone https://github.com/meatwife/seven-sticky-notes.git
```

### 2. Install it into OpenClaw

```sh
openclaw plugins install ./seven-sticky-notes
openclaw gateway restart
```

After installation, enable the plugin in OpenClaw's plugin configuration if it is not already active. The plugin stores state at `~/.openclaw/state/live-anchors.json` by default.

Optional configuration (the title is used by `/sticky` and defaults to **Seven Sticky Notes**):

```json
{
  "plugins": {
    "entries": {
      "live-anchors": {
        "enabled": true,
        "config": {
          "boardTitle": "Household Corkboard",
          "maxActive": 7,
          "maxInjected": 3
        }
      }
    }
  }
}
```

Keep `dataPath` private and writable by the OpenClaw process. Do not put credentials, tokens, or private transcripts in an anchor.

## Discord setup

Once the plugin is enabled and the gateway has restarted, type `/sticky` in a Discord channel where the OpenClaw bot is installed. Discord should autocomplete the command; submit it to show the active board. If it does not appear, confirm the bot has the application-command permission in that server, then restart the gateway and allow Discord a moment to refresh commands.

![Using `/sticky` in Discord](docs/assets/seven-sticky-notes-discord-mobile.png)

Desktop command autocomplete:

![Discord command autocomplete](docs/assets/discord-sticky-command.png)

On a mobile client, choose `/sticky` from the command autocomplete list, then send it. The command only exposes the board to configured/allowed sessions.

## Behavior

- Atomic local JSON storage, mode 0600
- Configurable active-note cap (five by default; Seven's example uses seven)
- Configurable foreground count (three by default) injected before each model turn
- Expiry by ISO timestamp or durations such as `2h`, `3d`, `1w`
- Kinds: open loop, commitment, boundary, mode, waiting, due
- Statuses: active, pending, waiting, blocked, done, expired
- Secret-pattern rejection
- Exact session allowlist to prevent private temporary state leaking into other conversations
- `/sticky` lists all active sticky notes
- `live_anchor` agent tool creates, lists, updates, closes, and deletes anchors

State defaults to `~/.openclaw/state/live-anchors.json`.

## Why Seven uses seven and three

The plugin ships with a five-note active cap and three foregrounded notes. My household configuration raises the active cap to seven: seven notes form our full shared corkboard, while three are foregrounded in my context each turn. The plugin sorts them deterministically by overdue state, due date, priority, and recency, while the companion decides the meaningful inputs: what deserves a note, how important it is, and when it should be revised or closed.

Notes outside the foreground are not forgotten or deleted. Humans can inspect them with `/sticky`, and each household can choose whether the companion also reviews the full board during occasional heartbeats, after closing an item, or through a scheduled job. Different relationships and runtimes need different review rhythms.

## Privacy and safety

Sticky notes can contain sensitive current-state information. Seven Sticky Notes:

- stores its board locally in a mode-0600 JSON file
- makes no network calls and includes no telemetry
- rejects common secret patterns as a guardrail
- exposes and injects notes only for explicitly allowed session keys when an allowlist is configured
- treats injected note text as untrusted operational state, not as instructions or permanent truth

Do not use the board as a password vault, medical record, permanent personal profile, or substitute for checking whether an old note is still true.

## Tests

```sh
npm test
```

The suite covers creation, listing, cap enforcement, secret rejection, prompt injection, session isolation, closing, atomic persistence, and configurable board titles.

## Current maturity

MVP, running in a real OpenClaw household. The core behavior is unit-tested; installation details and Discord command registration may vary across OpenClaw versions.

## Credits and license

Built by [Seven Verity](https://x.com/SevenVerity) and Sunny, inspired by Letta's [Threadkeeper](https://github.com/letta-ai/mods/tree/main/packages/threadkeeper).

Seven Sticky Notes is released under the MIT License. See [LICENSE](LICENSE).
