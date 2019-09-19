# Fail2Ban-AbuseIPDB-APIv2
HOW TO Configure Fail2Ban to report to AbuseIPDB using APIv2 (APIv1 is deprecated)

Original Article: https://0ut3r.space/2019/04/06/abuseipdb/

Activate AbuseIPDB Reporting Action

You can invoke the AbuseIPDB action from some or all of the jails configured in jail.local. The action must be called with two parameters - your AbuseIPDB API key, and the abuse category (or categories) you would like to report the IP for. If these parameters are missing or invalid, your reports will fail.


```
# Actions to report to abuseipdb.com via API.
# See action.d/abuseipdb.conf
action_abuseipdb_fraud     = abuseipdb[abuseipdb_category="3"]
action_abuseipdb_ddos      = abuseipdb[abuseipdb_category="4"]
action_abuseipdb_proxy     = abuseipdb[abuseipdb_category="9"]
action_abuseipdb_forumspam = abuseipdb[abuseipdb_category="10"]
action_abuseipdb_emailspam = abuseipdb[abuseipdb_category="11"]
action_abuseipdb_blogspam  = abuseipdb[abuseipdb_category="12"]
action_abuseipdb_portscan  = abuseipdb[abuseipdb_category="14"]
action_abuseipdb_hack      = abuseipdb[abuseipdb_category="15"]
action_abuseipdb_sqlinject = abuseipdb[abuseipdb_category="16"]
action_abuseipdb_spoofing  = abuseipdb[abuseipdb_category="17"]
action_abuseipdb_sshbrute  = abuseipdb[abuseipdb_category="18"]
action_abuseipdb_badwebbot = abuseipdb[abuseipdb_category="19"]
action_abuseipdb_exploited = abuseipdb[abuseipdb_category="20"]
action_abuseipdb_webappattack  = abuseipdb[abuseipdb_category="21"]
action_abuseipdb_sshabuse  = abuseipdb[abuseipdb_category="22"]
action_abuseipdb_iot  = abuseipdb[abuseipdb_category="23"]

```

This line of code must be added to each jail for which you want to activate AbuseIPDB reporting. Here’s an example of how you would configure the AbuseIPDB report action to run, in addition to your default ban actions, when the ssh brute force jail is triggered:

```
[sshd]

# To use more aggressive sshd modes set filter parameter "mode" in jail.local:
# normal (default), ddos, extra or aggressive (combines all).
# See "tests/files/logs/sshd" or "filter.d/sshd.conf" for usage example and details.
mode = aggressive
port   = ssh
filter = sshd
logpath = %(sshd_log)s
backend = %(sshd_backend)s
maxretry = 3
findtime = 604800 ; (1 week)
enabled = true

```

However, the AbuseIPDB action can also be added to the list of Fail2Ban actions in the global [DEFAULT] as shown below, which will cause it to run on all jails without an action specified:

```
# Choose default action.  To change, just override value of 'action' with the
# interpolation to the chosen action shortcut (e.g.  action_mw, action_mwl, etc) in jail.local
# globally (section [DEFAULT]) or per specific section
action = %(action_)s
         %(action_abuseipdb_sshbrute)s
         %(action_abuseipdb_sshabuse)s
```

But it’s recommended to add the AbuseIPDB action individually to your jails so you can customize the AbuseIPDB report categories.

As we configure APIv2 (because APIv1 is deprecated) one line need to be changed in abuseipdb.conf

```
sudo nano /etc/fail2ban/action.d/abuseipdb.conf
```

Find line starting from actionban and replace it with

```
actionban = curl --fail 'https://api.abuseipdb.com/api/v2/report' -H 'Accept: application/json' -H 'Key: <abuseipdb_apikey>' --data-urlencode "comment=<matches>" --data-urlencode 'ip=<ip>' --data 'categories=<abuseipdb_category>'

```

Now save configuration and restart Fail2Ban service.

```
sudo service fail2ban restart
```
or

```
sudo fail2ban-client reload
```
