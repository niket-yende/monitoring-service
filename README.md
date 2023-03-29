# monitoring-service
Project to monitor various services

## Running the containers
```bash
$ docker-compose up -d
```

## Bringing down the containers
```bash
$ docker-compose down
```

## Known Issue
The administrator user with clusterMonitor role added by the init-mongo.js script does not work. This mongodb user is required by the mongodb-exporter to establish connection with mongodb.
Check the log content `docker logs -f mongodb-exporter`, you may see the error:
`level=error msg="Cannot connect to MongoDB: cannot connect to MongoDB: connection() error occured during connection handshake: auth error: sasl conversation error: unable to authenticate using mechanism \"SCRAM-SHA-1\": (AuthenticationFailed) Authentication failed."`

### Solution
1. Create the required adminstrator user with clusterMonitor role manually:
   ```bash
    docker exec -it mongodb mongosh -u root -p
    Enter password: root
    
    test> use admin
    switched to db admin
    admin> db.createUser(
      {
        user: "mongodb_exporter",
        pwd: "password",
        roles: [
            { role: "clusterMonitor", db: "admin" },
            { role: "read", db: "local" }
        ]
      }
    )
    ``` 
2.  Add the newly created admin user to the `--mongodb.uri` command of `mongodb-exporter`.
3.  Restart the `mongodb-exporter` container.

