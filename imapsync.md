**Goal**: Sync mails from server1 to server2
* sync as the user
```
imapsync \
     --host1 test1.lamiral.info --user1 test1 --password1 secret1 \
     --host2 test2.lamiral.info --user2 test2 --password2 secret2
```

* sync on behalf of the user as masteruser
```
imapsync \
     --host1 test1.lamiral.info --user1 test1*masterusername --password1 secret1 \
     --host2 test2.lamiral.info --user2 test2 --password2 secret2
```

* sync for one folder
```
imapsync \
     --folder Archive.2006.Sent \
     --host1 test1.lamiral.info --user1 test1 --password1 secret1 \
     --host2 test2.lamiral.info --user2 test2 --password2 secret2
```

* fix longlines in message
```
imapsync \
     --folder Archive.2006.Sent --pipemess "reformime -r7" \
     --host1 test1.lamiral.info --user1 test1 --password1 secret1 \
     --host2 test2.lamiral.info --user2 test2 --password2 secret2
```

* fix "Request is trottled" - caused by limits on mailboxes placed by Office365
```
imapsync \
     --folder Archive.2006.Sent --pipemess "reformime -r7" \
     --maxbytespersecond 100_000 --maxmessagespersecond 2 --exitwhenover 1_000_000_000 \
     --host1 test1.lamiral.info --user1 test1 --password1 secret1 \
     --host2 test2.lamiral.info --user2 test2 --password2 secret2
```

# References
* [APPEND error - fix long lines](https://imapsync.lamiral.info/FAQ.d/FAQ.APPEND_errors.txt)
```
Err 1/11: - msg Archive.2006.Sent/810 {27848} could not append ( Subject:...], 
Date:..., Size:[27848], Flags:[\Seen] ) 
to folder Archive/2006/Sent: socket closed while reading data from server (4x)
```
* [Trottled error](https://github.com/imapsync/imapsync/issues/198)
```
Err 1/4: - msg Peter/118 {4792} could not append ( Subject:..., 
Date:..., Size:[4720], Flags:[\Seen] )
to folder Peter: Error sending '3074 APPEND Peter (\Seen) "06-Jul-2005 09:11:03 -0400" {4813}': 3074 NO 
Request is throttled. Suggested Backoff Time: 299839 milliseconds
```
* [Set up master user for Dovecot](https://github.com/xinxiang/notes/blob/master/linux/dovecot_masteruser.md)
* [Set up Stunnel](https://github.com/xinxiang/notes/blob/master/stunnel.md)
