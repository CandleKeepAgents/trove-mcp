# Changelog

## 1.0.0 — Trove

- First release under the Trove name. This plugin succeeds `candlekeep-cowork` 0.7.2; behaviour and workflow are unchanged.
- Plugin renamed to `trove-cowork`, skill to `trove`, setup skill to `trove-setup`. Sub-agents are addressed as `trove-cowork:<agent>`.
- The bundled connector is now the `trove` server at `https://heytrove.ai/api/v1/mcp`, so tools are called as `trove:<tool>`.
- All links point at `heytrove.ai`; support is `support@heytrove.ai`.
- The skill states once that CandleKeep is the former name of Trove, so a user who says "CandleKeep" still reaches it.
- The skill's tool inventory is a table of tool → calling agent, so a new server tool is one row.
- Plan limits are no longer quoted as numbers in the skill or README (they differ by pricing cohort); the agent relays the limit the server returns.
- Home repository is now `CandleKeepAgents/trove-mcp`.
