# About

[Elastic](https://www.elastic.co/guide/en/elastic-stack-get-started/current/get-started-docker.html) does not provide a dockerized solution with Logstash integrated. This repository is based on Elastic's dockerized setup and Logstash has been integrated into a compose file. All communication between containers are performed using TLS with self-signed certificates. Validation of certificates is disabled.

If you choose to use this solution, follow the steps below and add your filters to Logstash, or eventually use Auditbeat.

# How to install
Before proceeding, ensure that vm.max\_map\_count is set to a sufficient value, otherwise ELK might not be able to start.

```
sysctl -w vm.max_map_count=262144
echo "vm.max_map_count=262144" >> /etc/sysctl.conf
```

**Step 1** Define versions and instances. 

```
cp ~/elk/env.example ~/elk/.env
cp ~/elk/setup/instances.example.yml ~/elk/instances.yml
```

**Step 2** Create certificates

```
sudo docker-compose -f ~/elk/create-certs.yml run --rm create_certs
```

**Step 3** Start containers

```
sudo docker-compose -f ~/elk/elastic-docker-tls.yml up -d
```

**Step 4** Generate passwords

```
sudo docker exec es01 /bin/bash -c "bin/elasticsearch-setup-passwords auto --batch --url https://es01:9200" | tee -a passwords.txt
```

**Step 5** Update *.env* with elastic username and password. Credentials for *Kibana* and *Logstash* in the *elastic-docker-tls.yml* and *logstash/pipeline/9-output.conf* are set automatically from *.env* file.

**Step 6** Restart containers. Kibana might need 5-10 minutes to be ready at https://SERVERIP:5601 depending on your hardware.

```
sudo docker-compose stop && sudo docker-compose -f ~/elk/elastic-docker-tls.yml up -d
```
