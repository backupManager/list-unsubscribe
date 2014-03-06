# List-Unsubscribe

Automatically unsubscribes from messages threads when moved to a magic "Unsubscribe" IMAP folder.

Messages must support [`List-Unsubscribe` mail header standard](http://www.list-unsubscribe.com/).

## Setup

Requires running your own Heroku cron daemon.

Clone the app

```
$ git clone https://github.com/josh/list-unsubscribe.git
$ cd list-unsubscribe/
```

Create a Heroku app

```
$ heroku create
$ git push heroku master
```

Configure your IMAP and STMP settings. For GMail, use an [application-specific password](https://support.google.com/mail/answer/1173270?hl=en).

```
$ heroku config:set IMAP_ADDRESS=imap.gmail.com
$ heroku config:set SMTP_ADDRESS=smtp.gmail.com
$ heroku config:set IMAP_USERNAME=example@gmail.com
$ heroku config:set IMAP_PASSWORD=secret
```

Configure a 10 minute cron schedule.

```
$ heroku addons:add scheduler:standard
$ heroku addons:open scheduler
```

![](https://f.cloud.github.com/assets/137/326171/7c3a4e48-9b24-11e2-8313-fc659231efc1.png)
