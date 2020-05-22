# dokku-seafile

```sh
dokku mysql:create seafile
dokku apps:create seafile
dokku mysql:link seafile seafile
# Update the following with DB values from previous step
dokku config:set seafile DB_HOST=dokku-mysql-seafile DB_ROOT_PASSWD=db_dev
# Define main user
dokku config:set seafile SEAFILE_ADMIN_EMAIL=me@example.com SEAFILE_ADMIN_PASSWORD=asecret
dokku config:set seafile TIME_ZONE=Europe/Paris SEAFILE_SERVER_LETSENCRYPT=false SEAFILE_SERVER_HOSTNAME=docs.seafile.com
dokku storage:mount seafile /mnt/storage/seafile:/shared
```
