## WARNING: This role will be deprecated very soon

All of the functionality provided by this role and more is available in the [DebOps project](http://debops.org). If you are using some of my roles in conjunction with each other, you will find the move to DebOps most pleasurable.

This role will be **removed** from the **galaxy** and from **github** anywhere from 42 microseconds to 2-3 weeks after you read this message.

---


## What is ansible-monit? [![Build Status](https://secure.travis-ci.org/nickjj/ansible-monit.png)](http://travis-ci.org/nickjj/ansible-monit)

It is an [ansible](http://www.ansible.com/home) role to install monit and allow you to easily manage as many processes as you want.

### What problem does it solve and why is it useful?

I wasn't happy with any of the monit roles I came across. They were either missing features that I wanted or didn't work at all.

Here is a feature list of this role:

- Adjust the poll interval
- Optionally set a delay to when monit starts
- Optionally disable alerting by default
- Fully configure the mail details so you can get e-mail alerts
- Fully configure the http interface including optional ssl support
- Load arbitrary configs from the standard /etc/monit/conf.d path
- Create arbitrary monit tasks with the least amount of pain

## Role variables

Below is a list of default values along with a description of what they do.

```
---
# How often in seconds should monit check the process?
monit_interval: 60

# How many seconds delay should monit wait before monitoring on the first time?
monit_start_delay: 30 # 0 to disable

# Where should monit log to?
monit_logfile: syslog facility log_daemon

# Where should events be written in case the mail server is down?
monit_event_path: /var/lib/monit/events

# Should e-mail alerts be sent?
monit_alerts_on: true

# Skip certain types of events for the e-mail alerts.
monit_alert_skip_when: "action, instance, pid, ppid"

# The mail host name.
monit_mail_host: smtp.gmail.com

# The mail port.
monit_mail_port: 587

# The mail username.
monit_mail_username: you

# The mail password.
monit_mail_password: securepassword

# Which mail encryption should be used?
monit_mail_encryption: TLSV1 # SSLAUTO, SSLV2, SSLV3, TLSV1, TLSV11, TLSV12

# Who should receive the alerts?
monit_mail_alert_to: me@mydomain.com

# Who should the sender of the alert e-mail be?
monit_mail_default_from: monit@mydomain.com

# What should the subject be?
monit_mail_default_subject: "[Monit] $SERVICE $EVENT"

# What should the message say?
monit_mail_default_message: "Monit $ACTION $SERVICE at $DATE on $HOST: $DESCRIPTION."

# The host for the web interface.
monit_http_host: localhost

# Which port should it run on?
monit_http_port: 2812

# Which address(es) can access it?
monit_http_allow: ["localhost"]

# Should ssl be used?
monit_http_ssl: false

# If ssl is enabled, what local pem file should it copy?
monit_http_local_pemfile_path: ~/dev/testproject/secrets/monit.pem

# The lists to monitor your own processes. This is explained more below.
monit_process_list: []
monit_process_group_list: []
monit_process_host_list: []

# The amount in seconds to cache apt-update.
apt_cache_valid_time: 86400
```

### Adding processes to the list

In this example we will monitor 2 services that belong to an app server. It doesn't matter what they are doing, they just need to write out a pid file.

Before we do anything, let's look at how you would define a process.

```
monit_process_group_list:
    # Choose the template to use.
    # OPTIONAL: Defaults to base. No other templates are supported yet.
  - type: "base"

    # What is the process that you want to monitor?
    # REQUIRED.
    process: "foo"

    # What is the pid path for this process?
    # REQUIRED.
    pid_path: "/full/path/to/pid/file/foo.pid"

    # What is the command to start the program?
    # OPTIONAL: Defaults to using init.d with the process name.
    start: "/etc/init.d/$process start"

    # What is the command to stop the program?
    # OPTIONAL: Defaults to using init.d with the process name.
    stop: "/etc/init.d/$process stop"

    # How long in seconds should monit wait before it assumes the process timed out?
    # OPTIONAL: Defaults to 60 (use 0 to disable).
    timeout: 60

    # Additional monit configuration scripting.
    # OPTIONAL: Defaults to an empty string.
    script:

    # Should monit stop monitoring this process?
    # OPTIONAL: Defaults to false.
    delete: false
```

The above would write out the configuration to `/etc/monit/conf.d/foo.conf`.

#### Using a basic setup

Open your inventory directory and goto `group_vars/app.yml` and then add this:

```
monit_process_group_list:
  - process: "foo"
    pid_path: "/full/path/to/pid/file/foo.pid"
```

#### Add multiple processes and your own custom scripting 

Here is a brief example of adding multiple processes and your own custom monitoring script for one of them.

```
monit_process_group_list:
  - process: "foo"
    pid_path: "/full/path/to/pid/file/foo.pid"
  - process: "bar"
    pid_path: "/full/path/to/pid/file/bar.pid"
    script: |
      if cpu > 80% for 5 cycles then restart
      if 3 restarts within 5 cycles then timeout
```

## Example playbook without ssl

For the sake of this example let's assume you have a group called **app** and you have a typical `site.yml` file.

To use this role edit your `site.yml` file to look something like this:

```
---
- name: ensure app servers are configured
- hosts: app

  roles:
    - { role: nickjj.monit, tags: monit }
```

Let's say you want to edit a few defaults, you can do this by opening or creating `group_vars/app.yml` which is located relative to your `inventory` directory and then making it look something like this:

```
---
monit_interval: 120

# This would require you to enter myusername/mypassword in a browser while also
# being on localhost.
monit_http_allow: ["localhost", "myusername:mypassword"]
```

## Example playbook with ssl

First things first, make sure you read the section above because setting up the ssl version of this role is the same as the non-ssl version except it requires a few more defaults to be changed.

#### Do you need a pem file?

If you want your monit page to be accessible over ssl then you must generate a pem file.

You must run the following 2 commands:

`$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout monit.pem -out monit.pem`  
`$ openssl gendh 512 >> monit.pem`

Keep note of the path of where you created this file.

#### You are ready to change a few defaults

Overwrite at least the following default value(s) in `group_vars/all.yml`:

```
monit_http_ssl: true

monit_http_local_pemfile_path: PUT_YOUR_PEM_FILE_PATH_HERE
```

## Installation

`$ ansible-galaxy install nickjj.monit`

## Requirements

Tested on ubuntu 12.04 LTS but it should work on other versions that are similar.

## Troubleshooting common errors

#### The monit daemon is not running
In order to run it you must specify an interval for which monit to run at. You also need the web interface to be configured correctly.

#### I can't access the web page with curl with ssl enabled
You need to tell curl to run in insecure mode, `curl https://localhost:2812 --insecure`.

## Ansible galaxy

You can find it on the official [ansible galaxy](https://galaxy.ansible.com/list#/roles/949) if you want to rate it.

## License

MIT