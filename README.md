notify-twilio
===

**twilio-sms** is [Perl](http://www.perl.org) command line script which is able to send SMS messages
using [Twilio](http://www.twilio.com) SMS service.

Even though this script can be used for any kind of SMS messaging needs, it's
primary use is [Nagios](http://www.nagios.org)/[Icinga](http://www.icinga.org)
alerting.

Requirements
==

[Perl](http://www.perl.org) interpreter and the following modules:

  * [JSON](https://metacpan.org/module/JSON)
  * [MIME::Base64](https://metacpan.org/module/MIME::Base64)
  * [URI::Escape](https://metacpan.org/module/URI::Escape)
  * [LWP/libwww-perl](https://metacpan.org/module/LWP)

Setup
==

  * install this package and put the script in $PATH
  * Create default configuration file

```
twilio-sms --default-config > ~/.twilio-sms.conf
```

  * Edit configuration file to suit your twilio account settings

```
from          = "<your twilio telephone number>"
user_id       = "<your twilio user_id>"
auth_token    = "<your twilio auth token>"
timeout       = 5
verbose       = 0
split_big_msg = 0   # set to 1 if you want to send sms messages larger than 1600 chars
rewrite       = 0
```

**twilio-sms** searches for configuration files in the following filesystem locations:

```
/etc/twilio-sms.conf
/etc/twilio-sms/twilio-sms.conf
/usr/local/etc/twilio-sms.conf
/usr/local/etc/twilio-sms/twilio-sms.conf
/etc/icinga/twilio-sms.conf
/etc/nagios/twilio-sms.conf
/etc/nagios3/twilio-sms.conf
/usr/local/etc/icinga/twilio-sms.conf
/usr/local/etc/nagios/twilio-sms.conf
/usr/local/etc/nagios3/twilio-sms.conf
$HOME/config/twilio-sms/twilio-sms.conf
$HOME/config/twilio-sms.conf
$HOME/.config/twilio-sms/twilio-sms.conf
$HOME/.config/twilio-sms.conf
$HOME/.twilio-sms.conf
```

Script allways tries to read all configuration files listed above, variable values
are inherited from previously loaded configuration files if they are omitted from
configuration file currently being loaded.

  * Try to send a message :)

```
echo "Hello World" | twilio-sms -- +1987654321 +1987654322
```

Nagios/Icinga setup
===
  * add the following new commands to your Nagios/Icinga configuration:

```
define command {
    command_name    notify-service-by-twilio
    command_line    /usr/bin/printf "%b" "Service: $SERVICEDESC$\nHost: $HOSTALIAS$\nState: $SERVICESTATE$\n\n$SERVICEOUTPUT$" | /etc/icinga/plugins/twilio-sms -- $CONTACTPAGER$
}

define command {
    command_name    notify-host-by-twilio
    command_line    /usr/bin/printf "%b" "Host: $HOSTNAME$\nState: $HOSTSTATE$\n\n$HOSTOUTPUT$" | /etc/icinga/plugins/twilio-sms -- $CONTACTPAGER$
}
```

  * define new template and update contacts

```
# contact template
define contact {
    contact_name                    twilio-template
    name                            twilio-template
    alias                           Twilio contact template
    
    service_notification_period     24x7
    host_notification_period        24x7
    service_notification_options    w,u,c,r
    host_notification_options       d,r
    service_notification_commands   notify-service-by-twilio
    host_notification_commands      notify-host-by-twilio

    register                        0
}
    
# REAL contacts, based on twilio-template
define contact {
    use                             twilio-template
    contact_name                    joe_average
    pager                           +1234567890
}

# you probably want to use contact groups in your
# host/service definitions
define contactgroup {
    contactgroup_name       poor_bastards_on_call
    members                 joe_average
}
```

  * update service definitions to use new contactgroups

```
define service {
  name                            some_service
  # other boring stuff
  contact_groups                  poor_bastards_on_call
}
```

  * set up cron job for deferred message processing

```
# Try to re-send messages using Twilio that were not sent
# at the time of script invocation.
#
# There are many reasons for messages not being delivered
# on first attempt: internet connection unavailable, twilio
# api outage, etc...

*/15 * * * *	nagios /path/to/twilio-sms -P >/dev/null 2>&1
```

Usage
==

```
Usage: twilio-sms [OPTIONS] [-- <recipient> [<recipient> ...]]

This script is simple command line interface to
Twilio (http://www.twilio.com/) SMS service.

Input message length is not important since it is split into
multiple SMS messages if it's length exeedes limit of 1600
characters.

OPTIONS:
      --default-config     Prints out default configuration file
  -c  --config=FILE        Read configuration from specified
                           file. NOTE Script tries to read
                           the following files on startup:

                           /etc/twilio-sms.conf
                           /etc/twilio-sms/twilio-sms.conf
                           /usr/local/etc/twilio-sms.conf
                           /usr/local/etc/twilio-sms/twilio-sms.conf
                           /etc/icinga/twilio-sms.conf
                           /etc/nagios/twilio-sms.conf
                           /etc/nagios3/twilio-sms.conf
                           /usr/local/etc/icinga/twilio-sms.conf
                           /usr/local/etc/nagios/twilio-sms.conf
                           /usr/local/etc/nagios3/twilio-sms.conf
                           /home/username/config/twilio-sms/twilio-sms.conf
                           /home/username/config/twilio-sms.conf
                           /home/username/.config/twilio-sms/twilio-sms.conf
                           /home/username/.config/twilio-sms.conf
                           /home/username/.twilio-sms.conf

  -m  --msg=MSG            Message text
  -M  --msg-file=FILE      Read message text from specified file
                           NOTE: message is read from stdin if -m/-M
                           parameters are omitted.

      --rewrite            Rewrite variables in message with environmental
                           variable values (shell style)

  -R  --recipients=FILE    Read specified file containing recipient numbers
                           (one phone number per line, # comments supported)

  -f  --from=NUM           Sender number (Default: +19493834522)

  -P  --process            Process deferred messages
  -t  --timeout=SECS       HTTP request timeout in seconds (Default: 10)

  -r  --report             Report response statuses
  -v  --verbose            Verbose execution
  -D  --debug              Enable script debugging.
  -V  --version            Prints script version
  -h  --help               This help message

EXAMPLES:

  # send file contents over SMS
  cat /path/to/file | twilio-sms -- +1987654321 +145678923
  twilio-sms -M /path/to/file -- +1987654321 +145678923

  # send simple string
  twilio-sms -m 'Hello world from $USER'  -- +1987654321
  twilio-sms -m 'Hello world from $USER' --rewrite  -- +1987654321
  twilio-sms -m 'Hello world from \$USER' --rewrite  -- +1987654321

  echo 'Hello world from $USER' | twilio-sms  -- +1987654321
  echo 'Hello world from $USER' | twilio-sms --rewrite  -- +1987654321
  echo 'Hello world from \$USER' | twilio-sms --rewrite  -- +1987654321

  # send fortune of the day
  fortune | twilio-sms -- +1987654321

  # process deferred messages (usually from cron)
  twilio-sms -P
```

License
===
[The BSD 3-Clause License](http://opensource.org/licenses/BSD-3-Clause)
