# Grafana Setup

> This is a WIP and temporary setup...

Currently, using DigitalOcean Marketplace images for [Grafana](https://marketplace.digitalocean.com/apps/grafana).

## Modifications to Grafana instance

- Upgrade Grafana to 8.3
  https://grafana.com/grafana/download?utm_source=grafana_footer
  ```shell
  sudo apt-get install -y adduser libfontconfig1
  wget https://dl.grafana.com/enterprise/release/grafana-enterprise_8.4.2_amd64.deb
  sudo dpkg -i grafana-enterprise_8.4.2_amd64.deb
  ```

- Update config located at `/etc/grafana/grafana.ini`
  ```ini
  [server]
  domain = grafana.openlibra.space
  ```

- Restart
  `systemctl restart grafana-server.service`

## First login

- Change admin password
- Add data source
  - url: $PROMETHEUS_URL
  - auth: basic
  - set credentials
- Create viewer user
- Import dashboards

---

## Todo

- [ ] Refactor dashboards to utilise new tags that distinquish node types
- [ ] Run in K8s
- [ ] Setup auto provisioning: dashboards, alerts, settings
