Project Title
Web Server Performance Monitoring using ELK Stack

Objective
The objective of this mini-project is to monitor and analyze the performance of a web server by identifying slow-loading pages and backend inefficiencies. This is achieved using the ELK Stack (Elasticsearch, Logstash, Kibana) on a Debian-based setup.

System Architecture
Component	IP Address	Role
Deb 1	192.168.80.133	Web Server running Apache2
Deb 2	192.168.80.132	Monitoring Server running ELK
Deb 1 hosts a sample web application using Apache2, while Deb 2 runs the ELK stack to collect, process, and visualize logs.

Tools & Technologies Used
Tool/Technology	Purpose
Apache2	Web hosting and access logging
Filebeat	Log shipper from Debian 1 to Logstash
Logstash	Log parser and forwarder to Elasticsearch
Elasticsearch	Log indexing and search engine
Kibana	Dashboard and visualization

Implementation Steps
Step 1: Configure Apache2 on Debian 1
	• Enable status and log modules:
sudo a2enmod status
sudo a2enmod log_config
sudo systemctl restart apache2
	• Customize access log format in /etc/apache2/apache2.conf:
LogFormat "%h %l %u %t \"%r\" %>s %b %D" combined
	• %D provides response time in microseconds.
Step 2: Install and Configure Filebeat on Debian 1
	• Install Filebeat:
sudo apt install filebeat
	• Configure Filebeat (/etc/filebeat/filebeat.yml):
filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - /var/log/apache2/access.log

output.logstash:
  hosts: ["192.168.80.132:5044"]
	• Start and enable Filebeat:
sudo systemctl enable filebeat
sudo systemctl start filebeat
Step 3: Configure Logstash on Debian 2
	• Create pipeline configuration file /etc/logstash/conf.d/apache.conf:
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
	• Start Logstash:
sudo systemctl start logstash
Step 4: Configure Kibana on Debian 2
	• Access Kibana at: http://192.168.80.132:5601
	• Create index pattern: apache-logs
	• Visualizations to create:
		○ Average Response Time Over Time
		○ Top Slowest URLs
		○ HTTP Status Code Breakdown

 Sample Analysis
	• A Kibana dashboard showed response time spikes during peak hours.
	• /contact.php had higher average response times.
	• Backend issue traced to a slow SQL query.

Outcomes
	• Real-time visibility into Apache server performance.
	• Identification of performance bottlenecks.
	• Effective use of ELK for log analysis and monitoring.

Conclusion
This mini-project demonstrates the use of open-source tools to implement a performance monitoring solution for web servers. It provides actionable insights for improving web application performance and scalability.


