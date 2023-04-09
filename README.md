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
2. Update the below fields to receive the email alerts:
   ```
   auth_username: <EMAIL_AUTH_USERNAME>
   auth_password: <EMAIL_AUTH_PASSWORD>
   from: <EMAIL_FROM_ADDRESS>
   to: <EMAIL_TO_ADDRESS>
   ```
3. We can also update the email template as per our requirement.