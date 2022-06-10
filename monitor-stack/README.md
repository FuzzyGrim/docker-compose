# Monitoring Stack
```bash
mkdir -p monitoring/{grafana,loki,promtail,prometheus}
vim ~/docker/monitor/docker-compose.yml
```

## ./grafana/docker-compose.yml
```yml
version: "3"
services:
  grafana:
    image: grafana/grafana:latest
    user: "1000"
    volumes:
    - ./grafana:/var/lib/grafana
    ports:
      - "3000:3000"
    restart: unless-stopped
    networks:
      - default

  loki:
    image: grafana/loki:2.4.0
    volumes:
      - ./loki:/etc/loki
    ports:
      - "3100:3100"
    restart: unless-stopped
    command: -config.file=/etc/loki/loki-config.yml
    networks:
      - default

  promtail:
    image: grafana/promtail:2.4.0
    volumes:
      - /var/log:/var/log
      - ./promtail:/etc/promtail
    # ports:
    #   - "1514:1514" # this is only needed if you are going to send syslogs
    restart: unless-stopped
    command: -config.file=/etc/promtail/promtail-config.yml
    networks:
      - default

  prometheus:
    container_name: prometheus 
    ports:
      - '9080:9090'
    volumes:
      - './prometheus:/etc/prometheus'
    image: prom/prometheus
    restart: unless-stopped
    networks:
      - default
```

## ./loki/loki-config.yml
```yml
auth_enabled: false

server:
  http_listen_port: 3100
  grpc_listen_port: 9096

common:
  path_prefix: /tmp/loki
  storage:
    filesystem:
      chunks_directory: /tmp/loki/chunks
      rules_directory: /tmp/loki/rules
  replication_factor: 1
  ring:
    instance_addr: 127.0.0.1
    kvstore:
      store: inmemory

schema_config:
  configs:
    - from: 2020-10-24
      store: boltdb-shipper
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 24h

ruler:
  alertmanager_url: http://localhost:9093
```

## Loki Docker Driver

Install loki plugin: `docker plugin install grafana/loki-docker-driver:latest --alias loki --grant-all-permissions`

Check it is installed with `docker plugin ls`

`sudo vim /etc/docker/daemon.json`
```bash
{
    "log-driver": "loki",
    "log-opts": {
        "loki-url": "http://localhost:3100/loki/api/v1/push",
        "loki-batch-size": "400"
    }
}
```

Restart docker daemon: ```sudo systemctl restart docker```, you will also need to recreate the services you want logs or wait for next reboot.

## Promtail
CHANGE PROMTAIL CONFIG TO:

`vim ./promtail/promtail-config.yml`
```yml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:

# local machine logs
- job_name: local
  static_configs:
  - targets:
      - localhost
    labels:
      job: varlogs
      __path__: /var/log/*log
  
## docker logs

- job_name: docker 
  pipeline_stages:
    - docker: {}
  static_configs:
    - labels:
        job: docker
        __path__: /var/lib/docker/containers/*/*-json.log

# syslog target

#- job_name: syslog
#  syslog:
#    listen_address: 0.0.0.0:1514 # make sure you also expose this port on the container
#    idle_timeout: 60s
#    label_structured_data: yes
#    labels:
#      job: "syslog"
#  relabel_configs:
#    - source_labels: ['__syslog_message_hostname']
#      target_label: 'host'
```

Restart docker daemon: `sudo systemctl restart docker`.

You will also need to recreate the containers.

## LogQL sample queries
Query all logs from the `varlogs` stream
```
{job="varlogs"}
```
Query all logs from the `varlogs` stream and filter on `docker`
```
{job="varlogs"}  |= "docker"
```
Query all logs from the `container_name` label of `uptime-kuma` and filter on `host` of `juno`
```
{container_name="uptime-kuma", host="juno"}
```

## Netdata
### Docker Compose
`bash <(curl -Ss https://my-netdata.io/kickstart.sh)`

Check [http://192.168.1.126:19999](http://192.168.1.126:19999)

`sudo apt install lm-sensors`

Restart netdata with:
1. `sudo systemctl stop netdata`
2. `sudo systemctl start netdata`
### Delete netdata
Script located at: `/usr/libexec/netdata/netdata-uninstaleler.sh`

## Prometheus
`vim ./prometheus/prometheus.yml`
```yml
# my global config
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
    - targets: ['localhost:9090']
    
#############################################################################

  - job_name: 'netdata-scrape' # change_me

    metrics_path: '/api/v1/allmetrics'
    params:
      # format: prometheus | prometheus_all_hosts
      # You can use `prometheus_all_hosts` if you want Prometheus to set the `instance` to your hostname instead of IP 
      format: [prometheus]
      #
      # source: as-collected | raw | average | sum | volume
      # default is: average
      #source: [as-collected]
      #
      # server name for this prometheus - the default is the client IP
      # for Netdata to uniquely identify it
      #server: ['prometheus1']
    honor_labels: true
    static_configs:
      - targets: ['192.168.1.126:19999'] # change_me

#############################################################################
```

If you decide to monitor more than one Netdata instance just copy and paste from each pound line then edit the job name and target IP.

Once saved the config, restart the prometheus service.

## Start containers

`docker-compose up -d --force-recreate # be sure you've created promtail-config.yml and loki-config.yml before running this`


## Grafana config
Now:
1. Wait until [http://192.168.1.122:3100/ready](http://192.168.1.122:3100/ready) shows ready, 
2. Go to [http://192.168.1.122:3000](http://192.168.1.122:3000) and use user:admin password:admin
3. Go to configuration > data sources> on left panel.
4. Click 'Add data source'
5. Scroll down and search/select Loki
6. Change Http url to: [http://loki:3100](http://loki:3100)
7. Scroll down and click 'Save & test'
8. Go to explore in left panel, and make sure that data source is set to Loki
9. To add queries:
	- In 'Log browser' write: {}
    - Select 'job'
    - Select 'varlogs'
    - Type ctrl + enter, and search for logs
10. Add Prometheus as data source and put your HTTP URL, e.g, http://192.168.1.126:9080

### System uptime
Search `netdata_system_uptime_seconds_average` in Prometheus.
Get the query, e.g: 
```
netdata_system_uptime_seconds_average{chart="system.uptime", dimension="uptime", family="uptime", instance="192.168.1.126:19999", job="storage-netdata"}
```

Go to Grafana, add panel with "Metrics browser": query copied before. (Make that the data source selected is Prometheus)

- Scroll down until Standard options>Unit>Time>Seconds.
- On the top navbar, change from 'Time series' to 'Stat'.
- Stat styles>Horizontal.
- Change 'Threshold' to `43200` for 12H.
- Change 'Title' to 'Uptime'.

### CPU Temperature (requires sensors to be installed on the server using lm-sensors)
Search `netdata_sensors_temperature_Celsius_average`
Select the one with Package id 0, e.g: 
```
netdata_sensors_temperature_Celsius_average{chart="sensors.coretemp_isa_0000_temperature", dimension="Package id 0", family="temperature", instance="192.168.1.126:19999", job="storage-netdata"}
```
- Change Title: CPU Package Temp
- 'Time series' to 'Stat'
- Standard options>Unit>Temperature>Celcius
- Change 'Threshold'
- Change 'Base' color

### CPU USAGE
Search `netdata_cpu_cpu_percentage_average`
Select the one with user:
```
netdata_cpu_cpu_percentage_average{chart="cpu.cpu0", dimension="user", family="utilization", instance="192.168.1.126:19999", job="storage-netdata"}
```
- Standard Options>Unit>Misc>Percentage
- Standard Options>Max>100

Create another one but instead of Stat, use Time Series and name it CPU Usage Flow
- Legend (Under Metrics browser), write: CPU Chart
- Toggle 'Transparent Background'
- Remove Unit and Max
- Standard Options > Color scheme> Blue-Yellow-Red
- Thresholds> Base> BLue
- Thresholds > Show thresholds > As filled regions and lines

### Memory Used
Search `netdata_system_ram_MiB_average`
Select used, e.g:
```
netdata_system_ram_MiB_average{chart="system.ram", dimension="used", family="ram", instance="192.168.1.126:19999", job="storage-netdata"}
```
- Standard Options> Unit> Data> Megabytes

Also add another panel with:
```
netdata_system_ram_MiB_average{chart="system.ram", dimension="free", family="ram", instance="192.168.1.126:19999", job="storage-netdata"}
```
### Hard drive space
Search `netdata_disk_space_GiB_average`
We will add:
```
netdata_disk_space_GiB_average{chart="disk_space._", dimension="avail", family="/", instance="192.168.1.126:19999", job="storage-netdata"}
```
and (/data)
```
netdata_disk_space_GiB_average{chart="disk_space._data", dimension="avail", family="/data", instance="192.168.1.126:19999", job="storage-netdata"}
```
and (/backup)
```
netdata_disk_space_GiB_average{chart="disk_space._backup", dimension="avail", family="/backup", instance="192.168.1.126:19999", job="storage-netdata"}
```

All:
- Time Series to Stat:
- Unit to Gigabytes
- Set Thresholds