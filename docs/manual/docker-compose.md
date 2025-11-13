# Using the Zulip Docker Compose repository

## Understanding Docker

Docker and other container systems are built around shareable container
images. An image is a read-only template with instructions for creating a
container.

Often, an image is based on another image, with a bit of additional
customization. For example, Zulip's `zulip-postgresql` image extends the
standard `postgresql` image (by installing a couple `postgres`
extensions). Meanwhile, the `zulip` image is built on top of a standard `ubuntu`
image, adding all the code for a Zulip application/web server.

Every time you boot a container based on a given image, it's like booting off a
CD-ROM: you get the exact same image (and anything written to the image's
filesystem is lost). To handle persistent state that needs to persist after the
Docker equivalent of a reboot or upgrades (like uploaded files or the Zulip
database), container systems let you configure certain directories inside the
container from the host.

## Volumes

This project's `docker-compose.yml` configuration file uses [Docker
managed volumes][volumes] to store [persistent Zulip
data][persistent-data]. If you use the Docker Compose deployment, you
should make sure that Zulip's volumes are backed up, to ensure that
Zulip's data is backed up.

[volumes]: https://docs.docker.com/engine/storage/volumes/
[persistent-data]: https://zulip.readthedocs.io/en/latest/production/export-and-import.html#backups

## Secrets

XXX TODO

## Dependencies

This project defines a Docker image for a Zulip server, as well as
sample configuration to run that Zulip server with each of the major
[services that Zulip uses][zulip-architecture] in its own container:
`redis`, `postgres`, `rabbitmq`, `memcached`.

## See also

- [How Docker Compose works](https://docs.docker.com/compose/intro/compose-application-model/)
- {doc}`/how-to/compose-getting-started`
- {doc}`/how-to/compose-secrets`
- {doc}`/how-to/compose-settings`
