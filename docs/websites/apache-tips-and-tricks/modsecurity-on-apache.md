---
author:
  name: Chris Ciufo
  email: docs@linode.com
description: 'mod_security'
keywords: 'apache, mod_security'
license: '[CC BY-ND 4.0](https://creativecommons.org/licenses/by-nd/4.0)'
alias: ['web-servers/apache/mod-security/']
modified: Friday, February 14th, 2014
modified_by:
  name: Linode
published: 'Thursday, November 10th, 2011'
title: 'mod_security on Apache'
external_resources:
 - '[ModSecurity Home Page](http://www.modsecurity.org)'
 - '[OWASP Home Page](https://www.owasp.org/index.php/Main_Page)'
 - '[OWASP ModSecurity Core Rule Set Wiki](https://www.owasp.org/index.php/Category:OWASP_ModSecurity_Core_Rule_Set_Project#tab=Installation)'
---

ModSecurity is a web application firewall for the Apache web server. In addition to providing logging capabilities, ModSecurity can monitor the HTTP traffic in real time in order to detect attacks. ModSecurity also operates as a web intrusion detection tool, allowing you to react to suspicious events that take place at your web systems.


## Installing ModSecurity On Ubuntu (16.10)

Before you install ModSecurity, you'll want to have a LAMP stack set up on your Linode. For instructions, see the [LAMP Guides](/docs/websites/lamp/).

To install ModSecurity on a Linode running Ubuntu 16.10, enter the following command:
    >   sudo apt-get install libxml2 libxml2-dev libxml2-utils libaprutil1 libaprutil1-dev libapache2-mod-security2

ModSecurity is now installed on your Linode.


## Configuring ModSecurity

Download the recommended ModSecurity Configuration
-   <https://github.com/SpiderLabs/ModSecurity/blob/master/modsecurity.conf-recommended>

Rename and move the recommended configuration file keeping the Ubuntu default version for reference if you'd like 
    >   sudo mv /etc/modsecurity/modsecurity.conf /etc/modsecurity/modsecurity.conf.orig
    >   sudo mv modsecurity.conf-recommended /etc/modsecurity/modsecurity.conf
    
    
## Installing The OWASP ModSecurity Core Rule Set

For a base configuration, we are going to use the OWASP core rule set. 
-   <https://github.com/SpiderLabs/owasp-modsecurity-crs/blob/v3.0/master/INSTALL>

    >   sudo apt-get install git
    >   sudo mkdir /etc/apache2/modsecurity.d
    >   cd /etc/apache2/modsecurity.d
    >   sudo git clone https://github.com/SpiderLabs/owasp-modsecurity-crs
    >   cd owasp-modsecurity-crs
    >   sudo mv crs-setup.conf.example crs-setup.conf
    >   sudo mv rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf.example rules/REQUEST-900-EXCLUSION-RULES-BEFORE-CRS.conf
    >   sudo mv rules/RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf.example rules/RESPONSE-999-EXCLUSION-RULES-AFTER-CRS.conf

Now enable the rules in the apache security2 module configuration file
    >   sudo nano /etc/apache2/mods-available/security2.conf
    
Add these two lines inside the <IfModule security2_module> blocks
    >   IncludeOptional /etc/apache2/modsecurity.d/owasp-modsecurity-crs/crs-setup.conf
    >   IncludeOptional /etc/apache2/modsecurity.d/owasp-modsecurity-crs/rules/*.conf

Restart apache
    >   sudo service apache2 restart
    
    
## Test That The Module Is Working

Visit http://localhost/?param="><script>alert(1);</script> or http://yourdomain/?param="><script>alert(1);</script> 

If everything is working you should have a notice in your apache error log. If you have changed the module's behavior to On you should get a 403 Forbidden response and an error log entry.


## Changing The Detection Behavior

Once you've verified everything is working correctly you may want to change the module's behavior from DetectionOnly to On.
    >   sudo nano /etc/modsecurity/modsecurity.conf
Change "SecRuleEngine DetectionOnly" to "SecRuleEngine On" then restart apache
    >   sudo service apache2 restart
    
## Keeping Up To Date

To keep the rules up to date you may use cron and a tool included with the rules you downloaded earlier
0 2 * * *  util/upgrade.py --geoip --cron


You have successfully configured ModSecurity.


*There is additional information included in the core rules set linked above as well as the well commented configuration files themselves. Read over everything as suggested in those documents to ensure the module is configured to work with you particular environment.

Parts of this guide were taken directly from the core rule set installation instructions linked above and the previous version of this file. I claim no ownership to any of the content provided here. I've just condensed all of the information into a single accurate linear guide to get started with ModSecurity using the latest version of Ubuntu, ModSecurity, and the OWASP Core Rule Set.
