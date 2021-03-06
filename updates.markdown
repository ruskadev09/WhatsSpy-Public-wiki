# WhatsSpy Public updates

This page contains notes about how to update a running instance of WhatsSpy Public. This is always relatively easy. 

**Did you use the Raspberry Pi image installation? No need to update, this will happen automaticlly within 24 hours. HOWEVER: make sure your `config.php` complies with any changes.**

**[Video guide available](https://gitlab.maikel.pro/maikeldus/WhatsSpy-Public/wikis/updates-video) for the update process.**

## Before/After updating

Make sure that you before the update undertake these steps:

* Shutdown the tracker by using `screen -r` and then pressing `Ctrl+c` and `Ctrl+a` and `Ctrl+d`.
* Apply the update: by following the changelog.
* After the update start the tracker again by using `screen -r` and execute `` `which php` tracker.php``.




## Changelog

*At every update you can just re-download the repository and replace it with the current files (make sure you replace and not delete all the files).*

*If you used `git` (as stated in the getting-started) you can execute `cd /var/www/whatsspy/` followed by `git fetch --all
` followed by `git reset --hard origin/master`.*

---
*Unlabeled version* : 

* Fixed bug in overview where the 14 day status count was not displayed correct (thanks anomynous mailer!)
* Implemented assigning groups to new contact.
* Fixed invisible element overlapping "Expand view" and "Edit" buttons on low resolution screens.

---
**Update 1.5.6** : 

* Fixed notice errors in JSON webservice.
* Fixed exit on unknown message response from WhatsApp.
* Added ability to finetune online/offline times with heuristic options (in the `config.example.php`).
* Added ability to choose the "Top 10" amount of users to display (eg. Top 20, Top 30 ..)
* Updated to use the newest WhatsApp API
* Added [API docs](https://gitlab.maikel.pro/maikeldus/WhatsSpy-Public/wikis/data-storage-and-API) for developers that want to use the tracker for other applications.

---
**Update 1.5.5** *(CRITICAL UPDATE, you will get problems if you don't update soon!)* : 

* Fixed connection closed problem causing no tracking ability.
* Changed Tracker to work with the newest WhatsApp version.
* Added filter option to timeline page to filter any tracker events.
* Added tracker option to ignore `connection closed` errors (to prevent spamming in the GUI).

---
**Update 1.5.4** : 

* Fixed notifications not being sent for privacy changes and errors.
* Updated literature list and welcome page message.
* Fixed tracker warnings about closed socket on close.
* Updated scripts used for experimental feautures.

---
**Update 1.5.3** : 

* Fixed rare race condition in the tracker.
* Added duration and combined status information in timeline (thanks to `marcel@mesa.nl`).
* Added experimental options for start/stop/update from the GUI.
* Notifications for verfied persons.

---
**Update 1.5.2** : 

* Implemented Desktop notifications in the Timeline page.
* Added option to diable password login (only use if your installation is safe behind a firewall/router!). You will no longer see the login screen.
* Changed security mechanism to invalidate sessions with old passwords.

---
**Update 1.5.1** : 

* Fixed several timezone issues for people living in half hour timezones.
* Created "Performance options" to speed up the loading in the GUI, allows customization of the status history length and the tracker session background.
* Changed no profile picture to a placeholder.
* Fixed race condition in the Timeline page causing no live feed.

---
**Update 1.5.0** : 

* Performance improvements in the tracker and the compare page. 
* Ability to share a group statistics profile or user profile if the installation is available from the internet. 
* Timeline sound is based on the filter now.
* Small GUI improvements (like on add new number reset information etc.).

:horse: **Warning:** This update contains a database upgrade. This update will happen automatically. If WhatsSpy Public does not work anymore, this is probably a database issue. Check [troubleshooting](troubleshooting).

:horse: **Warning 2:** This update requires a reconfiguration of Nginx and your `config.php`.

**1)** For the `config.php` (you can also copy this over from the `config.example.php`):

* Remove `$whatsspyWebProfilePath = '...';`, this is no longer required.
* Add `$whatsspyPublicAuth = '<choose-your-login-password>';` to set the password for you WhatsSpy Public installation.

**2)** For Nginx **(Raspberry Pi Image: skip this step)**:

* Execute `sudo nano /etc/nginx/sites-available/default` (or similar, for your Nginx configuration).
* Remove :
```
location /whatsspy/ {
        auth_basic "Restricted";
        auth_basic_user_file /etc/nginx/.htpasswdwhatsspy; 
    }
```

* Add the following:
```
location /whatsspy/images/profilepicture/ {
        deny all;
        return 404;
}
```
* Don't forget to reload the configuration by using `sudo service nginx reload`.

**3)** Raspberry Pi image users **ONLY**:

* Execute `sudo nano /etc/nginx/sites-available/default`
* Remove:

```
        auth_basic "Restricted";
        auth_basic_user_file /etc/nginx/.htpasswdwhatsspy; 
```
* Add before the last `}`:

```
location /images/profilepicture/ {
        deny all;
        return 404;
}
```
* Reload config: `sudo service nginx reload`.

If you open the interface make sure you do a full reload by using `Ctrl` + `F5` in the browser.

---
**Update 1.4.4** : Updated hover on status, improved compare page performance slightly, custom script agent retrieves better parameters. Added notifications for privacy changes.

---
**Update 1.4.3** : About page update, prevent accidental delete, DB migrations user independent, custom script agent created, compare page slightly updated.

---
**Update 1.4.2** : Proper database errors now visible in the tracker.

---
**Update 1.4.1** : Multiple groups for one contact and play sounds in timeline if desired.

:horse: **Warning:** This update contains a database upgrade. This update will happen automatically. If WhatsSpy Public does not work anymore, this is probably a database issue. Check [troubleshooting](troubleshooting).

---
**Update 1.4.0** : Added groups, better notification handling, performance improvements and somewhat better responsive design.

:horse: **Warning:** This update contains a database upgrade. This update will happen automatically. If WhatsSpy Public does not work anymore, this is probably a database issue. Check [troubleshooting](troubleshooting).

:horse: **Warning 2:** The configuration file has changed! Take a look at `config.example.php` and copy over any missing variables `$whatsspyNotificatons` (which replaces the old NMA and LN variables) and a new `debug` option in `$whatsappAuth`. Make sure you restart the tracker!

---

**Update 1.3.7** : Fixed timezone issue in the statistics page in case PostgreSQL is improperly configured. Also some minor UI fixes. Same rules apply as `1.3.0`.

---

**Update 1.3.6** : Fixed several UI bugs (statistics, timeline, new number) as well improved performance in the GUI. Same rules apply as `1.3.0`.

:horse: **Warning:** This update contains a database upgrade. This will happen automatically once you open the GUI or start the tracker. In case you get an Database upgrade error you can update the database yourself:
```
cd <location of whatsspy>/api/update/
psql -U whatsspy -d whatsspy -f database-1.3.6.sql
```
(you can use `-U postgres` in case whatsspy doesn't work, but please try `-U whatsspy` first)

---
**Update 1.3.4/5** : WhatsApi core update, serious tracker improvements in accuracy, this update is a must. Same rules apply as `1.3.0`.

* Make sure you PHP can read/write in `api/whatsapp/src/wadata/`

---
**Update 1.3.3** : Serious tracker timing improvements. Applied heuristics in end of online statuses. Keep-alive re-implemented with a logout/login. Same rules apply as `1.3.0`.

---
**Update 1.3.2** : Time fix where (1d) became (2d) and small change in tracking logs. Same rules apply as `1.3.0`.

---
**Update 1.3.1** : Removed notification for user going offline. Same rules apply as `1.3.0`.

---
**Update 1.3.0** : Improved/added following things:

* Tracker timing and error correction added to online/offline timing.
* New page called *statistics* which shows you information about the current tracked users.
* Tracked user view in page "overview" now contains analytics about WhatsApp usage (today, 7 days, 14 days, alltime).
* You can now be notified of specific users via WhatsApp about there online/offline status and profile pictures/status messages. You need to edit your config (read down here).

:horse: **Warning:** This update contains a database upgrade. This update will happen automatically. If WhatsSpy Public does not work anymore, this is probably a database issue. Check [troubleshooting](troubleshooting).

:horse: **Warning 2:** Also make sure if you enable notifications, you have `$whatsspyWhatsAppUserNotification` in your `config.php` (you can copy from `config.example.php`).

---
**Update 1.2.3** : Various fixes for UI (timeline filtering, image loading, lastseen status). same rules apply as  `1.0.6`.

---
**Update 1.2.2** : Improved query performance on PostgreSQL and fixed responsive UI for mobile. same rules apply as `1.0.6`.

---
**Update 1.2.1** : Improved tracker capacity and error handling. Fixed timeline issues where it misses statuses. same rules apply as `1.0.6`.

---
**Update 1.2.0** : Various responsive design fixes. Contacts now need to expand/collapse. Tracker timing is editable in `data.php`. same rules apply as `1.0.6`.

---
**Update 1.1.3** : Cascading delete button in the UI, small timeline fix for "ghost" users. same rules apply as `1.0.6`.

---
**Update 1.1.1/1.1.2** : Bug fix on file rights, PHP timezone and message throttling by WhatsApp. same rules apply as `1.0.6`.

---
**Update 1.1.0** : Implemented better error logging, implemented new page called "timeline" which shows realtime data. Fixed some timezone issues. same rules apply as `1.0.6`.

---
**Update 1.0.8/1.0.9** : Fix for people living in timezones which contain +/- 30 minutes. same rules apply as `1.0.6`.

---
**Update 1.0.7** : Fixed UTF8 support (works in FF only), added lastseen option and fixed some other small issues. same rules apply as `1.0.6`.

---
**Update 1.0.6** : Better error reporting and LiveNotifier added. You can undertake the same steps as 1.0.3. 

:horse: **Warning:** Please add `$whatsspyLNKey = '';` (in case you haven't) to your current config.

---
**Update 1.0.5** : Fixed broken pipe and added DB check at startup. Same rules apply as for 1.0.3

---
**Update 1.0.4** : Status message bug fixed. Same rules apply as for 1.0.3

---
**Update 1.0.1/1.0.2/1.0.3** : Small UI and Tracker fix. Replace the `whatsspy/` folder with the one of the repository .


## Need help?

In case you have any trouble upgrading your version, you can create an issue on this Gitlab server (first create account and login).