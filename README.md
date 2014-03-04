# Lumen

[![Build Status](https://travis-ci.org/wordsandwriting/lumen.png?branch=master)](https://travis-ci.org/wordsandwriting/lumen)

Lumen started life as a group discussion platform akin to [Google Groups](http://groups.google.com), [GroupServer](http://groupserver.org/), 
[Mailman](http://www.list.org/) or [Sympa](http://www.sympa.org/). Since then, it's gained some powerful extras. An outline of its features:

* Open-source (under [Creative Commons Attribution-NonCommercial-ShareAlike 3.0 Unported](http://creativecommons.org/licenses/by-nc-sa/3.0/))
* Hosted using Heroku (for the web interface, free in initial case), Amazon S3 (for file attachments, typically free or a few pennies per month) and Virtualmin (for the mail sever, requires a low-end VPS at a cost of a few pounds per month)
* Designed for custom domains (group email addresses of the form yourgroup@yourdomain.org)
* Sends and receives mail via regular SMTP and IMAP accounts on Virtualmin
* Dual web/email access
* Extensible member profiles
* Flexible digest engine
* Events calendar
* Daily summaries of news stories tweeted by group members via news.me integration
* Geographically-aware lists feature for maintaining lists of venues, contacts etc

Lumen is written in Ruby using the [Padrino](http://padrinorb.com/) framework. It was originally created for the [New Economy Organisers Network](http://neweconomyorganisersnetwork.org/) (hosted by the [New Economics Foundation](http://neweconomics.org/)) who kindly agreed to open source the project and continue to sponsor its development.

[<img src="http://wordsandwriting.github.io/lumen/images/top.jpg">](http://wordsandwriting.github.io/lumen/images/top.jpg)

See below for more images.

## How the mailing lists work, in brief

1. Your VPS (running Postfix/Virtualmin) receives a mail to yourgroup@yourdomain.org
2. The mail triggers a simple notification script on the VPS that in turn alerts your Heroku app to the fact there's a new message for the group
3. Your Heroku app connects to the VPS via IMAP to fetch the new mail
4. Your Heroku app distributes the message to group members via SMTP

## Getting up and running

### 1. Register a domain

We'll add the DNS records shortly.

###  2. Purchase a VPS and set up Virtualmin

Purchase a VPS (check out [http://lowendbox.com/](http://lowendbox.com/), 512mb RAM should do). Use `mail.yourdomain.org` as the hostname and if in doubt choose CentOS 6 64-bit as your operating system. SSH into the VPS and install Virtualmin with:

```
wget http://software.virtualmin.com/gpl/scripts/install.sh
chmod +x install.sh
./install.sh
```

When the install is finished, visit https://{your VPS IP}:10000. Step through the post-install wizard, turning on the email domain lookup server, turning off MySQL, and choosing to only store hashed passwords. Go to System Settings > Features and Plugins leave only 'Mail for domain', 'Log file rotation' and 'Webmin login' checked. Create a virtual server for yourdomain.org with an administration password set to the same as the Virtualmin user password. Go to 'Edit Users' an create a user no-reply@yourdomain.org with the same password again.

### 3. Push code to Heroku

```
git clone https://github.com/wordsandwriting/lumen.git
cd lumen
heroku create yourappname
git push heroku master
heroku domains:add www.yourdomain.org
heroku addons:add mongohq
heroku addons:add memcachier
heroku addons:add papertrail
heroku addons:add scheduler
heroku config:set SESSION_SECRET=`rake secret`
heroku config:set HEROKU_APP_NAME=yourappname
heroku config:set HEROKU_API_KEY=yourapikey
heroku config:set DOMAIN=www.yourdomain.org
heroku config:set MAIL_DOMAIN=yourdomain.org
```

(You can find your Heroku API key at [http://dashboard.heroku.com/account](http://dashboard.heroku.com/account).)

### 4. Set DNS

* Point www. to Heroku `www.yourdomain.org CNAME yourappname.herokuapp.com`
* Naked domain redirect via [wwwizer.com](http://wwwizer.com) `yourdomain.org A 174.129.25.170` 
* For mail delivery `yourdomain.org MX mail.yourdomain.org` and `mail.yourdomain.org A {your VPS IP}`
* SPF `yourdomain.org TXT "v=spf1 a mx a:yourdomain.org ip4:{your VPS IP} ?all"`
* DKIM: Visit Email Messages > DomainKeys Identified Mail in Virtualmin. Set 'Reject incoming email with invalid DKIM signature?' to 'No' and enter yourdomain.org to 'Additional domains to sign for'. Then add the record under 'DNS records for additional domains'.

### 5. Configuration

Visit www.yourdomain.org and change the admin name, email address and password. Click 'Lumen configuration' and complete the configuration.

### 6. Rake tasks

Run `heroku run rake mi:create_indexes` to create the database indexes. Open the Scheduler add-on with `heroku addons:open scheduler` and add the following tasks:
```
rake news:update 7am
rake digests:daily 7.30am
rake digests:weekly 11pm (only runs on Sunday)
rake cleanup 4am
```

...and you're done!

## Switching mail servers

If you switch your mail server, you'll need to re-setup the group mail accounts on the new server. Fire up a console (`heroku run bash; padrino c`) and run:
```
Group.all.each { |g| g.setup_mail_accounts_and_forwarder }
ConversationPost.all.each { |c| c.update_attribute(:mid, nil) }
```

## Gallery

### Homepage (amalgamates newsfeeds from all groups)
[<img src="http://wordsandwriting.github.io/lumen/images/home.jpg">](http://wordsandwriting.github.io/lumen/images/home.jpg)

### Home calendar (amalgamates events from all groups)
[<img src="http://wordsandwriting.github.io/lumen/images/calendar.jpg">](http://wordsandwriting.github.io/lumen/images/calendar.jpg)

### A group
[<img src="http://wordsandwriting.github.io/lumen/images/group.jpg">](http://wordsandwriting.github.io/lumen/images/group.jpg)

### A group digest
[<img src="http://wordsandwriting.github.io/lumen/images/digest.jpg">](http://wordsandwriting.github.io/lumen/images/digest.jpg)

### A group's map
[<img src="http://wordsandwriting.github.io/lumen/images/map.jpg">](http://wordsandwriting.github.io/lumen/images/map.jpg)

### A list
[<img src="http://wordsandwriting.github.io/lumen/images/list.jpg">](http://wordsandwriting.github.io/lumen/images/list.jpg)

### A profile 
[<img src="http://wordsandwriting.github.io/lumen/images/profile.jpg">](http://wordsandwriting.github.io/lumen/images/profile.jpg)

### Email accounts on Virtualmin corresponding to groups 
[<img src="http://wordsandwriting.github.io/lumen/images/virtualmin.jpg">](http://wordsandwriting.github.io/lumen/images/virtualmin.jpg)

