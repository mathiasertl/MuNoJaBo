**MuNoJaBo** stands for "munin jabber notification bot". MuNoJaBo is a script
that is designed to be called by munin when the values monitored by munin
are at "warning" or "critical". As of version 2.0 this script also uses a simple
mysql-database to store notificatons that where already send and does not send
notifications again for six hours.

Installation
============

This script is written for Python3 and requires SleekXMPP and dnspython3. If
you want to run it in a virtualenv, just initialize the virtualenv and install
the required dependencies:

```
virtualenv -p /usr/bin/python3 .
source bin/activate
pip install -r requirements.txt
```

Under Debian/Ubuntu, you can also install the required dependencies:

```
apt-get install python3-sleekxmpp python3-dnspython
```

DATABASE
========

You need a mysql-database for the cache. This cache is *really* necessary
because munin will call this script every five minutes and you don't want to get
messages every 5 minutes for a whole night. To set up a database, you have to 
have mysql running. The required layout is already in db-layout.sql, simply
execute:

```
mysql -uroot -p < db-layout.sql
```

You still need to create a mysql-user, this will set you up on a mysql-prompt:

```
GRANT DELETE, SELECT, INSERT on munojabo.* TO 'munojabo'@'localhost' IDENTIFIED BY '<pass>';
```

CONFIGURATION
=============

This script requires a config-file located at /etc/munojabo.conf. An example
configuration file can be found in this directory. Just fill in the appropriate
details.

To configure munin to use this script, add a contact line like this to your
munin.conf:

contact.you.command munojabo-save.py --jid=<your_jid> --host=${var:host} \
	"--graph=${var:graph_title}" \
	"--warnings=${loop<;>:wfields \
		${var:label}=${var:value},${var:wrange},${var:crange}}" \
	"--critical=${loop<;>:cfields \
		${var:label}=${var:value},${var:wrange},${var:crange}}" \
	"--unknown=${loop<;>:ufields \
		${var:label}=${var:value},${var:wrange},${var:crange}}"

Replace <your_jid> with the jid that should receive the notifications. You can 
optionally include a resource, without a given resource, the resource with the
highest priority will receive the message.

All in all, a call to the script might look something like this:
	munojabo.py mati@fsinf.at pluto.htu.tuwien.ac.at Temperature --warnings= --critical=foo=5,,7: --unknown=

Note that the contact-line given above is not enough munin-configuration, since
munin has to be configured to send warnings at all. This is outside the scope of
this file, so please visit this page:
	http://munin.projects.linpro.no/wiki/HowToContact
