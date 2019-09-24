---
title: Alert Email
description: Learn how to set up alert email for AMC
---

This feature is only available for AMC version 4.0 and above.

AMC can send out email to all the registered email addresses if any of the monitored
Aerospike clusters generates an alert. The list of emails is configured at AMC start
up and cannot be changed dynamically.

The email configuration is a context in the configuration file.

```
[mailer]
template_path = "/home/amc/mailer/templates"
host          = "smtp.outlook.com"
port          = 587
user          = "user"
password      = "user123"
send_to       = ["monitorone@gmail.com", "monitortwo@yahoo.com"]
```


*template_path* - the directory containing the templates for the mails
```
template_path = "/home/amc/mailer/templates"
```

*host, port* - the host name and port of the smtp server
```
host = "smtp.outlook.com"
port = 587
```

*user, password* - the user name and password at the smtp server
```
user     = "user"
password = "user123"
```

*send_to* - the list of emails to send out alerts to
```
send_to = ["monitorone@gmail.com", "monitortwo@yahoo.com"]
```

{{#note}}The `user` field in the `[mailer]` configuration is used as the from address when sending notification alerts via e-mail. This field must populated in order for e-mail notifications to be successfully sent.{{/note}}