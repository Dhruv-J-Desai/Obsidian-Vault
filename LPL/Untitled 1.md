---

---
1. Below command is used to install Dynatrace OneAgent on a host system (likely an ARM-based Linux system). Dynatrace OneAgent is a monitoring tool that collects system metrics, traces, logs, and performance data for infrastructure and applications.
```sh
docker run --rm --privileged --pid=host --net=host \
  -v /:/mnt/root \
  -e ONEAGENT_INSTALLER_SCRIPT_URL="https://ofn16441.live.dynatrace.com/api/v1/deployment/installer/agent/unix/default/latest?arch=arm" \
  -e ONEAGENT_INSTALLER_DOWNLOAD_TOKEN="dt0c01.ZKP4XQRWJQUPI5SIINOG4N4Q.LXGRNEE4JJD5GD5RMI47A2H6VEXXDTQP2HLBLB5M6JAVQVABYNQVV34NBLHS6W7G" \
  dynatrace/oneagent
```

2. This curl command sends an authenticated HEAD request to the Dynatrace API to check the latest OneAgent installer for ARM-based UNIX systems. It uses an API token for access and only fetches the HTTP headers, not the full installer.
```sh
curl -I -H "Authorization: Api-Token dt0c01.ZKP4XQRWJQUPI5SIINOG4N4Q.LXGRNEE4JJD5GD5RMI47A2H6VEXXDTQP2HLBLB5M6JAVQVABYNQVV34NBLHS6W7G" \  "https://ofn16441.live.dynatrace.com/api/v1/deployment/installer/agent/unix/default/latest?arch=arm"
```
```sh
HTTP/1.1 200 OK
date: Wed, 15 Oct 2025 19:22:34 GMT
vary: Accept-Encoding
content-disposition: attachment; filename="Dynatrace-OneAgent-Linux-1.323.40.sh"; size=82031894
etag: "11b9c464d495838a2734b6a8850c3d8604eaca7be718f89bf58609d8fea1d07e"
content-encoding: none
content-type: application/octet-stream
cache-control: no-store, no-cache
pragma: no-cache
dynatrace-response-source: Cluster
x-robots-tag: noindex
server: ruxit server
content-length: 82031894
strict-transport-security: max-age=31536000;includeSubDomains
```

3. Broker health...CPU, memory
4. Total number of partitions
5. Number of consumers
6. Consumer wait time, Poll time, min. fetch bytes
7. 