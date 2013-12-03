wocut
=====

Tool to manage iptables-based load-balancing to local servers

Usage scenario
==============
Let's say you want to make a service scale, but the only software you have available for that service is
either single-threaded or has a bottleneck in its thread manager. And let's say you're on Linux, so you have
iptables available to you.

If you don't mind wrapping the daemon so you have multiple instances of it listening on different ports,
dropping different pidfiles, logging to different files, and all the other things a daemon should do, you can
use this to handle the IPTables side of things so your kernel will be able to randomly assign incoming connections
to one of those daemons, and for the duration of that connection the same daemon will manage everything it needs.
So far this has been tested with apache, nginx, OpenVPN, HAProxy, node (with code to act as a STunnel endpoint) and a
variety of other services. It makes them all scale better.

You do need to write your own scripts to launch/manage those services, probably including generating configfiles.

What exactly you get
====================
IPTables needs two kinds of rules to do this:
1) It marks incoming connections using a sequence of statistics, in the PREROUTING/mangle CT
2) It redirects those to a local port, in the PREROUTING/nat CT

wocut manages those sets of rules, letting you insert and remove them from a running configuration independently.

Caveats
=======
Existing connections will persist after the rules are removed, if there's still a daemon listening. This is how
the nat rules work.

wocut will use packet marks that are identical to the destination port for an instance of a service. If you do a
lot of your own marking, you need to be aware of this and adjust accordingly

Right now, wocut won't tell you if you insert two rulesets that interfere with each other. Be careful.

If you do a lot of your own iptables manipulation, you need to be careful in making sure wocut's rules land in
the CTs before any other rules that might interfere with them, and possibly after any other rules that you want to
be considered first.
