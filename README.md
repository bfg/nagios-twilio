twilio-sms
===

**twilio-sms** is [Perl](http://www.perl.org) command line script which is able to send SMS messages
using [Twilio](http://www.twilio.com) SMS service.

Even though this script can be used for any kind of sms messaging need, it's
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

        twilio-sms --default-config > ~/.twilio-sms.conf

  * Edit configuration file to suit your twilio account settings

        from        = "<your twilio telephone number>"
        user_id     = "<your twilio user_id>"
        auth_token  = "<your twilio auth token>"
        timeout     = 5
        verbose     = 0
        split_big_msg = 1   # you probably want to send sms messages larger than 160 chars

  * Try to send a message :)

        echo "Hello World" | twillio-sms -- +1987654321 +1987654322

Nagios/Icinga setup
===

TODO

License
===
[The BSD 3-Clause License](http://opensource.org/licenses/BSD-3-Clause)

