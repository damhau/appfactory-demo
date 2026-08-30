# appfactory-demo

The **definitions Git target** for the [Edge App Factory](https://github.com/damhau/appfactory) demo: a public, demo-only GitOps repo. ArgoCD deploys from it, the studio opens pull requests against it, and its merge history is the deployment log. Nothing here is code — apps are declarative definitions interpreted by the platform's one generic renderer.

Demo instance: single site **NAI** on `apps.dhconsulting.ch`. The full bring-up order lives in the platform repo's `docs/runbooks/demo-bringup.md`.

## Layout

```
apps/                The application definitions — where the studio writes on
                     submit. Starts empty: the first merged submission creates
                     the first directory (apps/<name>/app.yaml + a two-line
                     wrapper kustomization).
deploy/nai/          The site's membership: which definitions run at NAI.
                     One resource line per app; merging a line IS deploying the app.
fleet/sites/NAI.yaml The site registry entry the studio reads.
                     Duplicated in the platform repo for bootstrap/preflight — keep in sync.
```

(`apps/` is the platform's v1 name for this directory — the monorepo's
`examples/` is the v0 form. The studio is pointed here via `studio.appsPath`.)

The repo starts **empty** — empty membership (`resources: []`), no definitions.
The PoC's arc is authoring a **new** app in the studio and deploying it to NAI
through a pull request: empty portal at first login, then submit → merge →
`apps-nai` syncs → the app answers. If a start-from-template beat is ever
wanted, that material belongs in `blueprints/` (the studio's gallery) — never
in `apps/`, whose contents the studio presents as existing applications.

## What points at this repo

- The **definitions ApplicationSet** (`delivery/argocd/definitions-appset.yaml` in the platform repo): one `apps-nai` Application per `deploy/*` folder. The repo must be in the `appfactory` AppProject's `sourceRepos`.
- The **studio** (`delivery/charts/studio/values.yaml`): `gitRemoteUrl` for clone/push, `gitRepo: damhau/appfactory-demo` for the PR API, `appsPath: apps`, and a fine-grained PAT scoped to this repo only (Contents + Pull requests, read/write).
- ArgoCD needs no repository credential while this repo is public.

## Rules

- **Protect `main`** requiring the `definitions` check — "deployed = merged" only holds if merge implies green.
- Membership `kustomization.yaml` files list references only (sorted, `../../apps/<app>` — directory bases); the studio rewrites them byte-stably — no manual reformatting.
- The tree builds with **stock kustomize** — no flags, no `argocd-cm` settings. Each app directory's wrapper kustomization is plumbing (`resources: [app.yaml]`, nothing else); the platform's CI guards refuse anything richer.
- Demo data only: every definition, name, and user in this repo is fictional.
