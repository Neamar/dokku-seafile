# dokku-seafile
> This is very brittle and likely to break. Seafile is not making any effort to build a docker container outside of their own docker-compose solution. They don't provide config tweaks to use different URLs, nor allow for non-root MySQL user; I actually don't recommend using Seafile with Dokku. Even though Nextcloud has other drawbacks, at least it'll work perfectly with Dokku: https://github.com/Neamar/dokku-nextcloud

## Set up
First, let's create the services we'll need. You may need to install dokku `mysql` and `memcached` plugins if you don't have them already.

```sh
dokku memcached:create seafile -m 256
dokku mysql:create seafile
dokku apps:create seafile
dokku mysql:link seafile seafile
dokku memcached:link seafile seafile
dokku docker-options:add seafile deploy "--link dokku.memcached.seafile:memcached"
dokku docker-options:add seafile build "--link dokku.memcached.seafile:memcached"
dokku docker-options:add seafile run "--link dokku.memcached.seafile:memcached"
```

Next, seafile requires SQL root user. By default, Dokku gives us a "standard" user, so we need to get the root password; run `dokku mysql:info seafile --service-root` to get the path where dokku is storing data. Then `cd` into this directory and `cat` the file called `ROOTPASSWORD`. Take not of this password.

Now let's setup the configuration, don't forget to replace with your values!

```sh
# Update the following with DB values from previous step
dokku config:set seafile DB_HOST=dokku-mysql-seafile DB_ROOT_PASSWD=<root password>
# Define main user
dokku config:set seafile SEAFILE_ADMIN_EMAIL=me@example.com SEAFILE_ADMIN_PASSWORD=asecret
dokku config:set seafile TIME_ZONE=Europe/Paris SEAFILE_SERVER_LETSENCRYPT=false SEAFILE_SERVER_HOSTNAME=<your domain>
dokku storage:mount seafile /mnt/storage/seafile:/shared
```


From the main host, you can then `git push` to dokku's remote.

This will create all the files you need. You should now be able to run `dokku letsencrypt seafile` (or whatever you like to use for SSL).

And you can now access your domain! Don't try to log-in yet, we still need to add memcached.

Edit the file `/mnt/storage/seafile/seafile/conf/seahub_settings.py` and look for the `CACHES` key. For memcache, update the location to be `dokku-memcached-seafile`, this should look like this:

```python
CACHES = {
    'default': {
        'BACKEND': 'django_pylibmc.memcached.PyLibMCCache',
        'LOCATION': 'dokku-memcached-seafile:11211',
    },
```

Then `dokku ps:rebuild seafile` to restart the app.

Do not use dokku's logs feature, you won't get anything. Instead, in your storage mount point, look for a `logs` folder.

> Careful: since we're not using Dokku's standard user, Dokku's DB backup may not work as expected. Test your recovery system before using this in production.
