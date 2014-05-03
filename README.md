# Install Logstash & Kibana

## Overview

This document explains how to install [logstash](http://logstash.net/) and [kibana](http://rashidkpc.github.com/Kibana/) on a local machine to evaluate the tools and to get started easily.

## Please note

The setup is meant to run on localhost and it's not optimized for
production purposes - although the tools can be configured to work in
distributed environments but I leave that as homework for you ;)

This page might give you some hints: http://logstash.net/docs/1.1.1/tutorials/getting-started-centralized


# Installation

This has been tested on Ubuntu 11.10 but should work on other flavours
as well. Make sure you have Java installed.

1) Install Java

    sudo apt-get install openjdk-6-jdk

2) Download Logstash

    mkdir -p /opt/logstash/
    cd !$
    wget http://semicomplete.com/files/logstash/logstash-1.1.1-monolithic.jar

3) Create file "agent.conf"

    input {
      file {
        type => "syslog"
        path => [ "/var/log/*.log", "/var/log/apache2/*.log" ]
      }
    }

    filter {
      grok {
        type => "syslog"
        # See the following URL for a complete list of named patterns
        # logstash/grok ships with by default:
        # https://github.com/logstash/logstash/tree/master/patterns
        #
        # The grok filter will use the below pattern and on successful match use
        # any captured values as new fields in the event.
        pattern => "%{COMBINEDAPACHELOG}"
      }

      date {
        type => "syslog"
        # Try to pull the timestamp from the 'timestamp' field (parsed above with
        # grok). The apache time format looks like: "18/Aug/2011:05:44:34 -0700"
        timestamp => "dd/MMM/yyyy:HH:mm:ss Z"
      }
    }

    output {
      stdout { }
      elasticsearch { embedded => true }
    }

See http://logstash.net/docs/1.1.1/tutorials/getting-started-simple for
more details on the configuration.

4) Start logstash

    java -jar logstash-1.1.1-monolithic.jar agent -f agent.conf -- web

5) Install kibana: http://rashidkpc.github.com/Kibana/intro.html
Set elasticsearch host in config.php like this:

    'elasticsearch_server' => "localhost:9200", 

6) Open kibana web interface

## Configure TYPO3

If you want to see the TYPO3 logs in kibana, add these lines to
`localconf.php`:

    $TYPO3_CONF_VARS['SYS']['systemLog'] = 'syslog,USER';
    $TYPO3_CONF_VARS['SYS']['systemLogLevel'] = '0';
    $TYPO3_CONF_VARS['SYS']['belogErrorReporting'] = 0;

Now try to log into the backend with wrong credentials for example.

### Example

1. Log in with the username "faker" and a random password
2. After the log in failed, switch to Kibana Webinterface
3. Search for "faker"
4. Now this error message should appear in kibana:

    Aug 22 04:32:27 machine http://example.com/:  - Core: Login-attempt from 192.168.156.1 (), username 'faker' not found!
