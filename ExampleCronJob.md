# Example Cron Job #

The following cron job assumes that:
  * App RAILS\_ROOT: /u/apps/hydra/current
  * App environment: production
  * Sync users/torrents every 5 minutes
  * Sync transfer stats daily @ 1am

The cron job:

```

# Sync Users and Torrents Every 5 Minutes
0-59/5 * * * * RAILS_ENV=production /usr/bin/ruby /u/apps/hydra/current/script/runner -e production 'Sync.sync_every_five' > /dev/null 

# General Daily Cron - Sync transfer stats @ 1am
0 1 * * * RAILS_ENV=production /usr/bin/ruby /u/apps/hydra/current/script/runner -e production 'Sync.sync_daily' > /dev/null 

```

[See Here](http://www.unixgeeks.org/security/newbie/unix/cron-1.html) for a crontab tutorial.

Normally the workflow is just:
```
crontab -e
# Opens up your default text editor -- paste in your cron jobs
# Save the file and exit, and your new cron jobs should now be installed

# View your cron list with:
crontab -l
```