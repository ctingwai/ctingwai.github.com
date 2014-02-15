---
layout: post
title: "Postfix mail relays"
date: 2013-10-09 14:31
comments: true
categories:
- Linux
- Ubuntu
- Email
- Postfix
---
Recently I have to setup a mail server to notify me of system failure from Nagios. So, instead of setting the mail server from scratch, I decided to use my company's email as a relay for notification.

<!-- more -->

Postfix are installed on most distros as the default MTA (Mail Transfer Agent). In cases where it is not installed, you can install by searching for postfix in your package manager, e.g. `sudo apt-cache search postfix`. Since, I am installing on a Ubuntu 12.04 machine, the instructions here lean more to Ubuntu systems, but it should work with other distros too. Configuration of mail server used to relay email from my Ubuntu machine: \ 
    Incoming server: smtp.domain.com (IMAP = Port 143, POP = Port 110)
    Outgoing server: smtp.domain.com (SMTP = Port 587)

1\. Add the following lines to `/etc/postfix/main.cnf`: \ 
    # Settings for relay
    relayhost = smtp.domain.com:587
    smtp_sasl_auth_enable = yes
    smtp_sasl_password_maps = hash:/etc/postfix/sasl/sasl_passwd
    smtp_sasl_security_options = noanonymous, noplaintext
    smtp_sasl_mechanism_filter = login, plain
    smtp_generic_maps = hash:/etc/postfix/generic 

2\. Add the following lines to /etc/postfix/sasl/sasl\_passwd: \ 
    [smtp.domain.com]:587 username:password

3\. Generating sasl\_passwd.db file by running the following command: \ 
    $ sudo postfix /etc/sasl/sasl\_passwd

4\. Make sure that only root can read both file:
    $ sudo chmod 600 /etc/sasl/sasl\_passwd*

5\. Restart postfix:
    $ sudo /etc/init.d/postfix restart

6\. Send a test mail:
    echo "Testing" | mail -s "Test" email@address.com

You should receive your email in the address you sent above. If you are not getting an email, check the system mail, email failed to sent will get sent back to your own machine along with some error messages, you can check the system mail by using `cat /var/mail/root`. Note however, not all mail servers allow mail relays.
