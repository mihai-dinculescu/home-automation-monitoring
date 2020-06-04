# Overview
Docker Compose file containing:
- MQTT (eclipse-mosquitto)
- InfluxDB
- telegraf (RPI version - sebd/telegraf-rpi:latest)
- grafana

# Install Docker

```
curl -sSL https://get.docker.com | sh
# <restart>
sudo usermod -aG docker pi
```

```
sudo apt-get install libffi-dev libssl-dev
sudo apt install python3-dev
sudo apt-get install -y python3 python3-pip
```

```
sudo pip3 install docker-compose
```

# Environment variables
The following environment variables are required in `.env`:

```
INFLUXDB_DB=
INFLUXDB_ADMIN_USER=
INFLUXDB_ADMIN_PASSWORD=
DARK_SKY_URL=
DARK_SKY_KEY=
```

# Run Docker Compose
```
docker volume create --name=monitoring-influxdb-storage
docker volume create --name=monitoring-grafana-storage
docker-compose up -d
```

# Grafana Dashboard
- RPI: https://grafana.com/grafana/dashboards/10578
- Docker: https://grafana.com/grafana/dashboards/10585

# Telegraf configuration
These are the changes that have been applied to the default configuration.

```
nano telegraf/telegraf.conf
```

```
[agent]
  omit_hostname = true

[[outputs.influxdb]]
  urls = ["http://influxdb:8086"]

  ## The target database for metrics; will be created as needed.
  database = "telegraf"

  ## we create a seperate database for our measurements, so we don't want the data in the telegraf-database
  namedrop = ["sensors*", "boilers*", "weather*"]

  ## HTTP Basic Auth
  username = "${INFLUXDB_ADMIN_USER}"
  password = "${INFLUXDB_ADMIN_PASSWORD}"

[[inputs.docker]]

[[inputs.file]]
  files = ["/rootfs/sys/class/thermal/thermal_zone0/temp"]
  name_override = "cpu_temperature"
  data_format = "value"
  data_type = "integer"

[[inputs.exec]]
  commands = ["/rootfs/opt/vc/bin/vcgencmd measure_temp"]
  name_override = "gpu_temperature"
  data_format = "grok"
  grok_patterns = ["%{NUMBER:value:float}"]

```

# Generate default configuration files

Influx

```
docker run --rm influxdb influxd config > influxdb.conf
```

Telegraf

```
docker run --rm telegraf telegraf config > telegraf.conf
```
