# UAV parts: Claude Code plugin

Search an evidence-graded catalogue of drone parts from Claude Code:

- parts that meet your requirements,
- drop-in alternatives,
- checks on a whole build or BOM,
- country of origin, and makers' NDAA, Blue UAS, ITAR and export statements,
- where to buy.

Every answer says which evidence it rests on, and anything unconfirmed stays visible.

**Invite-only.** The plugin talks to a private server, `parts.ironhoard.com`, and needs a personal access key. Without one it does nothing.

## Install

1. Open the invite link you were sent, press **Get my key**, and save the key (it starts with `uavp_`). It's shown once.
2. In Claude Code:

   ```
   /plugin marketplace add titasorsha/uav-parts-plugin
   /plugin install uav-parts@uav-parts
   ```

   When it asks for your **access key**, paste it. It's kept in your computer's secure credential store.
3. In `/plugin` → **Marketplaces**, choose `uav-parts` and **Enable auto-update** so you get fixes as they're released.
4. Start a new session and ask, for example: *"a 6S 4-in-1 ESC not made in China"*.

To change your key later: `/plugin` → **Installed** → `uav-parts` → **Configure options**.

## What's sent where

- Your questions are turned into tool calls, for example a category and requirements, and sent to `parts.ironhoard.com` with your key.
- **The server logs** who called which tool, when, and how long it took. It doesn't log what you searched for. Logs are kept for 30 days.
- **Results pages** are private links that stop working after 30 days.
- **Problem reports.** If you tell Claude an answer is wrong, it can record your words with the `report_problem` tool so the maintainer can fix it.

## Updates

Catalogue and tool updates happen on the server, so you get them straight away. Changes to this plugin (the skill's instructions) arrive through auto-update and take effect in your next session, or straight away with `/reload-plugins`.
