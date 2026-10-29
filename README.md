# @pipeworx/nflverse

NFL play-by-play, player season stats, team game logs, schedules with results and betting lines, and weekly injury reports, 1999 to the current season, from nflverse.

Part of [Pipeworx](https://pipeworx.io) — an MCP gateway connecting AI agents to 1856+ live data sources. This is an independent, unofficial integration — not affiliated with, endorsed by, or published by the upstream provider.

## Tools

- `nfl_player_season_stats(player, season, season_type?)`: one player's season totals (passing, rushing, receiving, defense, kicking, returns, EPA, CPOE, target share, fantasy points). Look up by name or nflverse/GSIS id. Zero-valued fields are left out.
- `nfl_team_game_log(team, season)`: a team's games for one season. Each game has the opponent, home/away, final score, W/L, and passing, rushing, EPA, turnover, penalty, sack and kicking totals.
- `nfl_play_by_play(game_id, quarter?, play_type?, limit?)`: every play of one game, with down and distance, field position, play type, yards, the official play description, the players involved, score, EPA and win probability.
- `nfl_schedule(season, week?, team?)`: games for a season. Each game has its id, date, kickoff, scores, overtime, closing spread and total, venue, roof and surface, weather, starting QBs, coaches and referee.
- `nfl_injuries(season, team?, week?, player?)`: official weekly injury-report entries (game status and practice participation), latest week first. Covers 2009 to the current season.

Every response carries `data_as_of`, which is the last-modified time of the nflverse season file the answer came from, along with `source` and `licence`. A lookup that matches nothing returns `found: false` with a message. An unknown team code or a malformed game id is refused with an error that lists the valid forms.

## Auth

Keyless.

## Data sources

- nflverse, nflverse-data: play-by-play, player and team stats, schedules and injury reports, compiled by the nflverse project from NFL game data.

**Licence.** The nflverse-data repository is published under **Creative Commons Attribution 4.0 International (CC-BY-4.0)**. Reuse, including commercial reuse, is permitted with attribution. Every response carries `licence: "CC-BY-4.0 — data by nflverse (https://github.com/nflverse)"`. Each call reads the published season file at request time. This pack keeps no copy.

Notes for the next person:

- Team codes follow nflverse: `LA` is the Rams (`LAR` is accepted as an alias), `LAC`, `LV`, `WAS`. Pre-move franchises keep their old codes in old seasons (`OAK`, `SD`, `STL`).
- A season runs from September to the following February, so the 2024 season includes its playoff games played in 2025.
- Play-by-play is the large table, about 19 MB gzipped and 100 MB of CSV per season. It is streamed and filtered to one game. The rows are sorted by `game_id`, so the scan stops once it passes the requested game.
- The current season's files are refreshed by nflverse during the season, and `data_as_of` moves with them.

## Quick Start

Add to your MCP client (Claude Desktop, Cursor, Windsurf, etc.):

```json
{
  "mcpServers": {
    "nflverse": {
      "url": "https://gateway.pipeworx.io/nflverse/mcp"
    }
  }
}
```

### What this endpoint actually serves

`tools/list` at `https://gateway.pipeworx.io/nflverse/mcp` returns the tools in the table
above **plus the shared Pipeworx meta-tools** — `ask_pipeworx`,
`discover_tools`, `search_within`, `remember`/`recall` and the rest of the
gateway-wide set. So the tool count you see is larger than this table: a
single-pack endpoint currently lists roughly 30 shared tools alongside the
pack's own. The connection's `initialize` response states its exact scope, and
is the authoritative answer for a given day.

This is deliberate, not multiplexing by accident. The meta-tools are what let a
scoped connection answer a question this pack does not cover — via
`ask_pipeworx`, which routes across the whole catalog — without you adding a
second MCP server. There is currently no way to mount a pack endpoint without
them; if the extra schemas cost you more context than the routing is worth,
connect to the full gateway once rather than to several pack endpoints.

Or connect to the full Pipeworx gateway to get every pack's tools listed
directly, instead of just this one's:

```json
{
  "mcpServers": {
    "pipeworx": {
      "url": "https://gateway.pipeworx.io/mcp"
    }
  }
}
```

Both URLs reach the same gateway and the same 1856+ data sources. The
only difference is which pack's tools are listed **directly**; `ask_pipeworx`
reaches all of them from either one.

## No MCP client? Call it over HTTP

```bash
curl -X POST https://gateway.pipeworx.io/v1/tools/nfl_player_season_stats \
  -H 'Content-Type: application/json' \
  -d '{"player":"Joe Burrow","season":2024}'
```

No account needed for the first calls. Inspect any tool: `GET https://gateway.pipeworx.io/v1/tools/nfl_player_season_stats`. Find one: `POST https://gateway.pipeworx.io/v1/tools/search_packs` with `{"query":"..."}`.

## Run it from npm

`@pipeworx/mcp-nflverse` is a local stdio bridge to the Pipeworx gateway: your MCP client
starts it with `npx`, and it forwards every call to the same
`https://gateway.pipeworx.io/nflverse/mcp` endpoint shown above. It needs network access;
there is no offline mode.

```json
{
  "mcpServers": {
    "nflverse": {
      "command": "npx",
      "args": ["-y", "@pipeworx/mcp-nflverse"]
    }
  }
}
```

Without a key it calls anonymously, capped at 50 calls a day. A free account
raises that to 200 a day: get a key at https://pipeworx.io/signup and add
`"env": { "PIPEWORX_API_KEY": "<your key>" }` to the server entry.
The key is sent as a Bearer token, the same way a hosted connection sends it,
so calls are metered and billed exactly as they are there.

It lists **only** this pack's tools by default. Set `PIPEWORX_META_TOOLS=1` to
also list `ask_pipeworx` and the other gateway-wide tools.

Behind an HTTPS proxy: Node's built-in `fetch` does not read `HTTPS_PROXY` by
default. Newer Node releases honour it when `NODE_USE_ENV_PROXY=1` is set; check
your Node version's documentation.

## Using with ask_pipeworx

Instead of calling tools directly, you can ask questions in plain English —
this works on the pack endpoint above as well as on the full gateway:

```
ask_pipeworx({ question: "your question about Nflverse data" })
```

The gateway picks the right tool and fills the arguments automatically.

## More

- [Docs and guides](https://pipeworx.io/docs)
- [pipeworx.io](https://pipeworx.io)

## License

MIT
