# How this profile works

A cyberpunk terminal card (`dark.svg` / `light.svg`) replaces the usual badge-and-GIF README. It shows an ASCII portrait next to live GitHub stats, refreshed daily by GitHub Actions.

## Structure

```
.
├── .github/workflows/build.yml   # daily + on-push CI that refreshes the stats
├── cache/                        # per-repo LOC cache (auto-managed, do not edit by hand)
├── dark.svg / light.svg          # the terminal cards embedded in README.md
├── portrait.txt                  # cleaned ASCII portrait (source of truth)
├── portrait_tspan.txt            # portrait.txt converted to SVG <tspan> rows
├── ascii_to_svg.py               # regenerates portrait_tspan.txt from portrait.txt
├── update.py                     # fetches live stats via GitHub GraphQL, writes them into the SVGs
├── requirements.txt
└── README.md
```

## One-time setup: wire up live stats

1. **Create a fine-grained Personal Access Token**: GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens → Generate new token.
   - Repository access: this repo (or all repos, if you want cross-repo LOC counted).
   - Permissions: Contents → Read and write, Metadata → Read-only.
2. **Add repo secrets**: this repo's Settings → Secrets and variables → Actions → New repository secret.
   - `ACCESS_TOKEN` — the token from step 1.
   - `USER_NAME` — `iKajalpatel21`.
3. Push to `main` (or run the workflow manually from the Actions tab). The workflow installs dependencies, runs `update.py`, and commits the refreshed `dark.svg` / `light.svg` back to the repo.
4. After the first successful run, the placeholder `0`s in the terminal's "GitHub Stats" section will show your real repo/star/commit/LOC counts, and it'll re-run every day at 04:00 UTC.

## Updating the portrait

1. Drop a new front-facing, high-contrast photo somewhere on disk.
2. Regenerate the ASCII art (there's no bundled converter script — either use a tool like `ascii-image-converter --width 90 photo.jpg > portrait.txt`, or ask for a re-conversion), keeping it under ~60 lines and centered.
3. Run `python ascii_to_svg.py` — this reads `portrait.txt` and rewrites `portrait_tspan.txt`.
4. Paste the contents of `portrait_tspan.txt` into the `<text class="ascii">...</text>` block near the top of `dark.svg` and `light.svg`, replacing the old `<tspan>` rows.

## Editing the info panels

Everything in the right-hand panel (Subject, Role, ToolChain, Neural.* skills, Contact) is static text directly inside `dark.svg` / `light.svg` — edit both files' `<tspan class="value">...</tspan>` entries to update it. The `id="..._data"` spans (Repos, Stars, Commits, Followers, Lines of Code) are overwritten automatically by `update.py` — don't hand-edit those, your changes will be overwritten on the next run.

## Color palette

Defined once per file in the `<style>` block: `.key` (cyan/teal labels), `.value` (body text), `.cc` (dim leader dots), `.addColor` / `.delColor` (LOC diff), plus the section-header pink (`fill="#FF5F87"` in dark, a darker rose in light) and the ASCII fill color. Change those hex values to retheme.
