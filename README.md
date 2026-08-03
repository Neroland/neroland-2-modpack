# Neroland 2

The official repository and issue tracker for **Neroland 2** (Minecraft 26.2,
NeoForge).

## Reporting issues

Please use the issue templates. Include your pack version, a crash log (via
[mclo.gs](https://mclo.gs)), and clear reproduction steps.

## Modpack sources

- CurseForge: _link once published_
- Modrinth: _link once published_

---

## For maintainers

This pack is managed with [packwiz](https://packwiz.infra.link/). **`mods/*.pw.toml`
is the mod list** — the jars in `mods/` are downloaded output, like a `build/`
folder, and are never committed. Full detail lives in [AUTOMATION.md](AUTOMATION.md).

### Getting the jars

The metafiles describe the pack; packwiz-installer turns them into actual jars so
the instance runs:

```powershell
java -jar packwiz-installer-bootstrap.jar -s client pack.toml
```

If `mods/` looks empty, this is what you need. It's also the fix any time the
CurseForge app reports no mods.

### Adding mods

```powershell
packwiz cf add <slug>       # or: packwiz mr add <slug>
packwiz refresh
```

then re-run the installer to fetch the jar.

If you'd rather drop jars into `mods/` by hand, use the sync script — it imports
anything loose and downloads every jar back afterwards:

```powershell
.\scripts\sync-mods.ps1              # or: ./scripts/sync-mods.sh
.\scripts\sync-mods.ps1 -DryRun
```

> **Don't run `packwiz cf detect` on its own.** It deletes every jar it imports,
> which empties `mods/` and leaves the instance unlaunchable, and it re-adds mods
> that already have a metafile under a different slug — producing duplicate project
> IDs that break the CurseForge export. `sync-mods` wraps it safely: it backs the
> jars up first, de-duplicates afterwards, and reinstalls.

### Releasing

Nothing to tag. Push a pack change to `main` and the release cuts itself:

```powershell
git commit -am "Add Jade and bump NeroTech"
git push origin main
```

`build-release.yml` picks the next version, exports the CurseForge `.zip` and
Modrinth `.mrpack`, publishes both, commits the version bump and tags it last.
Only pack paths trigger it — `mods/`, `config/`, `kubejs/`, `datapacks/`,
`defaultconfigs/`, `pack.toml`, `index.toml` — so docs and workflow edits don't
ship. Use **Actions → Build & Publish Modpack → Run workflow** to force a
`patch`/`minor`/`major` bump or pin an explicit version.

Write a `## [x.y.z]` section in [CHANGELOG.md](CHANGELOG.md) if you want to
control the release notes; otherwise they're generated from the `mods/*.pw.toml`
diff since the last tag.
