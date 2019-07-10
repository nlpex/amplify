---
layout: post
title: Build net-snmp on macOS 10.14.5 (Mojave)
---

```diff
diff --git a/Makefile.in b/Makefile.in
index 912f6b24d..b32706717 100644
--- a/Makefile.in
+++ b/Makefile.in
@@ -18,7 +18,7 @@ INSTALLHEADERS=version.h net-snmp-features.h
 INCLUDESUBDIR=system
 INCLUDESUBDIRHEADERS= aix.h bsd.h bsdi3.h bsdi4.h bsdi.h cygwin.h \
        darwin.h darwin7.h darwin8.h darwin9.h darwin10.h darwin11.h darwin12.h \
-       darwin13.h darwin14.h darwin15.h darwin16.h darwin17.h \
+       darwin13.h darwin14.h darwin15.h darwin16.h darwin17.h darwin18.h \
        dragonfly.h dynix.h \
        freebsd2.h freebsd3.h freebsd4.h freebsd5.h freebsd6.h \
        freebsd7.h freebsd8.h freebsd9.h freebsd10.h freebsd11.h \
diff --git a/include/net-snmp/system/darwin18.h b/include/net-snmp/system/darwin18.h
new file mode 100644
index 000000000..565a09801
--- /dev/null
+++ b/include/net-snmp/system/darwin18.h
@@ -0,0 +1 @@
+#include <net-snmp/system/darwin17.h>
```

where darwin18 is the version of darwin (`uname -r` => `18.6.0`).

This fixes `make` error:

```hs
making all in /Users/wsh/src/net-snmp/build/agent/mibgroup
/bin/sh ../../libtool  --mode=compile gcc -I../../include -I../../../include -I. -I../../agent -I../../../agent -I../../agent/mibgroup -I../../../agent/mibgroup  -I../../snmplib -I../../../snmplib -D_GNU_SOURCE -D_ALL_SOURCE -D_THREAD_SAFE -D__EXTENSIONS__   -ggdb3 -O0 -DNETSNMP_ENABLE_IPV6 -fno-strict-aliasing -DNETSNMP_REMOVE_U64 -ggdb3 -O0 -Udarwin18 -Ddarwin18=darwin18  -Wall -Wextra -Wstrict-prototypes -Wwrite-strings -Wcast-qual -Wno-missing-field-initializers -Wno-sign-compare -Wno-unused-parameter -Wno-type-limits -c -o mibII/tcp.lo ../../../agent/mibgroup/mibII/tcp.c
libtool: compile:  gcc -I../../include -I../../../include -I. -I../../agent -I../../../agent -I../../agent/mibgroup -I../../../agent/mibgroup -I../../snmplib -I../../../snmplib -D_GNU_SOURCE -D_ALL_SOURCE -D_THREAD_SAFE -D__EXTENSIONS__ -ggdb3 -O0 -DNETSNMP_ENABLE_IPV6 -fno-strict-aliasing -DNETSNMP_REMOVE_U64 -ggdb3 -O0 -Udarwin18 -Ddarwin18=darwin18 -Wall -Wextra -Wstrict-prototypes -Wwrite-strings -Wcast-qual -Wno-missing-field-initializers -Wno-sign-compare -Wno-unused-parameter -Wno-type-limits -c ../../../agent/mibgroup/mibII/tcp.c  -fno-common -DPIC -o mibII/.libs/tcp.o
../../../agent/mibgroup/mibII/tcp.c:343:21: error: use of undeclared identifier 'TCPTV_MIN'
        ret_value = TCPTV_MIN / PR_SLOWHZ * 1000;
                    ^
../../../agent/mibgroup/mibII/tcp.c:351:21: error: use of undeclared identifier 'TCPTV_REXMTMAX'
        ret_value = TCPTV_REXMTMAX / PR_SLOWHZ * 1000;
                    ^
2 errors generated.
make[2]: *** [mibII/tcp.lo] Error 1
make[1]: *** [subdirs] Error 1
make: *** [subdirs] Error 1

Process finished with exit code 2
```

I didn't send this patch to [the upstream](https://sourceforge.net/p/net-snmp/patches/) because:

- it was too expensive for me to understand the way of sending patch to the net-snmp project
- the most of the sent patches seem to be neglected

## See also

- [Homebrew net-snmp.rb](https://github.com/Homebrew/homebrew-core/blob/master/Formula/net-snmp.rb)
