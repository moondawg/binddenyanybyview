See ISC BIND src COPYRIGHT file.

This is personal work product of Robert Sargent, May 15, 2014.  Permission is
granted for use by anyone within the constraints of the original ISC BIND
COPYRIGHT.

This patch (patch-bind9*denyanybyview4freebsd) will add functionality to allow
some operators to better manage an authoritative BIND server by selectively
returning "REFUSED" rcode based on either a global and/or view configuration
boolean option (denyany-byview).  This patch installs this feature as benign,
i.e., defaults to "no" both globally and within views.

One sample enterprise implementation defines three views listed in this order
in named.conf: inside, jokers, internet.  The inside view has a match-clients
list of 127.0.0.1, 10.0.0.0/24 and 192.168.0.0/24.  The jokers view has a
match-clients list of 100+ /24 nets who have only demonstrated a propensity to
send too many non-productive "ANY" queries.  The internet view has a match-clients
list of "any".  The operator must determine who the jokers are.  To activate
this feature simply add "denyany-byview yes;" to the jokers view.

A sample ISP openserver implementation defines two views listed in this order
in named.conf: inside and outside.  The inside view has a match-clients list
of 127.0.0.1, 10.0.0.0/8, 192.168.0.0/16 along with all the network numbers
owned and operated by the ISP.  The outside view has a match-clients list of
"any".  To activate this feature simply add "denyany-byview yes;" to the
recursion-enabled outside view.  Editor's note: the world would be a better
place without openservers but if you must operate one, you can limit qtype ANY
service to just your customers.

To install in FreeBSD ports/dns/bind910 copy the patch-bind910-20140525-denyanybyview
file into the /usr/ports/dns/bind910/files directory. cd /usr/ports/dns/bind910
and do a make clean; make [re]install

To install in FreeBSD ports/dns/bind99 copy the patch-bind99-20140518-denyanybyview
file into the /usr/ports/dns/bind99/files directory. cd /usr/ports/dns/bind99
and do a make clean; make [re]install

To install in other OSs, copy one of the patch files to the bind
src head and do a patch < patch-bind9-blah... and then follow the
normal build/install instructions.

To test it add your client IP address (i.e., move 127.0.0.1 from
the inside view to the jokers view) to the jokers list of IPs and
do a dig @ns1.example.com example.com any

you should get something like this in return:

; <<>> DiG 9.9.5 <<>> @ns1.example.com example.com any
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: REFUSED, id: 1948
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 1
...

The "status: REFUSED" and "ANSWER: 0" above indicate it's working. Don't
forget to remove 127.0.0.1 from jokers view after testing.

There are no warranties expressed nor implied.  Use at your own risk.  No stress
testing, testing with no views or performance testing have been performed on
these changes and are left entirely as an exercise for the operator.

On FreeBSD with ports installed/maintained:

cd /usr/ports/dns/bind99/
make clean
cp ~/patch-bind99-20140518-denyanybyview files
make reinstall

/etc/rc.d/named restart

tarball contents:
-rw-r--r--  0 root   wheel    4878 Jun 25 13:38 README.patch-binddenyanybyview4freebsd
-rw-r--r--  0 root   wheel   47926 May 16 14:26 view.c
-rw-r--r--  0 root   wheel   30099 May 15 10:53 view.h
-rw-r--r--  0 root   wheel   22324 May 15 10:55 config.c
-rw-r--r--  0 root   wheel   86137 May 16 13:55 namedconf.c
-rw-r--r--  0 root   wheel   29135 May 15 10:54 options
-rw-r--r--  0 root   wheel  242843 May 16 22:31 server.c
-rw-r--r--  0 root   wheel    4172 May 18 14:39 patch-bind99-20140518-denyanybyview
-rw-r--r--  0 root   wheel    4132 May 25 22:41 patch-bind910-20140525-denyanybyview
-rw-r--r--  0 root   wheel  220639 May 18 14:52 query.c
-rw-r--r--  0 root   wheel   20932 May 15 10:56 named.conf.5
-rw-r--r--  0 root   wheel     112 May 25 02:35 conf.jokers
-rw-r--r--  0 root   wheel    2498 May 25 02:29 named.conf.sample


tarball MD5s:
MD5 (view.c) = bb48b0c965fedf4980ab4aa6135b63ac
MD5 (view.h) = 0737895c1006f3cdf028e5f994e4966e
MD5 (config.c) = 15b8607bd9d5cbb2a2d006009b2dd953
MD5 (namedconf.c) = 8cb7849e8ada1ec29cb8e1d1b948ec04
MD5 (options) = 618c90b96e1ad79dbb47581ac4059423
MD5 (server.c) = cbf02b8e2d583e9007095d9f4d5bdae8
MD5 (patch-bind99-20140518-denyanybyview) = 73f917f9e9d5eab86ea9214703946bb1
MD5 (patch-bind910-20140525-denyanybyview) = 8fc9c326f4dd2a311e3fa02d5c72cebc
MD5 (query.c) = 82fc4201b3a316c5b5393811a8974648
MD5 (named.conf.5) = 45730f03caec90fcb3e0853924cf1ee0
MD5 (conf.jokers) = 974e5191655439b190f952cf8aded817
MD5 (named.conf.sample) = 99a555f0588ade488c4146ae89b65c2a

todo:

add tcp test "if (TCPCLIENT(client)) { ... }" to allow TCP connections (proves no spoofing)
