## Securing our Prometheus servers using Basic Auth and TLS(https)

This article is a last part of the [series]() of my articles on setting up a seamless monitoring system using Prometheus and Grafana. In this article, we will look how to secure our Prometheus server using Basic Auth and TLS.

In this article, I am going to use my Local Prometheus server for demonstration. You can also use the same steps to secure your Prometheus server running on any cloud provider or Local server.

If you haven't read my previous articles, I would recommend you to read them first. You can find the links to my previous articles at the end of this article.

Let's start by understanding what is Basic Auth and TLS.

We all know that security is very important for any software. We also need our Prometheus server secure to avoid any unauthorized access.

There are many ways to secure our Prometheus server. In this article, we are going to use Basic Auth and TLS to secure our Prometheus server.

### What is Basic Auth?

Basic Auth is nothing but simple authentication method of using a username and password to access the server. It is very easy to implement and widely used to secure the server.

### What is TLS?

TLS stands for Transport Layer Security. It provides secure communication between the client and the server. 

### Steps

#### Step 1: Generating a bcrypt password

In order to use Basic Auth in Prometheus, we need to generate a bcrypt password. We can generate a bcrypt password using the htpasswd command.

```bash
htpasswd -nbBC 10 "username"
```

Replace the username with your desired username. It will prompt you to enter your password. Once you enter your password, it will generate a bcrypt password for you.

#### Step 2: Creating a Web Config file

Now, we need to create a web config file for Prometheus. Create a file named `web.yml` and add the following content to it.

```yaml
basic_auth_users:
  username: "bcrypt_password"
```

#### Step 3: Gnerating a TLS certificate

For TLS, we need to generate a TLS certificate. We can generate a Self-signed TLS certificate using the following command. But remember one thing that Self-signed certificates are not recommended for any important environment. We can use self-signed certificates for testing purposes.

```bash
openssl req -new -newkey rsa:4096 -x509 -sha256 -days 365 -nodes -out prometheus.crt -keyout prometheus.key -subj "/C=IN/ST=Karnataka/L=Bangalore/O=Prometheus/CN=localhost"
```
Replace the values of the `-subj` flag with your desired values. Don't forget if want to secure your Production or any important environment, you should use a valid TLS certificate signed by a Certificate Authority. But for learning purposes, We can use a self-signed certificate.

#### Step 4: Updating the Web Config file

Now, we need to update the web config file with the TLS certificate and key. Add the following content to the `web.yml` file.

```yaml
basic_auth_users:
  username: "bcrypt_password"

tls_server_config:
  cert_file: "prometheus.crt" # You need to define the correct path where your certificate is.
  key_file: "prometheus.key" # You need to define the correct path where your key is.
```

#### Step 5: Running Prometheus

We need to Pass our web.yaml file to the Prometheus server. So that Prometheus can use the web.yaml file to secure the server.

Based on how you set up your Prometheus server, you can pass the web.yaml file to the Prometheus server.

If you started the Prometheus server by executing the Prometheus binary, you can pass the web.yaml file as a flag.
```bash
.\prometheus.exe --config.file=prometheus.yml --web.config.file=/path/to/web.yaml
```
If you are running Prometheus as a service, you can pass the web.yaml file in the service file.

```bash
ExecStart=/usr/local/bin/prometheus --config.file=/etc/prometheus/prometheus.yml --web.config.file=/etc/prometheus/web.yml
```

#### Step 6: Accessing Prometheus

Now we can access our Prometheus server using the URL `https://localhost:9090`. It will prompt you to enter your username and password. Enter your username and password that you generated using the htpasswd command.



You can see that our Prometheus server is secured using Basic Auth and TLS. We are using a self-signed certificate. So, our browser will show a warning message that the connection is not secure. To avoid this warning message, we need to use a valid TLS certificate signed by a Certificate Authority.

But there is one more problem we need to solve. If you navigate to targets, you can see our prometheus server is not able to scrape it's own metrics. Since we have enabled Basic Auth and TLS, we need to tell Prometheus to use Basic Auth and TLS to scrape it's own metrics.

#### Step 7: Updating the Prometheus Config file

We need to update the Prometheus config file to use Basic Auth and TLS to scrape it's own metrics. Add the following content to the `prometheus.yml` file.

```yaml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093
          - 'localhost:9097'

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"
    - "rules/alerts.yaml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"
    scheme: https
    tls_config:
      ca_file: 'public.crt'
      server_name: 'localhost' # For self signed certificates we need to add our domain/server name, If you have a valid certificate, you can skip this line.
      insecure_skip_verify: true
    basic_auth:
      username: "userprom"
      password: "grafana"
    static_configs:
      - targets: ["localhost:9090"]
```

Now, restart the Prometheus server. Navigate to targets page, you can see that our Prometheus is able to scrape it's own metrics. You can see that our Prometheus server is secured using Basic Auth and TLS.

That's all for this article. I hope you like this article. If you have any queries, Let me know in the comments.
