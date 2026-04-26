# Nokia YANG DB Skill

Fast local lookup of YANG paths and platform support for Nokia SR OS and SR Linux,
packaged as a Claude.ai custom skill.

Backed by a pre-built SQLite database with FTS5 full-text index. Covers the latest
release of each product (currently **SR OS 26.3.R2** and **SR Linux 26.3.1**).

## What it does

Answers questions like:

- *"Is SRv6 supported on the IXR-e family?"* → one command, full support matrix
- *"Does the 7250 IXR-e3x support EVPN-VXLAN?"* → yes/no with exit code
- *"Compare BGP-EVPN support on SROS vs SR Linux"* → cross-product check
- *"Give me a feature inventory of the 7220 IXR-D3 in SRL"* → grouped counts

See `SKILL.md` for the full reference.

## Layout

```
.
├── SKILL.md                  # skill description + usage docs (loaded by claude.ai)
├── scripts/
│   └── yang_browser.py       # the entire tool — query + build + update + pack
└── data/
    ├── sros_26.3.R2.db.xz    # ~5 MB compressed → ~71 MB SQLite DB
    └── srlinux_26.3.1.db.xz  # ~1 MB compressed → ~13 MB SQLite DB
```

## Installation in claude.ai

1. Build the skill zip:
   ```bash
   python3 scripts/yang_browser.py --pack-skill
   ```
   This produces `yang-browser.zip` (~6 MB) in the parent directory.

2. Upload it via **Settings → Skills → Create skill** in claude.ai.

## Updating to newer Nokia releases

```bash
python3 scripts/yang_browser.py --release
```

This single command:
- probes `yangbrowser.nokia.com` for newer SR OS / SR Linux releases
- downloads any new `paths.jsonl.gz`
- rebuilds the SQLite DBs and re-packs them as `.db.xz`
- edits the `RELEASES` dict in the script itself
- bundles the final `yang-browser.zip` ready for re-upload to claude.ai

The update is resumable — if interrupted, run `--update --skip-probe` to
continue from where it stopped, then `--pack-skill`.

## Standalone CLI usage

The script is also a usable CLI on its own (no claude.ai required):

```bash
# Feature support matrix
python3 scripts/yang_browser.py --feature srv6 -p "IXR-e" --matrix

# Strict path-level support check (exit 0/1/2 for scripting)
python3 scripts/yang_browser.py --is-supported \
    "/configure/router[router-name=*]/segment-routing/segment-routing-v6" \
    -p "7250 IXR-e3x"

# Cross-product check
python3 scripts/yang_browser.py --cross-product --feature bgp-evpn -p "IXR"

# Capability inventory of a platform
python3 scripts/yang_browser.py --by-platform "7220 IXR-D3" --product srlinux
```

See `python3 scripts/yang_browser.py --help` for all options, or `SKILL.md` for
worked examples.

## Data source

All data is sourced from `https://yangbrowser.nokia.com/releases/{sros|srlinux}/{release}/paths.jsonl.gz`.
This skill is unaffiliated with Nokia.
