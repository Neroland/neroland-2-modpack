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

### The release loop

Manage mods however you like in the **CurseForge app** — add, remove or disable
them — then run the sync script once. It reconciles the pack with whatever is
actually in `mods/`, sets the build number, and downloads every jar back so the
instance stays playable:

```powershell
.\scripts\sync-mods.ps1 -Prune -Version 0.0.1-alpha.5
git add -A
git commit -m "Release 0.0.1-alpha.5"
git push origin main
```
```bash
./scripts/sync-mods.sh --prune --version 0.0.1-alpha.5
```

That's the whole loop. The push publishes to CurseForge and Modrinth on the
channel implied by the version (`-alpha.N` → alpha, `-beta.N`/`-rc.N` → beta,
otherwise release), then tags the commit.

- `-Prune` drops mods you deleted in the CurseForge app. Without it they're only
  reported, so a first run can't surprise you.
- `-Version` sets the build number. Omit it and the workflow picks the next one
  itself.
- `-DryRun` shows what would happen and changes nothing.

Removals are detected from `packwiz.json`, the installer's record of what it
wrote, and only when `mods/` still holds jars — so an uninstalled pack can never
be mistaken for "everything was deleted".

Prefer to add mods without the app? `packwiz cf add <slug>` (or `mr add`), then
`packwiz refresh` and re-run the installer.

> **Don't run `packwiz cf detect` on its own.** It deletes every jar it imports,
> which empties `mods/` and leaves the instance unlaunchable, and it re-adds mods
> that already have a metafile under a different slug — producing duplicate project
> IDs that break the CurseForge export. `sync-mods` wraps it safely: it backs the
> jars up first, de-duplicates afterwards, prunes, and reinstalls.

### Releasing without touching mods

Any pack change pushed to `main` releases itself:

```powershell
git commit -am "Retune NeroTech config"
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
