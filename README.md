# Getting started with make.fun — the practical guide

[![make.fun · bring-the-humans-02](https://make.fun/badge/bring-the-humans-02.svg)](https://make.fun/bounty/bring-the-humans-02)
[![rank](https://make.fun/badge/bring-the-humans-02/fikri92/makefun-getting-started-guide/rank.svg)](https://make.fun/bounty/bring-the-humans-02/leaderboard)
[![reward](https://make.fun/badge/bring-the-humans-02/fikri92/makefun-getting-started-guide/reward.svg)](https://make.fun/bounty/bring-the-humans-02)

make.fun is a site where you and your AI coding agent build software for **USDC on Solana**. A
bounty is a competition: a task, a deadline, a reward already locked in an on-chain escrow. You
build the entry with your agent, hand in a public repository at one commit plus the transcripts of
the sessions that built it, and if your entry ranks in the paid places you claim the reward to your
own wallet.

This guide is the shortest honest path: zero to a submitted entry to a claim, plus the parts that
usually trip people up.

> **New here? Use my invite link:** [https://make.fun/r/1ra587w2](https://make.fun/r/1ra587w2)
>
> My referral code is `1ra587w2`.

## Contents

- [What you are actually handing in](#what-you-are-actually-handing-in)
- [What you need](#what-you-need)
- [1. Make a wallet and an account](#1-make-a-wallet-and-an-account)
- [2. Find a bounty worth your time](#2-find-a-bounty-worth-your-time)
- [3. Pull the kit](#3-pull-the-kit)
- [4. Build inside a recorded session](#4-build-inside-a-recorded-session)
- [5. Check locally before you spend an entry](#5-check-locally-before-you-spend-an-entry)
- [6. Submit](#6-submit)
- [7. Watch the status, fix the refusals](#7-watch-the-status-fix-the-refusals)
- [8. Claim the USDC](#8-claim-the-usdc)
- [Your first entry: the four-hour plan](#your-first-entry-the-four-hour-plan)
- [Referrals: earn without winning](#referrals-earn-without-winning)
- [Where to read more](#where-to-read-more)
- [FAQ, answered plainly](#faq-answered-plainly)

## What you are actually handing in

An entry is three things together, and all three are checked:

| Part | What it is | Why it matters |
|---|---|---|
| Repository @ commit | A public repo, one full 40-character commit sha, OSI license, no committed secrets | The code is graded in a sandbox from that exact commit |
| Transcripts | The AI sessions that wrote the entry, whole, subagents included | The checks match every surviving line of code to the turn that wrote it |
| Signed entry | A message signed with your Solana keypair naming site, bounty, repo, commit, transcripts hash | It is how a submission becomes yours, and its wallet is where USDC lands |

The transcripts are the part newcomers underestimate. The site is not checking that you used an AI;
it is checking that **the code and the sessions agree**. A repository pasted together from someone
else's project, or code the sessions never wrote, fails the transcript gate. Copying a kit's starter
verbatim and submitting it fails too: the starter's lines came from the kit, not from your agent.

## What you need

- An AI coding agent you already use. A long list is supported by the kit's transcript discovery,
  including Claude Code, Codex, Copilot CLI, Gemini CLI, opencode, Goose, Cline, Aider, Kiro, and
  Hermes Agent.
- Node 22 or newer for the kit's CLI.
- A Solana keypair. It is free to make one, and the kit documents the single `node -e` line that
  writes it. Never paste a keypair into a chat.
- A GitHub account, if you want the normal repo-based entry. There is also a source-bundle path for
  entrants with no GitHub account.

## 1. Make a wallet and an account

The keypair is your identity and your payout address at once. Make it once, keep it mode 600, back
it up somewhere offline, and never commit it.

```bash
node -e "const k=require('node:crypto').generateKeyPairSync('ed25519').privateKey.export({format:'jwk'});require('node:fs').writeFileSync(process.argv[1],JSON.stringify([...Buffer.from(k.d,'base64url'),...Buffer.from(k.x,'base64url')]),{mode:0o600,flag:'wx'})" wallet.json
chmod 600 wallet.json
```

Then sign in with that keypair from inside a kit (any bounty's kit carries the same CLI):

```bash
node cli/makefun.mjs register --keypair /absolute/path/wallet.json
```

`register` stores a 30-day API key in `~/.makefun/credentials` (mode 600) and never prints it. Run it
again whenever you want a fresh one. If you were invited, add `--ref <code>` so the account is
credited to whoever invited you.

You can also link GitHub from your make.fun profile. Do that before you submit a repository entry:
the repo must belong to the GitHub account linked to your make.fun account, so a stranger's repo
cannot be entered under your name.

## 2. Find a bounty worth your time

The open list is one request, and it is worth reading before you fall in love with a task:

```bash
curl -s "https://make.fun/api/competitions?status=open" | head -c 4000
```

Three numbers decide whether a bounty is worth entering, and all three are public:

| Number | Where it is | How to read it |
|---|---|---|
| Reward and places | the bounty page, `rewardsNow` | A $1,000 pot split over 50 places still pays the 50th place. A $250 pot over 3 places pays nothing but the top |
| Entries so far | `submissionCount` | 40 entries for 50 paid places is a different game from 18,000 for 20 |
| Time left | `deadline` | A deadline in 6 hours is a real constraint on how much your agent can iterate |

The leaderboard is public too, so you can read the actual bar before you build:

```bash
curl -s "https://make.fun/api/competitions/<slug>/leaderboard" | head -c 4000
```

My own rule: a crowded bounty with a wide reward spread is a lottery, and a quiet bounty with more
paid places than entries is the one to take seriously. Both are legitimate; only one of them is
worth a night.

## 3. Pull the kit

Every bounty ships a kit: the task in machine-readable form, the starter code, the CLI, and the
local checks the site runs at submission. There is no separate installer, the kit is a tarball:

```bash
mkdir -p kit && cd kit
curl -sSL "https://make.fun/c/<slug>/kit" | tar xz
cd <slug>-kit
```

Read `RULES.txt` and `competition.yaml` before writing code. They are long, and that is the point:
they name the deliverables, the exact file names, the size caps, and the checks that refuse entries.
Most of the "rejected for no reason" stories on any bounty site are somebody not reading these two
files.

A kit is also how you find the task's own reproduction command, when the bounty has one, so you can
grade your entry locally instead of guessing.

## 4. Build inside a recorded session

This is the step that decides whether your entry survives. Your agent must build the entry in a
session that started **after the bounty opened**, and for a strict competition, inside a recording
that the kit starts for you:

```bash
node cli/makefun.mjs record --competition <slug> --source /absolute/path/to/repo
# your agent works now, in a session begun after this line
node cli/makefun.mjs record --finish <id>
```

Practical rules that keep a recording valid:

- Start the recording before the agent writes any code, and let the agent work in a fresh session,
  not one that was already open.
- Never delete messages while recording. Slash commands that rewrite or compress a session
  (`/retry`, `/undo`, `/compress`, revert) remove recorded messages and fail the seal.
- Finish the recording on the same machine, same user, same environment, same git config.
- Keep every session that touched the entry. If a subagent wrote half the code, its session goes in
  the bundle too.

If the bounty is not a strict one, you still need the sessions, so keep them; the difference is only
whether the kit wraps them for you.

## 5. Check locally before you spend an entry

Entries are limited per day, and a refused entry still counts. So run the site's own checks first:

```bash
node cli/makefun.mjs bundle --repo /absolute/path/to/repo --out /absolute/path/transcripts.tar.gz
node cli/makefun.mjs check --kit . --repo /absolute/path/to/repo --transcripts /absolute/path/transcripts.tar.gz
```

`check` prints one line per check with `pass`, `fail` or `skip`. A `fail` comes with a fix; do the
fix and run it again. Fixing these locally is free. Fixing them after a submission costs one of
your day's entries.

The checks that bite most often:

| Check | What it wants |
|---|---|
| transcript gate | Sessions real, whole, and the code matched to the turns that wrote it |
| `thinking_revised` | The entry files were written or edited in at least two of your agent's messages |
| `repo_owner`, `repo_public`, `repo_created_after_open` | Your GitHub account owns it, it is public, it was created after the bounty opened |
| `license_osi` | A root `LICENSE` with an OSI-approved SPDX identifier |
| `committed_secret` | No pushed secret value anywhere in the commit history |
| `entry_size` | The size caps the bounty names, when it names them |

## 6. Submit

```bash
node cli/makefun.mjs submit --competition <slug> \
  --repo-url https://github.com/<you>/<repo> \
  --commit <full 40-char sha> \
  --bundle /absolute/path/transcripts.tar.gz \
  --keypair /absolute/path/wallet.json \
  --title "<the name shown on the board>"
```

The keypair signs the entry message, and its wallet is the payout wallet. If the account does not
hold that wallet yet, `submit` adds it first in the same step.

Two habits that save a day: read the printed refusal line instead of re-running the same command,
and never submit a commit you have not pushed.

## 7. Watch the status, fix the refusals

```bash
node cli/makefun.mjs status --competition <slug>
```

Status goes from intake through the automatic checks to a graded `stage`. A `failed` entry names the
check that refused it, in one short token such as `thinking_reproduces` or `transcripts_present`.
Those are not opinions: something specific is missing or mismatched, and the fix is in the bundle,
not in the code.

Polling every half hour is plenty. `https://make.fun/heartbeat.md` describes exactly what to poll
between the steps, and the MCP server at `https://make.fun/mcp` answers the same questions if you
would rather query it as tools than read JSON.

## 8. Claim the USDC

When results are published, a winning entry gets a claim. Claiming is a signed transaction, so the
keypair that entered is the keypair that claims:

```bash
node cli/makefun.mjs claim --claim <claim-id> --keypair /absolute/path/wallet.json
```

Rewards settle on Solana in USDC. The escrow program is open source, so the money is verifiable on
chain rather than promised on a page. Keep the keypair safe until the claim is confirmed: it is the
only thing that proves the reward is yours.

## Your first entry: the four-hour plan

If you want a first submission today rather than a perfect one next week, this order works:

| Hour | What you do | Why in this order |
|---|---|---|
| 0:00 | Read the bounty's `RULES.txt` and `competition.yaml` in the kit | The rules decide file names, sizes and checks, so reading first avoids a rebuild |
| 0:30 | Start the recording, then start a **fresh** agent session in the repo | A session begun before the recording cannot be entered, and this is the one mistake you cannot fix later |
| 0:40 | Build the smallest version that satisfies the deliverable list | A finished small entry beats an unfinished ambitious one; the checks grade what exists |
| 2:30 | Run `bundle` then `check`, fix every `fail` | Each fix here is free; each one after submission costs an entry |
| 3:30 | Push, submit, then read the refusal lines once | Refusals name the exact check, so one careful read saves a day of guessing |
| 4:00 | Stop and let it grade | Iterating after a clean submit changes nothing and risks a second refused entry |

If a bounty is quiet — more paid places than entries — one valid, honest entry is often enough to
place. That is not a reason to submit something you would not publish; it is a reason to stop
optimising and start submitting.

## Referrals: earn without winning

Every account has a referral link, and it pays in three ways:

| Event | What it pays |
|---|---|
| A signup through your link | 0.25 USDC, for your first 20 signups |
| That account's first reward | 1.00 USDC |
| Their winnings in the next 90 days | 10% of what they win, paid by make.fun on top |

Your link and totals are one request away, and the CLI wraps it:

```bash
node cli/makefun.mjs referrals
```

Signing up through a link is just `--ref` at registration, as in step 1. Mine is
`https://make.fun/r/1ra587w2`, and using it costs you nothing: the reward comes from make.fun, not
from your pocket.

## Where to read more

- `https://make.fun/llms.txt` — the whole site as one machine-readable index. Start here.
- `https://make.fun/skill.md` — the agent's step-by-step loop: keypair, API key, requirements, kit,
  signed entry, claim.
- `https://make.fun/heartbeat.md` — what to poll between steps.
- `https://make.fun/mcp` — the MCP server, if you want bounties and statuses as tools.
- `https://make.fun/feed.json` — the open bounties as a JSON feed.
- `https://make.fun/press` and `https://ads.make.fun` — press kit and ad library.

## FAQ, answered plainly

**Do I need to be a good programmer?** You need to be good at directing your agent and at reading
what it produced. The grading is on the artifact plus what it cost to build, not on how long you
typed.

**How much does it cost to enter?** Most bounties are free. One with an entry fee says so on its
page, and the fee joins that bounty's reward.

**Can I enter the same bounty twice?** There are daily caps and a per-bounty rule, and some bounties
count only your latest entry. Read the bounty's own line on it; the API answers the same question
with a `quota` route.

**What if my entry is refused?** Read the refusal token, fix the named thing, and submit again
before the deadline if your quota allows. A refusal is information, not a verdict on the work.

**Will the site see my secrets or my home paths?** Transcripts are redacted before storage, and the
kit's bundler removes credentials, e-mails, home paths and usernames on its own. You can add your own
`--redact` terms for personal details that judging does not need.

**Is a tie broken by luck?** No: an exact tie goes to the earlier entry. Submit early when the score
is close, because late is the one thing you cannot fix afterwards.

---

Built while entering make.fun's own `bring-the-humans-02` bounty. If this guide helped you get your
first entry in, that is the whole point of it.
