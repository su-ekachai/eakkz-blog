+++
title = "Automating My Daily HoYoLAB Check-In"
description = "How I handed a small daily gaming chore to a free GitHub Actions cron job, and the notification improvements I added on top of a neat open-source script."
date = 2026-07-01
updated = 2026-07-01
draft = false

[extra]
lang = "en"
toc = true
comment = false

[taxonomies]
tags = ["automation", "github-actions", "nodejs", "gaming"]
+++

## The daily tax

If you play any HoYoverse game (Genshin Impact, Honkai: Star Rail, Zenless Zone Zero), you know the drill. Every day HoYoLAB dangles a little pile of free rewards in front of you, and all you have to do is open the site and click one check-in button. Miss a day and that day's reward simply slips by. You get only a few make-up check-ins a month to reclaim what you skipped.

One button doesn't sound like much. But multiply it by a few games, times a couple of accounts, times *every single day forever*, and it turns into a small recurring tax on your attention. You either remember to pay it, or you quietly leak rewards. I wanted a third option: never think about it again.

This is a perfect job for a computer. It's the same request, on the same schedule, with the same inputs. The only reason I was still doing it by hand is that nobody had wired it up yet.

## What already existed

Before writing anything, I went looking, and found [`sglkc/hoyolab-auto-daily`](https://github.com/sglkc/hoyolab-auto-daily). It's exactly the kind of project I like: a single file, zero dependencies, no build step, no server to babysit. Just a Node script that hits HoYoLAB's check-in endpoint for each of your games.

The best part is *where* it runs. There's no machine to keep online. It rides a GitHub Actions cron job, entirely on GitHub's free tier. You commit your config as repository secrets, and a scheduled workflow wakes up once a day and runs the script for you.

### The inputs

Two environment variables drive the whole thing. Your login cookie and the list of games, each with one account per line:

```js
const cookies = process.env.COOKIE.split('\n').map(s => s.trim())
const games = process.env.GAMES.split('\n').map(s => s.trim())
```

The cookie is what proves you're you: the `ltuid_v2` / `ltoken_v2` pair you can pull from your browser after logging into HoYoLAB. It lives only in GitHub Secrets, never in the code.

### The endpoints

Each game has its own check-in endpoint and `act_id`. They all live in one map, which is what makes adding a game a one-line change:

```js
const endpoints = {
  zzz: 'https://sg-act-nap-api.hoyolab.com/event/luna/zzz/os/sign?act_id=e202406031448091',
  gi:  'https://sg-hk4e-api.hoyolab.com/event/sol/sign?act_id=e202102251931481',
  hsr: 'https://sg-public-api.hoyolab.com/event/luna/os/sign?act_id=e202303301540311',
  hi3: 'https://sg-public-api.hoyolab.com/event/mani/sign?act_id=e202110291205111',
  tot: 'https://sg-public-api.hoyolab.com/event/luna/os/sign?act_id=e202202281857121',
}
```

### The check-in itself

For each game, the script sends a `POST` (with a set of headers that mimic a real browser) and reads the API's `retcode`. HoYoLAB is friendly here: there's a code for "checked in" and a separate one for "already checked in today," and both mean success.

```js
const res = await fetch(url, { method: 'POST', headers, body })
const json = await res.json()

const code = String(json.retcode)
const successCodes = {
  '0': 'Successfully checked in!',
  '-5003': 'Already checked in for today',
}

if (code in successCodes) {
  log('info', game, successCodes[code])
  continue
}
```

### One run per account

At the bottom, the script simply loops over every account and checks each one in:

```js
for (const index in cookies) {
  const account = Number(index) + 1
  console.info(`\n-- CHECKING IN FOR ACCOUNT ${account} --`)
  messages.push({ type: 'header', string: `Account ${account}` })
  await run(cookies[index], games[index])
}
```

### The schedule

And the whole thing is kicked off once a day by the GitHub Actions cron:

```yaml
on:
  schedule:
    - cron: "0 22 * * *"  # 06:00 in UTC+8, every day
  workflow_dispatch:
```

That `workflow_dispatch` line is a nice touch: it lets you trigger a run manually from the Actions tab whenever you want to test it.

## What I added

The script did its job well. Where I wanted more was in knowing that it worked without opening GitHub every morning. So I focused on notifications.

### Telegram support

The project already knew how to post a summary to a Discord webhook. I added Telegram as a second, independent channel: set a bot token and a chat ID, and you get the same daily report as a Telegram message. It's a small function: build the report, then POST it to the Bot API's `sendMessage`.

```js
const text = formatReport()

const res = await fetch(`https://api.telegram.org/bot${telegramToken}/sendMessage`, {
  method: 'POST',
  headers: { 'content-type': 'application/json' },
  body: JSON.stringify({ chat_id: telegramChat, text })
})
```

Telegram and Discord are fully independent. You can enable either, both, or neither.

### A notification that's actually readable

The improvement I'm happiest with barely touches code. It's about emoji as signal versus noise.

The original message put a ✅ on *every* line: the header, each account, each game. When everything is marked "good," nothing stands out. Your eye has nothing to catch on, and a failure buried in the middle looks exactly like a success.

So I gave each emoji one job:

- `📅`: the date, on the header line
- `👤`: an account section
- `✅` / `❌`: the per-game result, and *only* that

And one more trick: if anything failed, the header flips from `📅` to `❌`. That matters because most chat apps show only the first line in a collapsed preview, so a bad run announces itself before you even open the message.

Both channels are fed by a single `formatReport()` function, so the Discord and Telegram messages can never drift apart:

```js
function formatReport() {
  const header = hasErrors
    ? `❌ Daily check-in · ${today} — error`
    : `📅 Daily check-in · ${today}`

  const body = messages
    .map(msg => msg.type === 'header'
      ? `\n👤 ${msg.string}`
      : `${icon[msg.type] ?? ''} ${msg.string}`)
    .join('\n')

  return `${header}\n${body}`
}
```

One small correctness detail hides in that `${today}`. The cron fires at 06:00 in UTC+8, but a server's default clock is UTC, where it's still the previous evening. Format the date naively and your 06:00 report gets stamped with *yesterday's* date. So the date is pinned to UTC+8 explicitly:

```js
const today = new Intl.DateTimeFormat('en-CA', { timeZone: 'Asia/Shanghai' }).format(new Date())
```

## Thanks

None of this would exist without [`sglkc/hoyolab-auto-daily`](https://github.com/sglkc/hoyolab-auto-daily), a clean, well-scoped script that got the hard parts right and left me a pleasant codebase to build on. Huge thanks to the maintainer for sharing it.

My version, with Telegram support and the notification redesign, lives here: [`su-ekachai/hoyolab-auto-daily`](https://github.com/su-ekachai/hoyolab-auto-daily).

The whole thing now runs itself at 6 a.m. and pings my phone when it's done. The best automation is the kind you forget is even there, right up until the one morning it tells you something went wrong.
