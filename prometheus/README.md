# Prometheus Setup

> This is a WIP and temporary setup...

Currently, using DigitalOcean Marketplace images for [Prometheus](https://marketplace.digitalocean.com/apps/prometheus).

## Modifications to Prometheus instance

- Route traffic using floating ip
  https://docs.digitalocean.com/products/networking/floating-ips/how-to/outbound-traffic/

- Stop prometheus
  `systemctl stop prometheus.service`

- Mount data volume `/mnt/data/` and `mkdir -p /mnt/data/prometheus`

- Move prom configs
  `mv /etc/prometheus/* /mnt/data/prometheus/`

- Move prom data
  `mv /var/lib/prometheus /mnt/data/prometheus/db`

- Generate `/mnt/data/prometheus/web.yml` with admin user and pass to enable basic auth
  https://prometheus.io/docs/guides/basic-auth/
  https://bcrypt-generator.com/

  ```
  basic_auth_users:
    admin: $encrypted-$pass
  ```

- Update `/etc/systemd/system/prometheus.service`
  - replace `/etc/prometheus` with `/mnt/data/prometheus`
  - update `--storage.tsdb.path=/mnt/data/prometheus/db`
  - update `ReadWritePaths=/mnt/data/prometheus`
  - Add `--web.config.file=/mnt/data/prometheus/web.yml`

## Upgrading Prometheus

https://rakeshjain-devops.medium.com/upgrading-prometheus-from-2-x-to-the-latest-2-17-2-version-15173e002a86

## Todo

- [ ] Share blueprint for prometheus config
- [ ] Refactor scraping jobs to distinguish node types
- [ ] Run in K8s

## Manual Node Exporter setup

- Download [Node Exporter](https://prometheus.io/docs/guides/node-exporter/)

  ```
  wget -q https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz
  tar -xvf node_exporter-1.3.1.linux-amd64.tar.gz
  cp node_exporter-1.3.1.linux-amd64/node_exporter /usr/local/bin
  ```

- Add user and group to run the node exporter
  ```
  adduser --system --shell /bin/false node_exporter
  ```

- Install node exporter as system service
  `/etc/systemd/system/node_exporter.service`
  ```ini
  [Unit]
  Description=Prometheus Node Exporter
  After=network.target

  [Service]
  Type=simple
  User=node_exporter
  Group=node_exporter
  ExecStart=/usr/local/bin/node_exporter \
      --web.listen-address=0.0.0.0:9100

  SyslogIdentifier=node_exporter
  Restart=always

  PrivateTmp=yes
  ProtectHome=yes
  NoNewPrivileges=yes
  ProtectSystem=strict
  ProtectControlGroups=true
  ProtectKernelModules=true
  ProtectKernelTunables=yes

  [Install]
  WantedBy=multi-user.target
  ```

- Reload system services and start node exporter
  ```
  systemctl daemon-reload
  systemctl start node_exporter.service 
  systemctl enable node_exporter.service 
  systemctl status node_exporter.service 
  ```

## Resources

- Prometheus Installation step-by-step on Ubuntu 20.04
  > https://gist.github.com/daniyalkz/0df0954a7b8bd52f3fb343da7dbec6a8
