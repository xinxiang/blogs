# Task
Sync emails from server1 to server 2 with IMAP sync
```
/usr/bin/imapsync --host1 test1.lamiral.info  --user1 test1 --password1  "secret1" --host2 test2.lamiral.info  --user2 test2 --password2  "secret2"
```
# Problem - server1 does not support SSL
```
...
Host1 IP address: ...
Host1 banner: * OK [CAPABILITY IMAP4rev1 LITERAL+ SASL-IR LOGIN-REFERRALS ID ENABLE IDLE LOGINDISABLED] Dovecot ready.
Host1 capability before authentication: IMAP4rev1 LITERAL+ SASL-IR LOGIN-REFERRALS ID ENABLE IDLE LOGINDISABLED
Host1 failure: Error login on [...] with user [...] auth [LOGIN]: 2 NO [PRIVACYREQUIRED] Plaintext authentication disallowed on non-secure (SSL/TLS) connections.
Exiting with return value 16 (EXIT_AUTHENTICATION_FAILURE)
...
```
# Solution - Run stunnel on server1
* sudo apt install stunnel4
* create stunnel.pem
```
cd /tmp/stunnel
openssl genrsa -out key.pem 2048
openssl req -new -x509 -key key.pem -out cert.pem -days 1095
cat key.pem cert.pem >> /etc/stunnel/stunnel.pem
rm -rf /tmp/stunnel
```
* create /etc/stunnel/tunnel.conf 
```
[imaps]
accept  = 993
connect = 143
cert = /etc/stunnel/stunnel.pem
```

* update /etc/default/stunnel4 [optional]
```
ENABLED=1
```
* chmod go-r /etc/stunnel/stunnel.pem [optional]

* service stunnel4 restart

# Allow access from server2 to server1
* server1:/etc/hosts.allow
```
imap: server2_ip 
imaps: server2_ip 
```
* ufw
```
ufw allow from server2 to server1 port 993
ufw allow from server2 to server1 port 143
```

# References
* https://imapsync.lamiral.info/
* https://www.digitalocean.com/community/tutorials/how-to-set-up-an-ssl-tunnel-using-stunnel-on-ubuntu
* https://www.octaldream.com/~scottm/talks/ssl/stunnel.html
