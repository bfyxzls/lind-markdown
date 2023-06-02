# monitor

## server

Deployed on server `bot-monitor`: `10.0.1.121`.

Forward ssh ports in local: 

```bash
ssh bot-monitor -N -L 3000:localhost:3000 -L 9090:localhost:9090 -L 9121:localhost:9121
```

Then access services:
- grafana web UI: <http://localhost:3000>, default user/passwd: `admin/admin`
- prometheus web UI: <http://localhost:9090>
- redis_exporter metrics: <http://localhost:9121/metrics>



## prometheus

### deploy

Deploy [prometheus](https://prometheus.io/) along with [redis_exporter](https://github.com/oliver006/redis_exporter).

Prepare folder:

```bash
sudo -i

mkdir -p /data/prometheus
cd /data/prometheus
mkdir prometheus
chmod -R 777 prometheus
```

Copy files in `monitor/prometheus` to server, as structure:

```
/data/prometheus
├── prometheus/
├── docker-compose.yml
└── prometheus.yml
```


### run

To launch prometheus (along with redis_exporter), in folder `/data/prometheus`, run: 

```bash
docker-compose up -d
```

Check logs:

```bash
docker-compose logs prometheus
docker-compose logs redis_exporter
```

Check redis_exporter metrics:

```bash
curl -s "localhost:9121/metrics"
curl -s "localhost:9121/scrape?target=redis://10.0.1.55:6379"
```

Check prometheus web UI: <http://localhost:9090>



## grafana

### deploy

Just prepare a folder for grafana:

```bash
sudo -i

mkdir -p /data/grafana/grafana
```


### run

Run:

```bash
ID=$(id -u)

docker run \
--detach \
--user $ID \
--name grafana \
-p 3000:3000 \
-v /data/grafana/grafana:/var/lib/grafana \
grafana/grafana:8.0.6
```

Check grafana web UI: <http://localhost:3000>, default user/passwd: `admin/admin`


### data source

Elasticsearch:
- HTTP
  - URL: `http://10.0.1.121:9200`
  - Access: `Server (default)`
- Elasticsearch details
  - Index name: `[logstash-]YYYY.MM.DD`
  - Pattern: `Daily`
  - Time field name: `@timestamp`
  - Version: `5.6+`

Prometheus:
- HTTP
  - URL: `http://10.0.1.121:9090`
  - Access: `Server (default)`


### alerting

Add notification channel in alerting:
- Type: `DingDing`
- Url: `https://oapi.dingtalk.com/robot/send?access_token=81e14f8acb13ddd8310cb3271a56abf059d41441baf1c5b1bd90732792c858c9`
- Message Type: `Link`
- Notification settings:
  - ☑ Default

It uses dingding chatbot in group *BOT运营小组*, and it requires alert message containing string: `RUYI-BOT`.


### dashboard

Dashboard template file:
- `dashboard/redis-instances.json`: revised from [Redis Dashboard for Prometheus Redis Exporter](https://grafana.com/grafana/dashboards/763) by changing var `instance` to custom list and upgrading to new visualization.

To create a new dashboard from it, simply import it, and set name and prometheus data source.

In the new dashboard, change variable `instance` if needed.



## notes

### grafana dashboard

If you want to export dashboard to json file and use it as a template:
- click `share` icon
- then click `export`, enable option `Export for sharing externally`, and click button `Save to file`
- delete entry `uid` in the exported json file


### prometheus admin api

If you want to enable `admin api` in prometheus, use command like below:

```yaml
  prometheus:
    image: prom/prometheus:v2.32.1
    ports:
      - "9090:9090"
    command:
      - "--config.file=/etc/prometheus/prometheus.yml"
      - "--storage.tsdb.path=/prometheus"
      - "--web.console.libraries=/usr/share/prometheus/console_libraries"
      - "--web.console.templates=/usr/share/prometheus/consoles"
      - "--web.enable-admin-api"
    volumes:
      - /data/prometheus/prometheus:/prometheus
      - /data/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
```



## references

Docs:
- [Prometheus Redis Metrics Exporter](https://github.com/oliver006/redis_exporter)

dockerfile:
- [prometheus/Dockerfile](https://github.com/prometheus/prometheus/blob/main/Dockerfile)



