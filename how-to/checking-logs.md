# Checking logs

All moving parts in the MUSIT backend generate logs in some form or another. There are
different categories of logs available for inspection. For all environments, except the
local development envs, logs are sent to a the USIT DS kibana/logstash installation.

How to access logs and perform queries through that system is out of scope of this document.

This document focuses on accessing the logs on the local development environment. The same
technique can be used to investigate logs on the `utv`, `test` and `prod` environments,
provided you have privileged access to these servers.


## Accessing logs in Docker

The main application running inside docker containers are configured to write the logs
to `stdout` and `stderr`. That means logs can be quickly accessed by running the following
command (with sufficient priveleges):

```bash
docker logs -tf <container_name>

# full examples

# Checking the service_backend log
docker logs -tf dev_backend_1

# Checking the nginx log
docker logs -tf dev_nginx_1
```

The `-tf` arguments ensure that the log is followed on the tail. So that it keeps printing
messages as they enter the logs.

Sometimes it might be necessary to investigate other logs inside the docker container. In that
case you will need to start a terminal session inside the container.

```bash
# Starting a bash session in the container
docker exec -it dev_nginx_1 /bin/bash
```

From there you can navigate to the appropriate log locations to dig into the log files. For details
about where you can find logs, please refer to the official documentation of the program you are
investigating.

## The MUSIT backend logs

The MUSIT system generates a few different logs. In addition to being written to the `stdout` of the
docker container, they are written to specific log files in host mapped folder. This means the log
files are made available on the host machine _outside_ the actual docker container. This is the
location where kibana/logstash is looking for logs to parse.

Each MUSIT service generates a log file named `service-<name>.log` (e.g. `service-backend.log`). The
log file is set up to create one new log file per day. Whenever the log file is "rolled" to a new day,
the previous log file will be named `service-<name>.<yyyy-MM-dd>.log`. We keep 30 days worth of log
files on disk. After that time period the oldest log is removed. But still searchable through
kibana/logstash.
