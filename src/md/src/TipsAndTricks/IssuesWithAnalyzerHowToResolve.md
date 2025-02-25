## Resolve Analyzer Known Issues

### Problem 1. Auto-Analysy doesn't work. Analyzer health check status failed: Elasticsearch is not healthy

#### Problem Description
Analyzer log:

```
2021-09-09 11:34:47,927 - analyzerApp - ERROR - Analyzer health check status failed: Elasticsearch is not healthy;
[pid: 10|app: 0|req: 1/3] 127.0.0.1 () {28 vars in 294 bytes} [Thu Sep  9 11:34:46 2021] GET / => generated 43 bytes in 1643 msecs (HTTP/1.1 503) 3 headers in 120 bytes (1 switches on core 0)
2021-09-09 11:35:48,737 - analyzerApp.utils - ERROR - Error with loading url: http://elasticsearch:9200/_cluster/health
2021-09-09 11:35:48,752 - analyzerApp.utils - ERROR - HTTPConnectionPool(host='elasticsearch', port=9200): Max retries exceeded with url: /_cluster/health (Caused by NewConnectionError('<urllib3.connection.HTTPConnection object at 0x7f5cb82d4290>: Failed to establish a new connection: [Errno 111] Connection refused'))
2021-09-09 11:35:48,753 - analyzerApp.esclient - ERROR - Elasticsearch is not healthy
2021-09-09 11:35:48,753 - analyzerApp.esclient - ERROR - list indices must be integers or slices, not str
```

ElasticSearch container restarting all the time:

```
STATUS                                     NAMES
Up Less than a second (health: starting)   reportportal_elasticsearch_1
```

#### Solution

Create a directory for ElasticSearch and assign permissions with the following commands

```bash
mkdir -p data/elasticsearch
chmod 777 data/elasticsearch
chgrp 1000 data/elasticsearch
```

Recreate ReportPortal services.

### Problem 2. Auto-Analysy doesn't work. KeyError: 'found_test_and_methods' not found

#### Problem Description

```
2021-09-09 11:35:48,737 - analyzerApp.utils - ERROR - KeyError: 'found_test_and_methods' not found
```

#### Solution

Regenerate index in the ElasticSearch. Project settings -> Auto-Analysis -> Genetate Index

![Regenerate index](Images/autoanalyzer-generate-index.gif)


### Problem 3. Amqp connection was not established

#### Problem Description

```
2021-09-09 11:32:00,579 - analyzerApp - INFO - Starting waiting for AMQP connection
2021-09-09 11:32:00,586 - analyzerApp.amqp - INFO - Try connect to amqp://rabbitmq:5672/analyzer?heartbeat=600
2021-09-09 11:32:00,595 - analyzerApp - ERROR - Amqp connection was not established
```

#### Solution

RabbitMQ container is not running. Wait for status `running` or recreate the RabbitMQ container.

### Problem 4. Perfomance

#### Problem Description

Slowing down analysis or waiting for a long time fore responce.

Analyzer logs:

```
DAMN ! worker 1 (pid: 9191) died, killed by signal 9 :( trying respawn ...
Respawned uWSGI worker 1 (new pid: 9490)
```

#### Solution

Increase VM stats. We recommend using the minimum memory:
* [Analyzer](https://github.com/reportportal/reportportal/blob/master/docker-compose.yml#L56) - 1024 Mb
* [Analyzer train](https://github.com/reportportal/reportportal/blob/master/docker-compose.yml#L69) - 512 Mb

Also you can reduce the number of Analyzer processes with processing environment variable `UWSGI_WORKERS: 2` (default `4`), then:
* [Analyzer](https://github.com/reportportal/reportportal/blob/master/docker-compose.yml#L56) - 768 Mb

However, `UWSGI_WORKERS` will slow down the Analyzer.
