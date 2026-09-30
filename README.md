# MFA plugins for Claude

The `mfa` marketplace. It holds one plugin, **mfa**, which lets an MFA member
work their account from Claude — on claude.ai, the Claude desktop and mobile
apps, Cowork and Claude Code.

## Install

1. In Claude, open **Customize → Plugins → Add → Add marketplace** and enter
   `pragmaticgrowth/mfa-plugins`.
2. Install **mfa**.
3. On the plugin's **Connectors** tab, connect **MFA** and sign in with your
   MFA email and password.

In Claude Code: `claude plugin marketplace add pragmaticgrowth/mfa-plugins`,
then `claude plugin install mfa@mfa`.

MFA is invite-only: the connector only lets in members of an MFA deployment.
See [plugins/mfa/README.md](plugins/mfa/README.md) for what the plugin does
and what data it sends.

This repository is published from the private MFA monorepo
(`plugin/` → here, by `scripts/publish-plugin.sh`); edit it there.
