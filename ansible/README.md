# awx-eda-project

Example Project for AWX and EDA

## Rulebook Activation Variables

```YAML
mqtt_host: mqtt.cgerber.local
mqtt_port: 31883
mqtt_topic: server-events
job_template_name: Scale Up K8s Service
```

## Sending Rulebook Events

```Shell
bin/send_event "example-webapp" "ScaleUp" "CPU High on example-webapp"
```

```Shell
bin/send_event "example-webapp" "ScaleDown" "CPU Low on example-webapp"
```
