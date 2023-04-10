# monitoring-service
Project to monitor various services

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
