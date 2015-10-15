# Introduction #

These templates add support for monitoring the BIND DNS server,
version 9.

# Graphs #

## Global QPS ##
![http://wiki.appaloosa-zabbix-templates.googlecode.com/hg/img/bind9_global_qps.png](http://wiki.appaloosa-zabbix-templates.googlecode.com/hg/img/bind9_global_qps.png)

## QPS grouped by response ##
![http://wiki.appaloosa-zabbix-templates.googlecode.com/hg/img/bind9_global_queries.png](http://wiki.appaloosa-zabbix-templates.googlecode.com/hg/img/bind9_global_queries.png)

## Memory usage of BIND virtual vs. resident ##
![http://wiki.appaloosa-zabbix-templates.googlecode.com/hg/img/bind9_memory_usage.png](http://wiki.appaloosa-zabbix-templates.googlecode.com/hg/img/bind9_memory_usage.png)

## Query latency ##
![http://wiki.appaloosa-zabbix-templates.googlecode.com/hg/img/bind9_query_latency.png](http://wiki.appaloosa-zabbix-templates.googlecode.com/hg/img/bind9_query_latency.png)

# Triggers Defined #

  * Named not running
  * Query latency exceeds 5 seconds

# Installation #

Refer to [Installation](Installation.md) for generic installation instructions common
to all templates.<br>
This section details the particulars needed for<br>
monitoring BIND.<br>
Both named, and the user script need special<br>
configuration to function correctly.<br>
<br>
<h2>Additional arguments for gen_template</h2>

This template accepts the following additional gen_template arguments:<br>
<br>
<ul><li>--domain<br />Generate a template for a specific zone, instead of the global zone.<br />Example: <code>tools/gen_template.pl defs/bind9.pl bind9_example.com.xml --domain example.com</code></li></ul>

<h2>Named</h2>

Named is not configured out of the box to support the <code>rndc stats</code>
command which the user script relies on. To enable support, you<br>
must add a couple of sections to <code>named.conf</code>.<br>
<br>
On Redhat and Centos, there's a tool called <code>rndc-confgen</code> which<br>
does exactly what you want.<br>
<br>
Example:<br>
<pre><code>[root@ip-10-122-129-242 ~]# rndc-confgen <br>
# Start of rndc.conf<br>
key "rndckey" {<br>
	algorithm hmac-md5;<br>
	secret "uSihJWW/idzcHT6SLqa1Mw==";<br>
};<br>
<br>
options {<br>
	default-key "rndckey";<br>
	default-server 127.0.0.1;<br>
	default-port 953;<br>
};<br>
# End of rndc.conf<br>
<br>
# Use with the following in named.conf, adjusting the allow list as needed:<br>
# key "rndckey" {<br>
# 	algorithm hmac-md5;<br>
# 	secret "uSihJWW/idzcHT6SLqa1Mw==";<br>
# };<br>
# <br>
# controls {<br>
# 	inet 127.0.0.1 port 953<br>
# 		allow { 127.0.0.1; } keys { "rndckey"; };<br>
# };<br>
# End of named.conf<br>
<br>
</code></pre>

<ol><li>Place the first section (between <code>Start of rndc.conf</code> and <code>End of rndc.conf</code>) in /etc/rndc.conf<br>
</li><li>Place the second section inside your named.conf.<br>On Redhat, with the <code>bind-chroot</code> package installed, named.conf lives at <code>/var/named/chroot/etc/named.conf</code>.</li></ol>

If you intend to make use of the per-zone statistics, you need to add the following to<br>
the global section in <code>named.cond</code>:<br>
<br>
<pre><code>zone-statistics yes;<br>
</code></pre>

<h2>The user script</h2>

<b>NOTE:</b> The monitoring script needs extended privileges.<br>
In order to work it <b>MUST</b> be setup in one of the following ways:<br>
<br>
<ul><li>Installed SUID root or named user,<br>
</li><li>Called via sudo as either root or named user.</li></ul>

This is because <code>rndc</code> which this plugin makes use of, causes named<br>
to write data to files in the named datadir, which is tightly owned<br>
by the named user under any sane installation.<br>
<br>
The following perl modules are needed:<br>
<br>
<ul><li>Net::DNS (perl-Net-DNS RPM)</li></ul>

The SUID option likely requires the least setup.<br>
The necessary steps on Redhat or Centos are:<br>
<br>
<pre><code>chown root:root bind9_stats.pl<br>
chmod +sx bind9_stats.pl<br>
yum install perl-suidperl<br>
</code></pre>

You could replace <code>root:root</code> above with <code>named:named</code>, as well.<br>
<br>
Setting up sudo requires adding a line like the following<br>
to <code>/etc/sudoers</code>:<br>
<br>
<pre><code>zabbix  ALL=(ALL)  NOPASSWD:/usr/local/zabbix/plugins/bind9_stats.pl<br>
</code></pre>

And then modifying the <code>UserParameter</code> from the generic instructions<br>
to read:<br>
<br>
<pre><code>UserParameter=bind9[*],sudo /usr/bin/perl /usr/local/zabbix/plugins/bind9_stats.pl $1 $2<br>
</code></pre>

<h3>Configuration</h3>

The user script has a few configurables at the top which allow you to<br>
specify the location of various resources necessary for proper<br>
operation. Their default values reflect the default CentOS/RedHat<br>
setup. The configuration values are clearly documented, so, all that<br>
should be necessary is to open the script with your favorite editor<br>
and customize.<br>
<br>
<br>
<h2>Troubleshooting</h2>

Setting the environment variable <code>DEBUG</code> will dump messages from <code>rndc</code>
to <code>STDERR</code> when it encounters an error.<br>
<br>
Verify that all of your paths are correct at the top of<br>
<code>bind9_stats.pl</code>.<br>
<br>
Ensure that it works consistently in your <code>zabbix</code> user account.<br>
<br>
Verify that <code>/etc/rndc.conf</code> has correct permissions so that it can be<br>
read by a SUID script. Permissions of <code>0600 named:named</code> are known to<br>
work.<br>
<br>
If you're using the SUID method described above, be sure that you've<br>
done Set <b>UID</b>, not Set <b>GID</b>. You might have accidentally done<br>
<code>chmod g+sx</code> while on auto-pilot. For reference, the permissions bits<br>
should look like the below:<br>
<br>
<pre><code>-rwsr-sr-x 1 root root 5.8K Dec 18 01:16 /usr/local/bin/bind9_stats.pl*<br>
</code></pre>

Ensure that the stats file produced by <code>rndc stats</code> actually contains<br>
information for the zone you want. We've observed that some<br>
configurations only show the global zone. We will update this document<br>
when we know exactly what is necessary. Make sure you have the <code>zone-statistics</code>
option present in the config. See above for exact syntax.