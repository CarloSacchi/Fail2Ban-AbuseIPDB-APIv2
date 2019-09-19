# Fail2Ban-AbuseIPDB-APIv2
Configure Fail2Ban to repurt to AbuseIPDB using APIv2 (APIv1 is deprecated)

Original Article: https://0ut3r.space/2019/04/06/abuseipdb/

Activate AbuseIPDB Reporting Action

You can invoke the AbuseIPDB action from some or all of the jails configured in jail.local. The action must be called with two parameters - your AbuseIPDB API key, and the abuse category (or categories) you would like to report the IP for. If these parameters are missing or invalid, your reports will fail.

```
%(action_abuseipdb)s[abuseipdb_apikey="my-api-key", abuseipdb_category="18,22"]
```

This line of code must be added to each jail for which you want to activate AbuseIPDB reporting. Here’s an example of how you would configure the AbuseIPDB report action to run, in addition to your default ban actions, when the ssh brute force jail is triggered:

```
[sshd]

# To use more aggressive sshd modes set filter parameter "mode" in jail.local:
# normal (default), ddos, extra or aggressive (combines all).
# See "tests/files/logs/sshd" or "filter.d/sshd.conf" for usage example and details.
#mode   = normal
enabled = true
port    = 22
logpath = %(sshd_log)s
backend = %(sshd_backend)s

# Ban IP and report to AbuseIPDB for SSH Brute-Forcing
action = %(action_)s
         %(action_abuseipdb)s[abuseipdb_apikey="my-api-key", abuseipdb_category="18,22"]
```

However, the AbuseIPDB action can also be added to the list of Fail2Ban actions in the global [DEFAULT] as shown below, which will cause it to run on all jails without an action specified:

```
# Choose default action.  To change, just override value of 'action' with the
# interpolation to the chosen action shortcut (e.g.  action_mw, action_mwl, etc) in jail.local
# globally (section [DEFAULT]) or per specific section
action = %(action_)s
         %(action_abuseipdb)s[abuseipdb_apikey="my-api-key", abuseipdb_category="18"]
```

But it’s recommended to add the AbuseIPDB action individually to your jails so you can customize the AbuseIPDB report categories.

As we configure APIv2 (because APIv1 is deprecated) one line need to be changed in abuseipdb.conf

```
sudo nano /etc/fail2ban/action.d/abuseipdb.conf
```

Find line starting from actionban and replace it with

```
actionban = curl --fail 'https://api.abuseipdb.com/api/v2/report' -H 'Accept: application/json' -H 'Key: <abuseipdb_apikey>' --data-urlencode "comment=<matches>" --data-urlencode 'ip=<ip>' --data 'categories=<abuseipdversionb_category>'
```

Now save configuration and restart Fail2Ban service.

```
sudo service fail2ban restart
```
or

```
sudo fail2ban-client reload
```
