# Compose: Upgrading

You can upgrade your Zulip installation to any newer version of Zulip with the
following instructions. At a high level, the strategy is to download a new
image, stop the `zulip` container, and then boot it back up with the new
image. When the upgraded `zulip` container boots the first time, it will run the
necessary database migrations.

If you ever find you need to downgrade your Zulip server, you'll need to use
`manage.py migrate` to downgrade the database schema manually.

## Upgrading to a release

1. (Optional) Upgrading does not delete your data, but it's generally good
   practice to [back up your Zulip data][backups] before upgrading to make
   switching back to the old version simple.

   You can back up your database onto the Docker Volume using:

   ```shell
   docker compose exec zulip /sbin/entrypoint.sh app:backup
   ```

   You can back up the contents of the Docker named volume itself:

   ```shell
   docker compose run --rm -v zulip:/data -v $(pwd):/backup zulip \
     tar czf /backup/backup.tar.gz -C /data .
   ```

1. Pull the new image version, e.g. for `11.4` run:

   ```shell
   docker pull zulip/docker-zulip:11.4-0
   ```

   We recommend always upgrading to the latest minor release within a major
   release series.

1. Update this repository to the corresponding `docker-zulip` version. Your
   changes to `compose.override.yaml` should always take preference over those
   in the repository.

1. You can execute the upgrade by running:

   ```shell
   # Restart the zulip container
   docker compose up
   ```

## Upgrading from a Git repository

1. Edit `compose.override.yml`, and specify the Git commit you'd like to build
   the zulip container from, via the `ZULIP_GIT_REF` build argument. For
   example:

   ```yaml
   services:
     zulip:
       image: zulip/docker-zulip:main
       build:
         args:
           # Change these if you want to build zulip from a different repo/branch
           ZULIP_GIT_URL: https://github.com/zulip/zulip.git
           ZULIP_GIT_REF: main
   ```

   You can set `ZULIP_GIT_URL` to any clone of the zulip/zulip git repository,
   and `ZULIP_GIT_REF` to be any ref name in that repository (e.g. `main` or
   `11.4` or `445932cc8613c77ced023125248c8b966b3b7528`).

2. Build the image:

   ```shell
   docker compose build zulip
   ```

3. Update the running Docker Compose instance with that image; this will run
   database migrations, etc:

   ```shell
   docker compose up
   ```

## Upgrading from `docker-zulip` 11.4-0 and earlier

Zulip's Docker deployment received a number of breaking changes and improvements
in 11.5-0.

- Local configurations are expected to be placed in `compose.override.yaml`

- The default configuration moved to being HTTP-only, to reflect that many
  Docker deployments are behind existing proxies. `DISABLE_HTTPS` and
  `SSL_CERTIFICATE_GENERATION` have been removed, and replaced with
  `CERTIFICATES`; see {doc}`compose-ssl`.

- Secrets are no longer environment variables in two places in `compose.yaml`,
  but rather have been moved to use Docker secrets. See {doc}`compose-secrets`.

- `DB_HOST` and `DB_HOST_PORT` have been replaced by
  `SETTING_REMOTE_POSTGRES_HOST` and `SETTING_REMOTE_POSTGRES_PORT`,
  respectively, to align more straightforwardly with standard Zulip settings.

- `REMOTE_POSTGRES_SSLMODE` has been removed, as the usual spelling of
  `SETTING_REMOTE_POSTGRES_SSLMODE` is more straightforward.

- `SPECIAL_SETTING_DETECTION_MODE` has been removed, since its behavior was
  confusing and at odds with its name.

- `NGINX_PROXY_BUFFERING` has been removed, since setting it could only break
  things.

- Contents of the named `zulip` Docker volume have been reorganized;
  certificates are stored in `certs/` subdirectories (`self-signed`, `certbot`,
  and `manual`) and `LINK_SETTINGS_TO_DATA` contents are stored in `etc-zulip/`
  and not `settings/etc-zulip/`. Files should be automatically moved to these
  new locations on first startup.
