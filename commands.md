### SSH
```
ssh -i <key-name.pem> ubuntu@<hostname-or-ip-addr>
```
### send json log message to dynatrace log ingest API
create log message contents in json format
```
nano log-ingest-api.json
########################
{
    "level": "INFO"
    "content": "this is a generic log ingest api message",
    "log.source": "generic log ingest",
    "dt.entity.custom_device": "CUSTOM_DEVICE-<ID>",
    "http.response.status_code": 404,
    "http.status_code": 404,
    "http.host": "localhost"
}
```
### install fluentd (td-agent)
```
curl -L https://toolbelt.treasuredata.com/sh/install-ubuntu-bionic-td-agent2.5.sh | sh
systemctl status td-agent.service
```
### install fluent-plugin-dynatrace
https://github.com/dynatrace-oss/fluent-plugin-dynatrace
```
td-agent-gem install fluent-plugin-dynatrace
```
```
apt-get install ubuntu-dev-tools
td-agent-gem install fluent-plugin-dynatrace
```
