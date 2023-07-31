# NGI Monitor Stack

Docker Compose configuration to deploy a Grafana, Prometheus and various exporters

## Requirements

- Docker
- Docker Compose
- A discourse installation deployed via docker

## Setup

- To get metrics from Discourse and its plugin [Discourse Prometheus](https://github.com/discourse/discourse-prometheus) the container on which discourse is running and Prometheus need to share network. The `docker-compose.yaml` already expect it to be on the `discourse` network. In case it's not, you can do the following to rebuild discourse container to be on that network instead of the default `bridge` while also assigning a fixed network alias to the container, useful to have a determined reference for it:
  - Run `docker network create -d bridge discourse`
  - Run `sudo /var/discourse/launcher rebuild app --docker-args '--network discourse --hostname=discourse_app'`

Be aware that from this moment and on, any rebuild (to install plugins, updare discourse, etc) need to have those parameters added to prevent the discourse installation to revert to default values for network and hostname.

To simplify the rebuild action a simple bash script can be created:

```
nano rebuild_discourse.sh
```

```
# /bin/bash
sudo /var/discourse/launcher rebuild app --docker-args '--network discourse --hostname=discourse_app'
```

`CTRL+X` -> `Y` -> `ENTER`

Then 

```
chmod +x rebuild_discourse.sh
```

Simply run `./rebuild_discourse.sh` from now on to run the default rebuild but specifying the networking.

## Grafana Setup

- If you are **not** deploying for the main domain, edit `grafana\grafana.ini` the attributes `root_url` and `domain` to match the actual domain you are working with.
- Add the SMTP credentials in `grafana\grafana.ini` to be able to receive alert emails
- Grafana is expected to run under https protocol. You'll need a `.crt` and `.key`. Replace the ones in `grafana/ssl-certs`.
- Edit the `docker-compose.yaml` file then replace the `ADMIN_USER_PLACEHOLDER` and `ADMIN_PASS_PLACEHOLDER` to generate the correct credentials to access the dashboard
- Edit the `docker-compose.yaml` file then replace the `A_VALID_PORT` with a valid https port for grafana that is not used already by the host
- Edit `grafana/provisioning/alerting/alerting.yaml` with a valid email to receive the notification instead of `ALERT_EMAIL_NOTIFICATION_PLACEHOLDER`
- Edit `grafana/provisioning/alerting/alerting.yaml` with a valid discord webhook to receive the notification instead of `DISCORD_WEBHOOK_PLACEHOLDER`


## Deploy

To deploy the stack simply move into the directory root of this project and run `docker compose up -d`

## Logs

You can check the logs with `docker compose logs --follow`

## Known Issues

Currently Discourse Prometheus doesn't seem to allow connection to the expected endpoint even from internal IP while it should. A topic is open on the subject.
