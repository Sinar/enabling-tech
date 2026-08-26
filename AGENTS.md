# Enabling-Tech — Agent Quick Reference

Plone 6.1.5 CMS project. Buildout-driven, Python 3.12, GPLv2.

This branch (`plone-6.1`) targets Plone 6.1; `main` stays on Plone 6.0.15.
All `src/*` packages are checked out from their `plone-6.1` git branches
(shared with the sibling `~/src/sinarproject.org` workspace — see the
`ploneupgrade` skill under `.opencode/skills/`).

## Boot

```
bin/buildout          # bootstrap + install
bin/instance fg       # dev server, port 8080, login admin:admin
bin/zopepy            # interactive Plone shell
bin/update_locale     # regenerate i18n catalogs
```

## Initialize project

1. Ensure `uv` is installed
2. Check for `.venv` virtual environment for the supported Python version (3.12)
   - If `.venv` exists → activate it
   - If `.venv` doesn't exist → create it with `uv venv --python 3.12` then activate
3. Install recommended build tool versions from Plone 6.1.5:
   - `zc.buildout = 4.2.0`
   - `wheel = 0.47.0`
   - `setuptools = 81.0.0`
   - WARNING: `uv pip install` targets `$VIRTUAL_ENV` when it is set — in
     this multi-workspace machine it can point at another project's venv.
     Check `echo $VIRTUAL_ENV` before installing; pin the env explicitly
     with `VIRTUAL_ENV=$PWD/.venv uv pip install ...`
4. Run `bin/buildout` to bootstrap the project

## Deployment mode

```
bin/buildout -c deployment.cfg
# starts ZEO server (8101) + instance as ZEO client (8090)
```

## Source layout

All packages are namespace packages under `src/`, managed by `mr.developer` from git:

| Package | Content / Purpose |
|---|---|
| `et.customizations` | Main customization glue (browserlayer, registry, views, z3c.jbot overrides) |
| `sinar.activity` | Activity / ProjectActivity content types |
| `sinar.article` | Article content type |
| `sinar.coalition` | Coalition content type |
| `sinar.indicators` | M&E indicators |
| `sinar.miscbehavior` | Extra behaviors for content types |
| `sinar.opportunities` | Opportunity content type |
| `sinar.organization` | Organization content type |
| `sinar.project` | Folderish Project content type |
| `sinar.resource` | Resource content type |
| `collective.vocabularies.iso` | ISO country/currency vocabularies |

## Testing changes

After making changes, verify nothing breaks:

1. Run per-package tests: `cd src/<pkg> && tox`
2. Start the dev server and watch for errors: `bin/instance fg`

The instance will log any import errors, missing dependencies, or configuration issues on startup. Check the console output for failures before proceeding.

## Gotchas

- **`mr.developer` auto-checks out every package** (`auto-checkout = *`, `always-checkout = true`). Never edit checked-out packages directly — make changes in the git working copy and re-run buildout.
- **`deployment.cfg` adds `sinar.advisory`** (not present in `src/`, only in the buildout sources section). It will fail if the repo is missing.
- **`eea.facetednavigation`** appears in `.installed.cfg` but not in `buildout.cfg` — it was likely added manually at some point.
- **All source packages are on their `plone-6.1` branches** (per
  `[sources]` in `buildout.cfg` / `deployment.cfg`). `main` in each
  package repo is the Plone 6.0.15 line — commit 6.1 work to
  `plone-6.1`, never to `main`.
- **Venv `nspkg.pth` pitfall:** after any venv rebuild,
  `.venv/lib/python3.12/site-packages/zc.buildout-*-nspkg.pth` may
  reappear and silently break `bin/instance` / `bin/zopepy`
  (`ModuleNotFoundError: No module named 'zc.relation'`). Remove the
  file; see the `ploneupgrade` skill.
- **Port 8080 collision:** another workspace on this machine
  (`kaeru.my`) also wants 8080. Check `ss -ltnp` before starting; use a
  spare port via `fast-listen` in `parts/instance/etc/wsgi.ini` if needed.
- **ZODB holds two sites** (`/ET/` live, `/enabling-tech/` dev); `/`
  serves the plone.distribution multi-site overview. Site-root changes
  apply per site.
- Lint/format: `isort`, `flake8`, `black`. Run per-package: `cd src/<pkg> && tox -e lint` or `tox -e black-check`.

## Commit messages

- Author: check current git user via `git config user.name`
- Follow [Conventional Commits](https://github.com/conventional-commits/conventionalcommits.org/blob/master/content/v1.0.0/index.md) with multi-paragraph bodies (summary, blank line, explanation, blank line, attribution)
- If AI-assisted, add the attribution line at the end using the format `Assisted-by: <AGENT_NAME>:<MODEL_VERSION>` (see [Linux Kernel AI Attribution](https://github.com/torvalds/linux/blob/master/Documentation/process/coding-assistants.rst))

## Content model highlights

- `FrontPage` view (in `et.customizations.views`) queries Resource, Activity, and Opportunity types.
- `Project` is folderish — can contain child content.
- `ProjectActivity` ties an Activity to a Project via relations.
