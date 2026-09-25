# App Blueprint draft — Cal.diy (community build)

**Status:** `Draft` — seed content for the Blueprint repository `ever-works/cal-template` (topic
`ever-works-app-blueprint`). **Owner:** [APW-13](../../spec.md). **Shape:** [CONTRACTS.md §1](../../../CONTRACTS.md).
**Unverified** in the sense of [APW-13 spec §4.5](../../spec.md): nothing here has run on a cluster.

This Blueprint tells Ever Works how to build and run a fork of `calcom/cal.diy` as an App Work. It contains
**no upstream source**: only [`.works/works.yml`](./.works/works.yml) and this README.

## Licence and edition

**Cal.diy is MIT-licensed.** That is the upstream project's licence, read from `calcom/cal.diy`'s repository
metadata and its root `LICENSE`; the App spec records it as MIT, class `green`. It is re-evaluated on every
upstream sync by the platform's licence gate.

**This Blueprint targets the community edition — `calcom/cal.diy` — and nothing else.** Cal.diy is the Open
Scheduling community build; it is not the commercial Cal.com product, and it is not affiliated with or
endorsed by Cal.com, Inc.:

> Cal.diy® is a trademark of Cal.com, Inc. This community build is not affiliated with or endorsed by
> Cal.com, Inc.

Two consequences worth stating plainly:

- **Version tags `v6.2.0` and older are a different licence and a different code base.** A pin must never be
  moved back to them.
- **Upstream recommends personal, non-production use**, so this Blueprint is **not** offered for managed
  hosting; it runs on the user's own cluster.

## What this repository is

The **Ever Works App Blueprint for Cal.diy (community build)** — the App spec the platform applies when it
builds and runs a fork of `calcom/cal.diy` as an App Work.

| Path | What it is |
| --- | --- |
| [`app-spec.yml`](./app-spec.yml) | the App spec itself — the file the platform applies |
| [`.works/works.yml`](./.works/works.yml) | the same App spec, byte for byte, at the path the platform's Blueprint resolver reads (FR-43 / CONTRACTS §8); change it together with `app-spec.yml` |
| [`.works/template.yml`](./.works/template.yml) | this repository's shape and app source, read by the catalog/resolver |

It is **metadata-only**: the application's source is not in this repository, and the file layout below is
the specification's, not the upstream project's. The listing in `ever-works/templates` records this
Blueprint with the shape a *published* Cal.diy Blueprint takes (`code-bearing`, a public fork of the
upstream); this repository is the metadata-only seed, so it declares the shape it actually is.

**Not released, not verified.** Nothing here has run on a cluster, and `blueprint.sha` is a placeholder
until the release workflow stamps it. Links in this document that point outside the repository were written
in the platform specification's repository and do not resolve from here.

## What the Blueprint decides

| Concern                | Decision                                                                                                                      |
| ---------------------- | ----------------------------------------------------------------------------------------------------------------------------- |
| Name and marks         | Displayed as **Cal.diy (community build)** with the trademark notice; logo and licence files are read-only to agents.         |
| Licence                | MIT (green). Re-evaluated on every Upstream sync by the licence gate.                                                         |
| Build                  | The upstream Dockerfile, target `runner`, 6 GB Node heap, 4 CPU / 12 GiB / 60 min, a throwaway Postgres 16 during the build.  |
| Secrets at build time  | None. The Dockerfile's build-only placeholders satisfy the framework; real values exist only at run time.                     |
| Runtime                | One `web` component, writable root filesystem (the image rewrites its public URL at boot), 10-minute startup budget.          |
| Database migrations    | A `pre-deploy` job, so a failed migration stops the rollout.                                                                  |
| First administrator    | Created by a `first-deploy` job over the in-cluster URL before the ingress is published; the email and password are prompted. |
| Scheduled calls        | Seven CronJobs whose routes exist at the pin; the rest are listed and disabled with the reason.                               |
| Cron credentials       | Both generated per App Work; nothing uses an example value.                                                                   |
| Telemetry              | Disabled.                                                                                                                     |
| Email                  | An SMTP App dependency is required.                                                                                           |
| Evolve loop            | Agents load the upstream `AGENTS.md`; PRs are capped at 500 changed lines; `yarn type-check:ci --force` is a required check.  |
| Upstream contributions | Off by default; always human-approved.                                                                                        |

## Facts and where they were read

Every fact below was read on **2026-09-17** in the public upstream repository at commit
`6bc45298226f96ff79e0c070c8b2ce39727e8477` (committed 2026-09-14; the default-branch head on the day of reading).
"Read" is not "run" — see the unverified list.

| Fact                                                                                                     | Source (at the pin)                                                                                                                    |
| -------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| Licence MIT                                                                                              | repository metadata, `LICENSE`                                                                                                         |
| Build stages, `ARG MAX_OLD_SPACE_SIZE=6144`, build-only secret placeholders, `EXPOSE 3000`, CMD          | `Dockerfile`                                                                                                                           |
| Boot sequence: public-URL placeholder rewrite → wait for `DATABASE_HOST` → migrate → seed → start        | `scripts/start.sh`                                                                                                                     |
| `/api/version` returns the package version without touching the database                                 | `apps/web/app/api/version/route.ts`                                                                                                    |
| Framework refuses to start without `NEXTAUTH_SECRET` / `CALENDSO_ENCRYPTION_KEY`; derives `NEXTAUTH_URL` | `apps/web/next.config.ts`                                                                                                              |
| `CALENDSO_ENCRYPTION_KEY` must be 32 characters; `CRON_API_KEY`; telemetry opt-out variable              | `.env.example`                                                                                                                         |
| First-run setup route body fields and password rule; "No setup needed." once a user exists               | `apps/web/app/api/auth/setup/route.ts`                                                                                                 |
| `/auth/login` redirects to first-run setup while no user exists                                          | `apps/web/server/lib/auth/login/getServerSideProps.tsx`                                                                                |
| Cron handlers, methods and credential checks                                                             | `apps/web/app/api/cron/*/route.ts`, `apps/web/app/api/tasks/{cron,cleanup}/route.ts`, `packages/features/tasker/api/{cron,cleanup}.ts` |
| Cron schedules                                                                                           | `apps/web/vercel.json`, `.github/workflows/cron-*.yml`                                                                                 |
| Agent guidance: type check command, PR size, draft PRs, ask before schema changes                        | `AGENTS.md`                                                                                                                            |
| Branding assets                                                                                          | `apps/web/public/`, `packages/ui/components/logo/`                                                                                     |

Public references: <https://github.com/calcom/cal.diy> · <https://github.com/calcom/cal.diy/blob/main/AGENTS.md>.

## Unverified — resolve on the first verification run

1. Whether the build still needs a reachable database (the Blueprint provides one either way).
2. The host name and authentication the build plugin uses for build services (APW-05).
3. The output names of the `postgres` and `smtp` dependencies (APW-07).
4. Memory request and limit for `web` — measure, then replace the placeholders.
5. Whether `selected-calendars` and `sync-app-meta` do useful work in this edition (both disabled).
6. How to close public signup: the variable is build-time and the Dockerfile does not accept it as an argument.
7. The `biome` command form. (Resolved: checks install nothing implicitly — APW-08 plan §2.3 — so each check command
   starts with `yarn install --immutable`.)

## Refreshing the pin

The pin lives in the Apps catalog entry's ref range (APW-03) and in the comment at the top of `works.yml`.

1. The weekly **upstream-sync canary** (APW-13 spec §4.5) builds and smoke-tests the upstream head. Wait for it to be
   green three runs in a row at the candidate commit.
2. Re-read every row of the facts table at the candidate commit. Any change to the Dockerfile, the start script,
   the setup route, a cron route or `.env.example` means editing `works.yml` in the same pull request.
3. Re-check the licence. **Tag `v6.2.0` and older are a different licence and a different code base — never pin
   to them.**
4. Open one pull request that bumps the comment, the catalog ref range and `blueprint.version`. A person merges it.
   The Blueprint is `verified` again only after the verification lane's pass streak at the new pin.
