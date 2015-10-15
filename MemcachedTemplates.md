# Introduction #

A recent addition to the suite are templates Memcached/Membase.

# Defined Graphs #

( **NOTE:** check back here for an example of each of these graphs in the coming weeks )

  * CAS Hits/Misses
  * CPU Time
  * Commands
  * Connection Rate
  * Connections
  * Current Items
  * Decr/Incr Hits/Misses
  * Delete Hits/Misses
  * Get Hits/Misses
  * Items/Evictions
  * Memory Usage
  * Network Traffic
  * Reclaimed
  * Threads

# Defined Triggers #

The list of triggers is presently a little thin, but, is going to grow over time as this set of templates matures.

  * Memcached port not responding
  * Memcached process not running
  * Memcached CPU user time over 90%

# Example Graph #

![http://wiki.appaloosa-zabbix-templates.googlecode.com/hg/img/memcached_memory.png](http://wiki.appaloosa-zabbix-templates.googlecode.com/hg/img/memcached_memory.png)

# Installation #

Begin by following the basic instructions in [Installation](Installation.md).

## Additional options ##

This template accepts one additional option on the gen\_template commandline:

  * --memcached-port<br />Default: 11211<br />Example: `tools/gen_template.pl defs/memcached.pl memcached_9000.xml --memcached-port 9000`