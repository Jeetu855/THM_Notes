(**Better to use shodan on Host OS**)

Shodan.io is a search engine for the Internet of Things.
Shodan scans the whole internet and indexes the services run on each IP address.

Let’s say we are performing a pentest on a company, and we want to find out what services one of their servers run.

We need to grab their IP address. We can do this using `ping`.

We can ping tryhackme.com and the ping response will tell us their IP address.

Then once we do this, we put the IP address into Shodan to get more info about the services

![[googleASN.png]]


We can see that TryHackMe runs on Cloudflare in the United States and they have many ports open.

***Cloudflare acts as a proxy between TryHackMe and their real servers. If we were pentesting a large company, this isn’t helpful. We need some way to get their IP addresses.***

We can do this using Autonomous System Numbers.

***If website using Proxy like Cloudflare, when we do dig or nslookup, we get the IP of  Cloudflare and not the actual IP of website.***

An autonomous system number (ASN) is a global identifier of a range of IP addresses. If you are an enormous company like Google you will likely have your own ASN for all of the IP addresses you own.


We can put the IP address into an ASN lookup tool such as \https://www.ultratools.com/tools/asnInfo,
Which tells us they have the ASN AS14061.

On Shodan.io,we can search using the ASN filter. The filter is `ASN:[number]` where number is the number we got from earlier, which is AS14061.

```sh
asn:AS14061
```


![[googleASN2.png]]


Knowing the ASN is helpful, because we can search Shodan for things such as coffee makers or vulnerable computers within our ASN, which we know (if we are a large company) is on our network.

Devices run services, and Shodan stores information about them. The information is stored in a _banner_. It’s the most fundamental part of Shodan.

---

#### Filters

On the Shodan.io homepage, we can click on “explore” to view the most up voted search queries. The most popular one is webcams.

```sh
product:MySQL
```

```sh
asn:AS14061 product:MySQL
```


![[productMySQL.png]]

Let’s say we want to find IP addresses vulnerable to Eternal Blue:

```sh
vuln:ms17-010
```

**However, this is only available for academic or business users, to prevent bad actors from abusing this!**

```sh
asn:AS15169 product:"MySQL"
```

```sh
asn:AS15169 country:"US"
```

```sh
asn:AS15169 country:"US" city:"Los Angeles"
```


---

#### Shodan Monitor


Shodan Monitor is an application for monitoring your devices in your own network. In their words:

 Keep track of the devices that you have exposed to the Internet. Setup notifications, launch scans and gain complete visibility into what you have connected.

Once we add a network, we can see it in our dashboard.
If we click on the settings cog, we can see that we have a range of “scans” Shodan performs against our network.

Anytime Shodan detects a security vulnrability in one of these categories, it will email us.

If we go to the dashboard again we can see it lays some things out for us.
Most notably:

- Top Open Ports (most common)
- Top Vulnerabilities (stuff we need to deal with right away)
- Notable Ports (unusual ports that are open)
- Potential Vulenrabilites
- Notable IPs (things we should investigate in more depth).

The interesting part is that you can actually monitor other peoples networks using this. For bug bounties you can save a list of IPs and Shodan will email you if it finds any problems.


---

#### Shodan Dorking

```sh
has_screenshot:true encrypted attention
```

Which uses optical character recognition and remote desktop to find machines compromised by ransomware on the internet.

```sh
vuln:CVE-2014-0160
```

CVE search is only allowed to academic or business subscribers.

---

#### Shodan Extension

When installed, you can click on it and it’ll tell you the IP address of the webserver running, what ports are open, where it’s based and if it has any security issues.

---

Articles

\https://github.com/beesecurity/How-I-Hacked-Your-Pi-Hole/blob/master/README.md
\https://skerritt.blog/shodan/
