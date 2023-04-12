# monitoring-service
Project to monitor various services

## Steps:
1. Start the containers: <br />
   `docker-compose up -d`
1. Initialize the mongodb replica set: <br />
   `docker exec mongodb1 /scripts/rs-init.sh`
1. Restart the mongodb-exporter: <br />
   `docker restart mongodb-exporter`
1. Create a redis cluster: <br />
   `docker exec -it redis1 redis-cli -a eYVX7EwVmmxKPCDmwMtyKVge8oLd2t81 --cluster create --cluster-replicas 0 172.29.0.10:6379 172.29.0.11:6380 172.29.0.12:6381 --cluster-yes`
1. Restart the redis-exporter: <br />
   `docker restart redis-exporter`

## Start monitoring services
```bash
$ ./start-monitoring.sh
```

## Stop monitoring services
```bash
$ ./stop-monitoring.sh
```

## Grafana (Please change the default login credentials)
```
user: admin
password: admin
```

## Configuring the email alerts using alertmanager
1. Navigate to the config.yml file inside the config/alertmanager directoy.
1. Update the below fields to receive the email alerts:
   ```
   auth_username: <EMAIL_AUTH_USERNAME>
   auth_password: <EMAIL_AUTH_PASSWORD>
   from: <EMAIL_FROM_ADDRESS>
   to: <EMAIL_TO_ADDRESS>
   ```
1. We can also update the email template as per our requirement.

## Steps to monitor mongodb replica sets: (Some of these steps only applicable if mongo is auth enabled)
1. Generate a shared secret key: <br />
	`openssl rand -base64 741 > replica.key.devel`
1. Pass this replica key while running the mongo command for each replicaset: <br />
	command: `mongod --replSet rs0 --auth --bind_ip_all --keyFile /data/replica.key`
1. Mount this key file in the volume.
1. Once all the mongo replicaset instances are ready, initialize them with below command: <br /> 
	`docker exec mongodb1 /scripts/rs-init.sh`
1. Check if the replicaset is configured correctly with below commands: <br />
	```
   docker exec -it mongodb1 mongosh -u root -p
	Enter password: root
	
   rs.status() - It will list all the members of the replicaSet out of which only one would be primary node.
   ```
1. Make sure the mongodb-exporter is running with the below command: <br />
	command: `--mongodb.uri mongodb://exporter:password@mongodb1:27017/admin?replicaSet=rs0&ssl=false&tls=false`
1. Restart the mongodb-exporter: <br />
	`docker restart mongodb-exporter`
1. The exporter only allows single replica set member:
	```
   To monitor a MongoDB replica set with the Percona MongoDB exporter, you need to specify a single replica set member in the MONGODB_URI environment variable. The exporter will then discover the other replica set members by querying the replica set configuration.
   ```

## Steps to monitor redis cluster:
1. Atleast 3 instances of redis must be added to form a cluster.
1. Each redis startup command would look like below: <br />
   `redis-server --port 6379 --cluster-enabled yes --cluster-config-file /data/nodes.conf --cluster-node-timeout 5000 --requirepass $$REDIS_PASSWORD`
1. Add the redis_exporter_targets in the prometheus.yml file: <br />
   ```yml
   - job_name: 'redis_exporter_targets'
    static_configs:
      - targets:
        - redis://redis1:6379
        - redis://redis2:6380
        - redis://redis3:6381
    metrics_path: /scrape
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: redis-exporter:9121
   ```
1. Command to create redis cluster after all redis instances are up: <br />
   `docker exec -it redis1 redis-cli -a eYVX7EwVmmxKPCDmwMtyKVge8oLd2t81 --cluster create --cluster-replicas 0 172.29.0.10:6379 172.29.0.11:6380 172.29.0.12:6381 --cluster-yes`
1. Restart the redis-exporter: <br />
   `docker restart redis-exporter`
1. Check if all redis instances are being monitored: <br />
   http://localhost:9121/metrics <br />
	http://localhost:9121/scrape?target=redis1:6379 <br />
	http://localhost:9121/scrape?target=redis2:6380 <br />
	http://localhost:9121/scrape?target=redis3:6381 <br />