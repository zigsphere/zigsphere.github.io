---
layout: post
title: Self-hosted DNS Server
tags: [Tech]
author: zigsphere
excerpt_separator: <!--more-->
---

I didn't think it was a good idea to host my own DNS server back in the day. Was always thinking in the back of my mind that people would DDoS it. Well, one day I exposed my Raspberry PI-hosted DNS server to the internet intentionally, and that's exactly what happened. 

For a few years now, I've been hosting my own DNS server on my LAN using [PI-Hole](https://github.com/pi-hole/pi-hole), which is basically a DNS server which blocks all ad-related traffic. It saves a lot of time loading pages and prevents ads from showing up everywhere on my computer, including in-game ads on my phone. Was great! I wanted to take it to the next level and host PI-Hole on the internet. Yes, PI-Hole was made for the Raspberry PI, but it was indeed compatible with other Linux-based operating systems such as Debian, Ubuntu, and CentOS. I opened up my PI to the internet to allow my phone and other devices to connect to it from anywhere. This way, I would never get ads no matter where I was. The disadvantage, of course, is that extra hop I have to take when making a DNS lookup from my devices to my house. Was there a difference? Yes, a little, but I still think it was worth it. 

I made the mistake of just opening port 53 with no type of rate limiting or dropping of IPs if there was a DDoS or some type of DNS amplification attack. I thought, what's the worst that could happen? I was being DDoSed by an IP in Hong Hong. I closed the port immediately and then read up on some steps I could take to somewhat prevent it from happening again.

* I stopped hosting PI-Hole from my Raspberry PI and instead started hosting using a cloud service, Linode.
* I implemented a service called fail2ban, which is a service that scans log files and bans IPs that show malicious activity. I also use it to update firewall rules accordingly.
* Used Puppet for configuration management.
* Implemented cron jobs to update blocked DNS records weekly.
* Closed all inbound ports except for the ones being used for DNS and HTTP.

After the above steps were completed, my cloud-hosted DNS service was working wonderfully. It's actually working much better than the PI hosted at home, with roughly the same speeds, even with that extra hop. Anyone is welcome to use it. Please message me if you would like the IP.

---

* [Steps](https://freek.ws/2017/03/18/blocking-dns-amplification-attacks-using-iptables/) performed for Fail2ban setup. Steps shown in documentation have been made to the Puppet configuration below.

{% highlight ruby %}
class profile::dns {

  include profile::firewall::web_server

  package { 'fail2ban':
    ensure => installed,
  }

  file { '/etc/network/interfaces.tail':
    ensure => file,
    owner  => 'root',
    mode   => '0755',
    source => 'puppet:///modules/profile/dns/interfaces.tail',
  }

  file { '/etc/fail2ban/filter.d/iptables-dns.conf':
    ensure  => file,
    owner   => 'root',
    mode    => '0755',
    source  => 'puppet:///modules/profile/dns/iptables-dns.conf',
    require => Package['fail2ban']
  }

  file { '/etc/fail2ban/jail.local':
    ensure  => file,
    owner   => 'root',
    mode    => '0644',
    source  => 'puppet:///modules/profile/dns/jail.conf',
    require => Package['fail2ban']
  }

  remote_file { '/usr/local/bin/gravity.sh':
    ensure => present,
    source => 'https://raw.githubusercontent.com/pi-hole/pi-hole/master/gravity.sh',
    owner  => 'root',
    mode   => '0744'
  }

  cron { 'gravity.sh':
    command => '/bin/bash /usr/local/bin/gravity.sh',
    user    => 'root',
    hour    => '3',
    minute  => '0',
    weekday => '6'
  }

  firewall { '100 allow dns access':
    ensure => present,
    dport  => '53',
    proto  => 'udp',
    action => 'accept',
  }
}
{% endhighlight %}
<center>
For questions about the complete Puppet configuration setup, please email joseph@josephziegler.com and I'd be happy to provide that information.</center>

<center><img src="https://www.josephziegler.com/media/pihole.png" width="400" alt="house1"></center>