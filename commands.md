### SSH
connect to instance via ssh using private key
```
ssh -i <key-name.pem> ubuntu@<hostname-or-ip-addr>
```
become root user
```
sudo su -
```
### send json log message to dynatrace log ingest API
create log message contents in json format
```
nano log-ingest-api.json

{
    "level": "INFO",
    "content": "this is a generic log ingest api message",
    "log.source": "generic log ingest",
    "dt.entity.custom_device": "CUSTOM_DEVICE-<ID>",
    "lab.user": "<your name>",
    "lab.location": "<your location>"
}
```
create script to send log message to the dynatrace log ingest api
```
nano send-to-dynatrace.sh
```
```
curl -ki https://<active-gate>:9999/e/<environment>/api/v2/logs/ingest --data-binary "@log-ingest-api.json" -H 'Content-Type: application/json; charset=utf-8' -H 'Authorization: Api-Token <API-TOKEN>'
```
execute script to send log message to the dynatrace log ingest api
```
chmod o+rwx send-to-dynatrace.sh
./send-to-dynatrace.sh
```
>HTTP/1.1 204 No Content

add a cron to execute script every 1 minute
```
crontab -e
```
```
*/1 * * * * /root/send-to-dynatrace.sh >/dev/null 2>&1
```
>crontab: installing new crontab

### install fluentd (td-agent)
install td-agent for ubuntu https://docs.fluentd.org/installation
```
curl -fsSL https://toolbelt.treasuredata.com/sh/install-ubuntu-focal-td-agent4.sh | sh
```
```
systemctl status td-agent.service
```
>td-agent.service - td-agent: Fluentd based data collector for Treasure Data
>     Loaded: loaded (/lib/systemd/system/td-agent.service; enabled; vendor preset: enabled)
>     Active: active (running) since

### install fluent-plugin-dynatrace
install dynatrace fluentd plugin https://github.com/dynatrace-oss/fluent-plugin-dynatrace
```
td-agent-gem install fluent-plugin-dynatrace
```
>Done installing documentation for fluent-plugin-dynatrace after 0 seconds
>1 gem installed

backup td-agent configuration, clear configuration
```
cp /etc/td-agent/td-agent.conf /etc/td-agent/td-agent.conf.original
rm /etc/td-agent/td-agent.conf
```
modify td-agent configuration, add dynatrace integration
```
nano /etc/td-agent/td-agent.conf
```
```
# HTTP input
# POST http://localhost:8888/<tag>?json=<json>
# POST http://localhost:8888/td.myapp.login?json={"user"%3A"me"}
# @see http://docs.fluentd.org/articles/in_http
<source>
  @type http
  @id input_http
  port 8888
</source>

# Dynatrace output
<match dt.*>
  @type              dynatrace
  active_gate_url    https://<activegate>:9999/e/<environment>/api/v2/logs/ingest
  api_token          <API-TOKEN>
  ssl_verify_none    true
  <buffer>
   @type memory
   flush_mode interval
   flush_interval 30s
   flush_thread_count 5
   chunk_limit_size 12m
   queue_limit_length 10000
   chunk_limit_records 450
   retry_wait 1m
   retry_type exponential_backoff
  </buffer>
</match>
```
restart td-agent to update configuration
```
systemctl restart td-agent.service
```
### send json log message to fluentd
create log message contents in json format
```
nano log-fluentd.json
```
```
{
    "level": "INFO",
    "content": "this is a fluentd integration log message",
    "log.source": "fluentd integration",
    "dt.entity.custom_device": "CUSTOM_DEVICE-<ID>",
    "lab.user": "<your name>",
    "lab.favorite": "<your favorite>"
}
```
create script to send log message to fluentd
```
nano send-to-fluentd.sh
```
```
curl -ki http://localhost:8888/dt.test --data-binary "@log-fluentd.json" -H 'Content-Type: application/json'
```
execute script to send log message to fluentd
```
chmod o+rwx send-to-fluentd.sh
./send-to-fluentd.sh
```
>HTTP/1.1 200 OK

add a cron to execute script every 1 minute (keep existing cron as well)
```
crontab -e
```
```
*/1 * * * * /root/send-to-fluentd.sh >/dev/null 2>&1
```
>crontab: installing new crontab
