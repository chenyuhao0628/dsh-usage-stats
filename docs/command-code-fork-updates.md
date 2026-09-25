# Command Code fork maintenance

This fork is based on `Ychris12138/dsh-usage-stats` and keeps the local
Command Code account adapter in the `command-code-adapter` branch.

## Install the fork in DSH

```bash
dsh plugin --profile web add "github:chenyuhao0628/dsh-usage-stats#command-code-adapter"
```

The package keeps the upstream name `@ychris12138/dsh-usage-stats`, so the
existing bundle entry remains unique. Keep the Command Code monitor in the
profile patch, using only the credential reference:

```yaml
- id: usage-stats
  name: "@ychris12138/dsh-usage-stats"
  config:
    monitors:
      command-code:
        adapter: command-code
        usageBaseURL: https://api.commandcode.ai
        credentialRef: COMMAND_CODE_API_KEY
```

## Upgrade the installed fork

After the branch is updated on GitHub, the normal plugin manager can update
the same dependency:

```bash
dsh plugin --profile web update "@ychris12138/dsh-usage-stats"
```

If the manager cannot resolve the Git branch from the existing lockfile, use
the explicit fork spec once; it preserves the fork source for later updates:

```bash
dsh plugin --profile web add "github:chenyuhao0628/dsh-usage-stats#command-code-adapter"
```

Then restart `dsh web` and hard-refresh the browser.

## Fork-local changes on top of upstream v0.3.3

### Removed: the upstream OrcaRouter sponsored integration

Upstream shipped an OrcaRouter sponsor surface: a synthetic `+ OrcaRouter`
entry in the provider picker, the sponsorship label suffix, a loopback
`GET|POST /api/usage-stats/integrations/orcarouter` route that wrote a provider
preset into the harness settings, the `orcarouter` balance adapter, and the README
sponsorship badge. This fork removes all of it:

- deleted `lib/orcarouter.js`, its route and handler in `lib/index.js`, and the
  client-side picker injection plus its locale strings;
- dropped the `orcarouter` / `orcarouter-balance` entries from
  `lib/provider-identity.js`, `lib/accounts.js` and `lib/balance.js`;
- removed the matching tests, the README badge, the sponsor paragraph, the
  provider table row and the release-checklist items.

As a side effect the plugin no longer exposes any settings-write surface: it
registers nine read-only GET endpoints and no POST route at all.


### Added: 5-hour, weekly and monthly Command Code windows

`/alpha/billing/credits` reports `windowLimits.fiveHour` and
`windowLimits.weekly` plus the remaining monthly credits;
`/alpha/billing/subscriptions` reports the billing period; `/alpha/usage/summary`
reports what that period has spent. The account card renders three rows, each
with a refresh countdown:

| Row | used | cap | resets at |
| --- | --- | --- | --- |
| 5-hour (session) | `fiveHour.used` | `fiveHour.cap` | `fiveHour.resetAt` |
| weekly | `weekly.used` | `weekly.cap` | `weekly.resetAt` |
| monthly | period spend | period spend + remaining monthly credits | `currentPeriodEnd` |

- `resetAt: 0` means the rolling window has not started yet (the 5-hour window
  opens with the first request of the period). The adapter reports
  `started: false` and omits `resetsAt`, and the card shows
  `Starts on the first request` (zh: the localized equivalent) instead of a
  permanent `reset due` state.
- The monthly cap is derived, not reported: the alpha API never states the
  monthly allowance, so the row uses period spend plus the remaining monthly
  credits and carries `derived: true`. When `/alpha/usage/summary` is
  unavailable the monthly row is omitted and the other two rows are unaffected;
  every optional call fails soft.
- The card issues three requests per refresh instead of two.


## Sync the fork with upstream

Run these commands in a checkout of the fork. Use a merge instead of rebase
if the branch is already shared with other people.

```bash
git fetch upstream main
git switch command-code-adapter
git rebase upstream/main
npm test
git push --force-with-lease origin command-code-adapter
```

If the upstream changes the account protocol or the Command Code alpha
response shape, update the adapter fixtures and tests before pushing the
branch. Never edit the installed `node_modules` copy; plugin updates replace
that directory.
