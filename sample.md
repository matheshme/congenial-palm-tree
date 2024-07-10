## Setting up a Alerting System in Prometheus for sending Alerts to Slack

This article is a continuation of the previous article on [Setting up Prometheus](). If you didn't read that first part, I would recommend you to read that first. In this article, we will see how to set up a alerting system in Prometheus for sending alert notifications to Slack. 

Also I have one tip, If you are trying all these things for just learning purpose, I would recommend you to use Local Host Machine instead of using AWS EC2 instance. Because, AWS EC2 instance is not free and you will be charged for using it. So, if you are just learning, use Local Host Machine.

Issues I had with AWS EC2 instance: If you already read the previous article, you know I used `t2.large` instance type for setting up Prometheus and Grafana. But, still I faced some issues like server crash sometimes I don't know why. So, Now I switched to my Local Host Machine and I don't have any issues now. So, I would recommend you to use Local Host Machine for learning purpose. If you have purpose other than learning, Please have your own choice of using Cloud Providers or Linux Machines.

Now, We already know how to set up Prometheus and Grafana and How to create a dashboard in Grafana. Now, Let's see how to set up a Alerting System in Prometheus for sending Alerts to Slack.

### Step 1: Setting up Alert Manager

Before setting up Alert Manager, Let's see why we need Alerts in Prometheus? and what is AlertManager in Prometheus?

#### Why alerts are needed?
Let's say we have a website running on a server. We configured prometheus to monitor the website. Prometheus is collecting the metrics like CPU, Memory, Disk, Network, I/O etc. Now let's say our website is down. We don't know that our website is down. And after some our customers are calling us and saying that they are not able to access the website. Now It's not a ideal approach. Because if our customers are not satisfied with our service, they will move to another service. So we need to know that our website is down before our customers know. In this case we need to setup some alerts. So that we will get notified when our website is down.

### Why we need AlertManager?

Normally if we define any alerts in Prometheus it will only raised in Prometheus Web UI. So to get our alerts as notifications we need to use AlertManager. AlertManager will convert the alerts from Prometheus to the notification format and send the alerts to the notification channels like Email, Slack, PagerDuty etc. AlertManager comes with Web UI to manage the alerts. AlertManager will expose on port 9093. If we want to configure AlertManager behavior we have to configure the AlertManager behavior in the alertmanager.yml file.

#### What is AlertManager?

Alertmanager is a component of Prometheus that convert alerts from Prometheus to the notification format and send the alerts to the notification channels like Email, Slack, PagerDuty etc. Alertmanager comes with Web UI to manage the alerts. Alertmanager will expse on port 9093. If we want to cnfigure Alertmanager behaviour we have to configure the Alertmanager behavior in the alertmanager.yml file.

Now, Let's see how to set up AlertManager.

#### Setting up AlertManager in Linux Machine for Development Purpose

- Get the alert package from the [official site](https://prometheus.io/download/#alertmanager). Copy the link of the tar.gz file and use the command `wget` to download the file.
```bash
wget https://github.com/prometheus/alertmanager/releases/download/v0.27.0/alertmanager-0.27.0.linux-amd64.tar.gz
```

- Extract the downloaded file.
```bash
tar -xvf alertmanager-0.27.0.linux-amd64.tar.gz
```

- Change the directory to the extracted directory.
```bash
cd alertmanager-0.27.0.linux-amd64
```

- Execute the `alertmanager` binary file.
```bash
./alertmanager # If you want to use any other ports use "--web.listen-address" flag with the port number. Example: ./alertmanager --web.listen-address=":9094"
```

#### Setting up AlertManager as a Service in Linux Machine for Production Purpose

If you are setting up AlertManager for some important purpose, I would recommend you to set up AlertManager as a service. So that, If the server restarts, AlertManager will start automatically. Also if we just execute the `alertmanager` binary file, It will get terminated once we close the terminal. So, It's better to set up AlertManager as a service.

So instead of executing the `alertmanager` binary file, You can follow the below steps to set up AlertManager as a service.

- To make ur `alertmanager` installation clean, We can create one Separate directory for `alertmanager` and move the extracted files to that directory.
```bash
sudo mkdir /var/lib/alertmanager
sudo mv alertmanager-0.27.0.linux-amd64/* /var/lib/alertmanager
```

- Go to the `alertmanager` directory.
```bash
cd /var/lib/alertmanager
```

- Now grant our `prometheus` user the ownership and access for the `alertmanager` directory.
```bash
sudo chown -R prometheus:prometheus /var/lib/alertmanager
sudo chown -R prometheus:prometheus /var/lib/alertmanager/*
sudo chmod -R 775 /var/lib/alertmanager
sudo chmod -R 775 /var/lib/alertmanager/*
```

- Now we can execute `alertmanager` binary file if we want. But if we are starting our Alertmanager as Process it will get terminated once we close the terminal. So we have to start the Alertmanager as a service. We can create a service file for the Alertmanager and start the Alertmanager as a service.

- We need to create one storage directory for Alertmanager. We can create one directory called `data` in the `/var/lib/alertmanager` directory.
```bash
sudo mkdir /var/lib/alertmanager/data
```

- Create a service file for Alertmanager.
```bash
sudo vi /etc/systemd/system/alertmanager.service

# Add the below content to the file
[Unit]
Description=Prometheus Alertmanager
Documentation=https://prometheus.io/docs/introduction/overview/
Wants=network-online.target
After=network-online.target

[Service]
Type=simple
User=prometheus
Group=prometheus
ExecReload=/bin/kill -HUP $MAINPID
ExecStart=/var/lib/alertmanager/alertmanager --storage.path="/var/lib/alertmanager/data" --config.file="/var/lib/alertmanager/alertmanager.yml"

SyslogIdentifier=prometheus_alert_manager
Restart=always

[Install]
WantedBy=multi-user.target
```

- Reload the systemd daemon and start the Alertmanager service.
```bash
sudo systemctl daemon-reload
sudo systemctl start alertmanager
sudo systemctl enable alertmanager
```

- To check the status of Alertmanager service.
```bash
sudo systemctl status alertmanager
```

Now, We can access the Alertmanager Web UI by using the URL `http://localhost:9093`. You can see the Alertmanager Web UI.

### Step 2: Configuring Alerts

Now, We have set up the Alertmanager. Now, Let's see how to configure the alerts in Prometheus.

Since we already have one target in Prometheus, We are now going to set up a simpe alert for that target.

- Create a folder called `rules` in the location where the `prometheus.yml` file is located. Let's say I have my `prometheus.yml` file in the location `/etc/prometheus/`. So I will create a folder called `rules` in the `/etc/prometheus/` location. And then I will create a file called `alert.yaml` in the `rules` folder.
```bash
sudo mkdir /etc/prometheus/rules
sudo vi /etc/prometheus/rules/alert.yaml
```

- Copy the below content to the `alert.yaml` file.
```yaml
  - alert: PrometheusTargetMissing
    expr: up == 0
    for: 0m
    labels:
      severity: critical
    annotations:
      summary: Prometheus target missing (instance {{ $labels.instance }})
      description: "A Prometheus target has disappeared. An exporter might be crashed.\n  VALUE = {{ $value }}\n  LABELS = {{ $labels }}"
```

What the above alert does is, It will check whether our targets is up or not. If the target is down, It will send an alert to the Alertmanager with the details of which target is down.

- Now, We have to include the `alert.yaml` file in the `prometheus.yml` file. So that Prometheus will read  `alert.yaml` file and it will start sending the alerts to the Alertmanager.
```bash
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration, Prometheus will use this to send alerts to Alertmanager.
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          - 'localhost:9097'

# Location of the alert rules.
rule_files:
    - "rules/alerts.yaml"


scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]

  - job_name: 'application-server'
    static_configs:
      - targets: ['3.90.108.255:9100']
```

- Now, We have to restart the Prometheus service to apply the changes. If you are running as a docker container, You can restart the container using `docker restart container_id`. If you are running as a service, You can restart the service.
```bash
sudo systemctl restart prometheus
```

- Now, We have to check whether the alerts are working or not. We can check the alerts in the Alertmanager Web UI. You can see the alerts in the Alertmanager Web UI.

### Step 3: Sending Alerts to Slack

Now, We have set up the Alertmanager and to send the alerts to the Slack, We have to configure the Slack in the Alertmanager. Let's see how to configure the Slack in the Alertmanager.

- Now we have to update the `alertmanager.yml` file to send the alerts to the Slack. You can find the `alertmanager.yml` file in the location of the extracted `alertmanager` installation directory. 

```yaml
global:
  slack_api_url: 'https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXXXXXX'

route:
  # The below receiver is a Default receiver. If the alert doesn't match any of the receivers in routes section, It will send the alert to the default receiver.
  receiver: 'slack-notifications'
  # To send alerts to different receivers based on different conditions, We can use the "routes" section.
  # routes:
  #   - match:
  #       severity: critical
  #     receiver: 'slack-notifications'
  #   - match:
  #       severity: warning
  #     receiver: 'slack-notifications'
  

receivers:
  - name: 'slack-notifications'
    slack_configs:
      - send_resolved: true
        channel: '#channel_name'
        icon_emjoi: ':warning:'
```
- Now you can see in the above configuration, We have to provide the `slack_api_url`. You can get the `slack_api_url` by creating a Slack App. You can create a Slack App by following the below steps.

  - Go to the [Slack API](https://api.slack.com/apps) and click on the `Create New App` button.
  - Give a name to the App and select the workspace where you want to send the alerts.
  - Click on the `Create App` button.
  - Now, You will be redirected to the App dashboard. Click on the `Incoming Webhooks` option.
  - Turn on the `Incoming Webhooks` option.
  - Click on the `Add New Webhook to Workspace` button.
  - Select the channel where you want to send the alerts and click on the `Allow` button.
  - Now, You will get the `Webhook URL`. Copy the `Webhook URL` and paste it in the `alertmanager.yml` file.
- Now we have to restart the Alertmanager service to apply the changes. If you are running alertmanager as a docker container, You can restart the container using `docker restart container_id`. If you are just executing the `alertmanager` binary file, You can restart by closing the terminal and executing the `alertmanager` binary file again or If you are running **alertmanager** as a service, You can restart the service.
```bash
sudo systemctl restart alertmanager
```

- Now, We have to check whether the alerts are working or not. We can check the alerts in the Alertmanager Web UI. You can see the alerts in the Alertmanager Web UI.

### Testing the Alerts

Let's test the alert by stopping the target. Since my **Node Exporter** is running on the AWS EC2 instance as a executable file, I will stop the Node Exporter by killing the process. You can stop the target by stopping the service or by killing the process.

Now navigate to the Prometheus Web UI and click on the `Status` option. You can see the `Targets` option. Click on the `Targets` option. You can see the `Node Exporter` target. You can also check the alerts in the alerts tab.

Navigate to the Alertmanager Web UI and you can see the alerts in the Alertmanager Web UI.

Now, You can see the alerts in the Slack channel as well.

