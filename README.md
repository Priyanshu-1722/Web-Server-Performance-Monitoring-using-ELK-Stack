# Web Server Performance Monitoring using ELK Stack

## 📌 Objective
The objective of this mini-project is to monitor and analyze the performance of a web server by identifying slow-loading pages and backend inefficiencies. This is achieved using the **ELK Stack** (Elasticsearch, Logstash, Kibana) on a Debian-based environment.

---

## 🖥️ System Architecture

| Component | IP Address      | Role                                |
|-----------|------------------|-------------------------------------|
| Deb 1     | 192.168.80.133  | Web Server running Apache2          |
| Deb 2     | 192.168.80.132  | Monitoring Server running ELK Stack |

- **Deb 1** hosts a sample web application using Apache2.
- **Deb 2** collects, parses, and visualizes logs using the ELK stack.

---

## 🔧 Tools & Technologies Used

| Tool/Technology | Purpose                                      |
|------------------|----------------------------------------------|
| Apache2          | Web hosting and access logging               |
| Filebeat         | Log shipper (from Deb 1 to Logstash)         |
| Logstash         | Log parser and forwarder to Elasticsearch    |
| Elasticsearch    | Log indexing and search engine               |
| Kibana           | Dashboard and data visualization             |

---

## ⚙️ Implementation Steps

### 🧩 Step 1: Configure Apache2 on Debian 1
```bash
sudo a2enmod status
sudo a2enmod log_config
sudo systemctl restart apache2
````

* Customize access log format in `/etc/apache2/apache2.conf`:

```apache
LogFormat "%h %l %u %t \"%r\" %>s %b %D" combined
```

* `%D` logs response time in microseconds.

---

### 📦 Step 2: Install and Configure Filebeat on Debian 1

```bash
sudo apt install filebeat
```

* Edit `/etc/filebeat/filebeat.yml`:

```yaml
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /var/log/apache2/access.log

output.logstash:
  hosts: ["192.168.80.132:5044"]
```

* Enable and start Filebeat:

```bash
sudo systemctl enable filebeat
sudo systemctl start filebeat
```

---

### 🔄 Step 3: Configure Logstash on Debian 2

* Create pipeline config at `/etc/logstash/conf.d/apache.conf`:

```ruby
input {
  beats {
    port => 5044
  }
}

filter {
  grok {
    match => { "message" => "%{COMMONAPACHELOG} %{NUMBER:response_time}" }
  }
  mutate {
    convert => { "response_time" => "integer" }
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "apache-logs"
  }
}
```

* Start Logstash:

```bash
sudo systemctl start logstash
```

---

### 📊 Step 4: Configure Kibana on Debian 2

* Access Kibana at: `http://192.168.80.132:5601`
* Create index pattern: `apache-logs`
* Recommended visualizations:

  * 📈 Average Response Time Over Time
  * 🐢 Top Slowest URLs
  * 🔁 HTTP Status Code Breakdown

---

## 🔍 Sample Analysis

* Kibana dashboard indicated response time spikes during peak hours.
* `/contact.php` showed consistently higher average response times.
* Root cause: slow SQL query in backend processing.

---

## ✅ Outcomes

* Real-time monitoring of Apache server performance.
* Easy identification of performance bottlenecks.
* Effective utilization of ELK stack for log analysis.

---

## 🧾 Conclusion

This mini-project showcases the power of open-source monitoring tools in identifying and resolving web server performance issues. The ELK Stack provides actionable insights that can be used to improve application performance and ensure scalability.

---

## 📁 Project Directory Structure

```text
├── apache.conf (Logstash config)
├── filebeat.yml (Filebeat config)
└── README.md
```
