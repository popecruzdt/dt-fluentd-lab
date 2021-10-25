### SSH
```
ssh -i <key-name.pem> ubuntu@<hostname-or-ip-addr>
```
### send json log message to dynatrace log ingest API
create log message contents in json format
```
nano log-ingest-api.json

{
    "level": "INFO",
    "content": "this is a generic log ingest api message",
    "log.source": "generic log ingest",
    "dt.entity.custom_device": "CUSTOM_DEVICE-<ID>"
}
```
create script to send log message to the dynatrace log ingest api
```
nano send-to-dynatrace.sh

curl -ki https://<active-gate>:9999/e/<environment>/api/v2/logs/ingest --data-binary "@log-ingest-api.json" -H 'Content-Type: application/json; charset=utf-8' -H 'Authorization: Api-Token <API-TOKEN>'
```
execute script to send log message to the dynatrace log ingest api
```
./send-to-dynatrace.sh

HTTP/1.1 204 No Content
```
add a cron to execute script every 1 minute
```
crontab -e

*/1 * * * * /root/send-to-dynatrace.sh >/dev/null 2>&1
```
### install fluentd (td-agent)
https://docs.fluentd.org/installation
install td-agent for ubuntu
```
curl -fsSL https://toolbelt.treasuredata.com/sh/install-ubuntu-bionic-td-agent4.sh | sh
systemctl status td-agent.service
```
### install fluent-plugin-dynatrace
https://github.com/dynatrace-oss/fluent-plugin-dynatrace
install dynatrace fluentd plugin
```
td-agent-gem install fluent-plugin-dynatrace
```
try again with dependencies addressed
```
apt-get install ubuntu-dev-tools
td-agent-gem install fluent-plugin-dynatrace
```
modify td-agent configuration
```
nano /etc/td-agent/td-agent.conf

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
