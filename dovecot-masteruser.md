In order to use IMAPSync to transfer emails from server1 to server2, I need a masteruser that can read every user's emails.
# Environment
```
Linux tir 4.4.0-154-generic #181-Ubuntu SMP Tue Jun 25 05:29:03 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
Ubuntu 16.04.6 LTS
Dovecot for IMAP/POP3 email server
```

# Steps
* Create /etc/dovecot/dovecot-acl
```
* user=masteruser lr
```
* Create /etc/dovecot/master-users
**Note**: -d or -s option is needed; otherwise, authentication fails.
```
htpasswd -d -c master-users masteruser
```
* Update /etc/dovecot/conf.d/auth-master.conf.ext
```
passdb {
  driver = passwd-file
  master = yes
  args = /etc/dovecot/master-users

  # Unless you're using PAM, you probably still want the destination user to
  # be looked up from passdb that it really exists. pass=yes does that.
  #pass = yes
  result_success = continue
}
passdb {
  driver = shadow
}
userdb {
  driver = passwd
}
```
  * Added the **passdb** and **userdb** sections
  * Commented out *pass = yes*
  * Addred *result_success = continue*

# Update /etc/dovecot/conf.d/10-auth.conf
```
!include auth-master.conf.ext
auth_debug = yes
auth_debug_passwords = yes
auth_verbose = yes
```
  * Uncomment the _auth-master.conf.ext_ line
  * Add the _auth_debug_* lines (optional, for trouble shooting)
  
# Add in /etc/dovecot/conf.d/90-acl.conf
```
plugin {
  acl = vfile:/etc/dovecot/dovecot-acl:cache_secs=300
}
```
# Syntax check
```
doveconf -n
```
# Restart dovecot
```
service dovecot restart
```
# Test
```
telnet localhost 143
* OK Dovecot ready.
1 login loginuser*masteruser masterpass
1 OK Logged in.
```
# Debug
When I used the htpasswd to create master-users without -s or -d option, the authentication failed; logged in /var/log/dovecot_info.log.
```
May 26 23:09:54 auth: Debug: passwd-file(loginuser,::1,masteruser,<S....>): CRYPT(masteruser_pwd) != 'encrypted_string_in_master-users'
May 26 23:09:56 auth: Debug: client passdb out: FAIL	1	user=loginuser
```

# References
* https://doc.dovecot.org/configuration_manual/authentication/master_users/
