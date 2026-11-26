# CI/CD setup

The workflow tests pushes and pull requests targeting `main` and `develop`.
Deployment runs only after successful CI for a push to `main`, including a merge
commit pushed by GitHub. It never runs for a pull-request event.

## Required repository secrets

Configure these under Settings > Secrets and variables > Actions:

| Secret | Value |
| --- | --- |
| `SSH_HOST` | VPS IP address or hostname, without a URL scheme |
| `SSH_USERNAME` | Dedicated non-root deployment account |
| `SSH_PRIVATE_KEY` | Complete private key authorized for that account; this configuration expects a key without a passphrase |
| `SSH_HOST_FINGERPRINT` | Trusted VPS SSH host-key SHA256 fingerprint |

Obtain the fingerprint through the VPS console or another trusted channel, e.g.
`ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub -E sha256`. Copy the `SHA256:...`
field. Do not trust a fingerprint learned only from the first network connection.
SSH uses port 22. No production database credentials or Laravel application key
belong in this workflow; keep them in the server's existing `api/.env`.

## Server contract (required before enabling deployment)

- Linux with Bash, Git, flock, curl, Composer 2, PHP 8.4.1 or newer within 8.4,
  the workflow's PHP extensions, PHP-FPM, and the latest Node.js 20 patch
  (at least 20.19 for the build tooling). CI uses PHP 8.4 by explicit project
  choice: the committed lockfile cannot install on PHP 8.3. Node 20 is retained
  as requested; plan a supported Node LTS upgrade separately.
- Clone this repository to `/var/www/nokify`, check out `main`, and keep the
  checkout clean. Configure the server's own read-only GitHub deploy key and
  trusted GitHub host key so `git fetch` and `git pull` work noninteractively.
  The Actions SSH key grants access to the VPS, not to the repository.
- Give the deployment account ownership of the checkout and a pre-created
  `/var/www/nokify-deploy.lock` file. Give PHP-FPM appropriate access to Laravel's
  `storage` and `bootstrap/cache`; do not use world-writable permissions.
- Bootstrap the first installation manually: install dependencies, prepare
  production `api/.env` with a stable generated `APP_KEY`, `APP_ENV=production`,
  `APP_DEBUG=false`, MySQL credentials, database-backed queues and a persistent
  cache store; migrate and build both packages. Never regenerate the production
  key during deployment. Configure `ui/.env.production.local` on the server if
  needed; public Next.js variables are embedded during the build.
- Configure `php8.4-fpm.service`, `nokify-ui.service` (runs `npm run start --
  --hostname 127.0.0.1 --port 3000` in `ui/`), and `nokify-queue.service` (runs
  `php artisan queue:work` in `api/`). Use production environments and restart
  policies. Queue service shutdown must wait for in-flight jobs to finish; choose
  timeouts matching the longest supported job.
- Permit the deployment user to run only the workflow's exact systemctl
  stop/start/restart/is-active commands via noninteractive sudo. Do not grant
  unrestricted passwordless sudo. Ensure service runtimes use the same PHP/Node
  versions as the build commands and that commands are on the noninteractive PATH.
- Configure the reverse proxy with a local Laravel listener at
  `http://127.0.0.1:8080`, rooted at `api/public`, and proxy Next.js to port 3000.
  Keep both ports private and terminate public HTTPS at the reverse proxy.
  `/up` must reach Laravel and return 2xx without authentication or redirection.
  During deployment, Laravel returns maintenance responses and the stopped
  frontend is unavailable; configure a static 503 page at the proxy if desired.
- Create a GitHub `production` environment restricted to `main`. Add approval
  requirements only if desired. Require `build-and-test` in branch protection.

## Failure handling and validation

This is a safe maintenance-window deployment, not a zero-downtime release switch.
It intentionally does not reopen the application after a failed install or
migration. CI dependencies are installed from lockfiles; the VPS rebuilds the
tested commit using its production configuration rather than receiving a CI build.

Use backups and backward-compatible migrations. Do not automatically roll back
the database: migrations may have partially committed or changed live data.
On failure, inspect the Actions output and server logs, keep Laravel in maintenance
and the UI/workers stopped, and repair or restore a known-good release and database
as appropriate. Rebuild dependencies/assets and caches, restart PHP-FPM and the UI,
verify health, run `php artisan up`, then start workers. The server SHA check
prevents a delayed run from deploying an older commit after a newer main push.

The current frontend has linting but no test script; `npm test --if-present`
automatically enables tests once one exists. Its script must exit after one run
in CI. Backend tests receive MySQL environment variables despite local SQLite
defaults. Laravel's separate Vite assets are built for the welcome view.

Before relying on production deployment, exercise this workflow on a staging VPS
with the same contract: successful deploy, invalid SSH fingerprint, migration/build
failure, health-check failure, and concurrent main pushes. Verify protected branch
behavior and secret availability. Local syntax checks cannot prove a remote deploy.
