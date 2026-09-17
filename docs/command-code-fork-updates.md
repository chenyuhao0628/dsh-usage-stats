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
