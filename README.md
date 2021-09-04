# Multi-Server-Setup
This is the multi-server configuration  setup for dedicated server addresses for the panel, web, DNS, mail, and webmail. Both the DNS and mail server will have a mirror server for redundancy. 

<h1> Multiserver setup on Ubuntu 20.04 and Debian 10</h1>
<div class=contributeEdit style=float:right;width:450px;max-width:100%>
<div id=tocContainer>
<h3>On this page</h3>
<ol class=toc>
<li><a href=#g0.0.2>1. Preliminary Note</a>
<li><a href=#-installing-the-master-server>2. Installing the master server</a>
<ol>
<li><a href=#-configure-the-hostname-and-hosts>2.1 Configure the hostname and hosts</a>
<li><a href=#-setting-up-the-remote-mysql-users-for-our-slave-servers>2.2 Setting up the remote MySQL users for our slave servers</a>
<li><a href=#-setting-up-the-firewall>2.3 Setting up the firewall</a>
</ol>
</ol>
</div>
</div>
<h2 id=g0.0.2>1. Preliminary Note</h2>
<p>These will be the hosts we're installing:
<pre>host       FQDN                   IP<br>panel      panel.example.com      10.0.64.12<br>web01      web01.example.com      10.0.64.13<br>mx1        mx1.example.com        10.0.64.14<br>mx2        mx2.example.com        10.0.64.15<br>ns1        ns1.example.com        10.0.64.16<br>ns2        ns2.example.com        10.0.64.17<br>webmail    webmail.example.com    10.0.64.18</pre>
<p>We will be using example hostnames, IP addresses, and IP ranges. Make sure to change them accordingly in your commands/configuration.<div style="width:336px;float:left;margin:10px 15px 10px 0;background-color:#fff">
</div>
<p>All servers are on the same private network but have their own public IP. If your servers don't have a shared local network, use their public IPv4 addresses.
<p>Before starting the installation of a server, set up an A and eventual AAAA record that points to the <strong>public</strong> IP address of your server. For example, if the hostname is panel.example.com and the public IP is 11.22.33.44, you should set up an A record for panel.example.com pointing to 11.22.33.44. <strong>Every server should have its own public IP and hostname.</strong><strong></strong>
<h2 id=-installing-the-master-server>2. Installing the master server</h2>
<p>Log in as root or run
<pre class=command>su -</pre>
<p>to become the root user on your server before you proceed. <strong>IMPORTANT</strong>: You must use 'su -' and not just 'su', otherwise your PATH variable is set wrong by Debian.
<h3 id=-configure-the-hostname-and-hosts>2.1 Configure the hostname and hosts</h3>
<p>The hostname of your server should be a subdomain like "panel.example.com". Do not use a domain name without a subdomain part like "example.com" as hostname as this will cause problems later with your mail setup. First, you should check the hostname in <span class=system>/etc/hosts</span> and change it when necessary. The line should be: "IP Address - space - full hostname incl. domain - space - subdomain part". For our hostname panel.example.com, the file shall look like this (some lines may be different, it can differ per hosting provider):
<pre>127.0.0.1 localhost.localdomain   localhost<br># This line should be changed on every node to the correct servername:<br>127.0.1.1 panel.example.com panel<br># These lines are the same on every node:
10.0.64.12 panel.example.com panel<br>10.0.64.13 web01.example.com web01<br>10.0.64.14 mx1.example.com mx1<br>10.0.64.15 mx2.example.com mx2<br>10.0.64.16 ns1.example.com ns1<br>10.0.64.17 ns2.example.com ns2<br>10.0.64.18 webmail.example.com webmail

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters</pre>
<p>As you can see, we added the hostnames of our other servers aswell, so they can communicate over the internal network later.
<p>Then edit the /etc/hostname file:
<pre class=command>nano /etc/hostname</pre>
<p>It shall contain only the subdomain part, in our case:
<pre>panel</pre>
<p>Finally, reboot the server to apply the change:
<pre class=command><span>systemctl reboot</span></pre>
<p>Log in again and check if the hostname is correct now with these commands:<span id=ezoic-pub-ad-placeholder-110 class=ezoic-adpicker-ad></span><span class="ezoic-ad box-4 box-4110 adtester-container adtester-container-110" data-ez-name=howtoforge_com-box-4><span id=div-gpt-ad-howtoforge_com-box-4-0 ezaw=580 ezah=400 style=position:relative;z-index:0;display:inline-block;padding:0;width:100%;max-width:1200px;margin-left:auto!important;margin-right:auto!important;min-height:400px;min-width:580px class=ezoic-ad><script data-ezscrex=false data-cfasync=false style=display:none>typeof __ez_fad_position!='undefined'&&__ez_fad_position('div-gpt-ad-howtoforge_com-box-4-0')</script></span></span>
<pre class=command>hostname<br>hostname -f</pre>
<p>The output shall be like this:
<pre class=system><a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="82f0ededf6c2f2e3ece7ee">[email&#160;protected]</a>:~$ hostname<br>panel<br><a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="e5978a8a91a595848b8089">[email&#160;protected]</a>:~$ hostname -f<br>panel.example.com</pre>
<p>Now we can run the autoinstaller to install all necessary packages and ISPConfig:
<pre>wget -O - https://get.ispconfig.org | sh -s -- --no-mail --no-dns --use-php=system</pre>
<p>After some time, you will see:
<pre class=command>WARNING! This script will reconfigure your complete server!<br>It should be run on a freshly installed server and all current configuration that you have done will most likely be lost!<br>Type &#39;yes&#39; if you really want to continue:</pre>
<p>Answer "yes" and hit enter. The installer will now start.
<p>When the installer is finished it will show you the ISPConfig admin and MySQL root password like this:<span id=ezoic-pub-ad-placeholder-111 class=ezoic-adpicker-ad></span>
<pre>[INFO] Your ISPConfig admin password is: 5GvfSSSYsdfdYC<br>[INFO] Your MySQL root password is: kkAkft82d!kafMwqxdtYs</pre>
<p>Make sure you write this information down, along with server they are for, as you will need them later.
<h3 id=-setting-up-the-remote-mysql-users-for-our-slave-servers>2.2 Setting up the remote MySQL users for our slave servers</h3>
<p>We will log in to MySQL to allow the other servers to connect to the ISPConfig database on this node during installation, by adding MySQL root user records in the master database for every slave server hostname and IP address.
<p>On the terminal, run
<pre class=command>mysql -u root -p</pre>
<p>Enter your MySQL password and then run the following commands:
<pre class=command>CREATE USER &#39;root&#39;@&#39;<span class=highlight>10.0.64.13</span>&#39; IDENTIFIED BY &#39;<span class=highlight>myrootpassword</span>&#39;;<br>GRANT ALL PRIVILEGES ON * . * TO &#39;root&#39;@&#39;<span class=highlight>10.0.64.13</span>&#39; IDENTIFIED BY &#39;<span class=highlight>myrootpassword</span>&#39; WITH GRANT OPTION MAX_QUERIES_PER_HOUR 0 MAX_CONNECTIONS_PER_HOUR 0 MAX_UPDATES_PER_HOUR 0 MAX_USER_CONNECTIONS 0 ;</pre>
<pre class=command>CREATE USER &#39;root&#39;@&#39;<span class=highlight>10.0.64.14</span>&#39; IDENTIFIED BY &#39;<span class=highlight>myrootpassword</span>&#39;;<br>GRANT ALL PRIVILEGES ON * . * TO &#39;root&#39;@&#39;<span class=highlight>10.0.64.14</span>&#39; IDENTIFIED BY &#39;<span class=highlight>myrootpassword</span>&#39; WITH GRANT OPTION MAX_QUERIES_PER_HOUR 0 MAX_CONNECTIONS_PER_HOUR 0 MAX_UPDATES_PER_HOUR 0 MAX_USER_CONNECTIONS 0 ;</pre>
<pre class=command>CREATE USER &#39;root&#39;@&#39;<span class=highlight>10.0.64.15</span>&#39; IDENTIFIED BY &#39;<span class=highlight>myrootpassword</span>&#39;;<br>GRANT ALL PRIVILEGES ON * . * TO &#39;root&#39;@&#39;<span class=highlight>10.0.64.15</span>&#39; IDENTIFIED BY &#39;<span class=highlight>myrootpassword</span>&#39; WITH GRANT OPTION MAX_QUERIES_PER_HOUR 0 MAX_CONNECTIONS_PER_HOUR 0 MAX_UPDATES_PER_HOUR 0 MAX_USER_CONNECTIONS 0 ;</pre>
<pre class=command>CREATE USER &#39;root&#39;@&#39;<span class=highlight>10.0.64.16</span>&#39; IDENTIFIED BY &#39;<span class=highlight>myrootpassword</span>&#39;;<br>GRANT ALL PRIVILEGES ON * . * TO &#39;root&#39;@&#39;<span class=highlight>10.0.64.16</span>&#39; IDENTIFIED BY &#39;<span class=highlight>myrootpassword</span>&#39; WITH GRANT OPTION MAX_QUERIES_PER_HOUR 0 MAX_CONNECTIONS_PER_HOUR 0 MAX_UPDATES_PER_HOUR 0 MAX_USER_CONNECTIONS 0 ;</pre>
<pre class=command>CREATE USER &#39;root&#39;@&#39;<span class=highlight>10.0.64.17</span>&#39; IDENTIFIED BY &#39;<span class=highlight>myrootpassword</span>&#39;;<br>GRANT ALL PRIVILEGES ON * . * TO &#39;root&#39;@&#39;<span class=highlight>10.0.64.17</span>&#39; IDENTIFIED BY &#39;<span class=highlight>myrootpassword</span>&#39; WITH GRANT OPTION MAX_QUERIES_PER_HOUR 0 MAX_CONNECTIONS_PER_HOUR 0 MAX_UPDATES_PER_HOUR 0 MAX_USER_CONNECTIONS 0 ;</pre>
<pre class=command>CREATE USER &#39;root&#39;@&#39;<span class=highlight>10.0.64.18</span>&#39; IDENTIFIED BY &#39;<span class=highlight>myrootpassword</span>&#39;;<br>GRANT ALL PRIVILEGES ON * . * TO &#39;root&#39;@&#39;<span class=highlight>10.0.64.18</span>&#39; IDENTIFIED BY &#39;<span class=highlight>myrootpassword</span>&#39; WITH GRANT OPTION MAX_QUERIES_PER_HOUR 0 MAX_CONNECTIONS_PER_HOUR 0 MAX_UPDATES_PER_HOUR 0 MAX_USER_CONNECTIONS 0 ;</pre>
<pre class=command>CREATE USER &#39;root&#39;@&#39;<span class=highlight>web01.example.com</span>&#39; IDENTIFIED BY &#39;<span class=highlight>myrootpassword</span>&#39;;<br>GRANT ALL PRIVILEGES ON * . * TO &#39;root&#39;@&#39;<span class=highlight>web01.example.com</span>&#39; IDENTIFIED BY &#39;<span class=highlight>myrootpassword</span>&#39; WITH GRANT OPTION MAX_QUERIES_PER_HOUR 0 MAX_CONNECTIONS_PER_HOUR 0 MAX_UPDATES_PER_HOUR 0 MAX_USER_CONNECTIONS 0 ;</pre>
<pre class=command>CREATE USER &#39;root&#39;@&#39;<span class=highlight>mx1.example.com</span>&#39; IDENTIFIED BY &#39;<span class=highlight>myrootpassword</span>&#39;;<br>GRANT ALL PRIVILEGES ON * . * TO &#39;root&#39;@&#39;<span class=highlight>mx1.example.com</span>&#39; IDENTIFIED BY &#39;<span class=highlight>myrootpassword</span>&#39; WITH GRANT OPTION MAX_QUERIES_PER_HOUR 0 MAX_CONNECTIONS_PER_HOUR 0 MAX_UPDATES_PER_HOUR 0 MAX_USER_CONNECTIONS 0 ;</pre>
<pre class=command>CREATE USER &#39;root&#39;@&#39;<span class=highlight>mx2.example.com</span>&#39; IDENTIFIED BY &#39;<span class=highlight>myrootpassword</span>&#39;;<br>GRANT ALL PRIVILEGES ON * . * TO &#39;root&#39;@&#39;<span class=highlight>mx2.example.com</span>&#39; IDENTIFIED BY &#39;<span class=highlight>myrootpassword</span>&#39; WITH GRANT OPTION MAX_QUERIES_PER_HOUR 0 MAX_CONNECTIONS_PER_HOUR 0 MAX_UPDATES_PER_HOUR 0 MAX_USER_CONNECTIONS 0 ;</pre>
<pre class=command>CREATE USER &#39;root&#39;@&#39;<span class=highlight>ns1.example.com</span>&#39; IDENTIFIED BY &#39;<span class=highlight>myrootpassword</span>&#39;;<br>GRANT ALL PRIVILEGES ON * . * TO &#39;root&#39;@&#39;<span class=highlight>ns1.example.com</span>&#39; IDENTIFIED BY &#39;<span class=highlight>myrootpassword</span>&#39; WITH GRANT OPTION MAX_QUERIES_PER_HOUR 0 MAX_CONNECTIONS_PER_HOUR 0 MAX_UPDATES_PER_HOUR 0 MAX_USER_CONNECTIONS 0 ;</pre>
<pre class=command>CREATE USER &#39;root&#39;@&#39;<span class=highlight>ns2.example.com</span>&#39; IDENTIFIED BY &#39;<span class=highlight>myrootpassword</span>&#39;;<br>GRANT ALL PRIVILEGES ON * . * TO &#39;root&#39;@&#39;<span class=highlight>ns2.example.com</span>&#39; IDENTIFIED BY &#39;<span class=highlight>myrootpassword</span>&#39; WITH GRANT OPTION MAX_QUERIES_PER_HOUR 0 MAX_CONNECTIONS_PER_HOUR 0 MAX_UPDATES_PER_HOUR 0 MAX_USER_CONNECTIONS 0 ;</pre>
<pre class=command>CREATE USER &#39;root&#39;@&#39;<span class=highlight>webmail.example.com</span>&#39; IDENTIFIED BY &#39;<span class=highlight>myrootpassword</span>&#39;;<br>GRANT ALL PRIVILEGES ON * . * TO &#39;root&#39;@&#39;<span class=highlight>webmail.example.com</span>&#39; IDENTIFIED BY &#39;<span class=highlight>myrootpassword</span>&#39; WITH GRANT OPTION MAX_QUERIES_PER_HOUR 0 MAX_CONNECTIONS_PER_HOUR 0 MAX_UPDATES_PER_HOUR 0 MAX_USER_CONNECTIONS 0 ;</pre>
<p>In the above SQL commands, replace the IP adresses (<span class=system>10.0.64.12 - 10.0.64.18</span>) with the IP addresses of your servers, <span class=system>web01.example.com</span>, <span class=system>mx1.example.com</span>, <span class=system>mx2.example.com</span>, <span class=system>ns1.example.com</span>, <span class=system>ns2.example.com</span>, and <span class=system>webmail.example.com </span>with the hostnames of your servers and <span class=system>myrootpassword</span> with the desired root password (it is good practice to use a different password for each host. Write them down, as you will need them later when installing or updating your slave servers).<span id=ezoic-pub-ad-placeholder-112 class=ezoic-adpicker-ad></span>
<p>When this is done, you can exit MySQL with:
<pre class=command>EXIT;</pre>
<p>You can now log in to ISPConfig on <a href=https://panel.example.com:8080 target=_blank rel=noopener>https://panel.example.com:8080</a> with the username admin and the password the installer showed you.
<h3 id=-setting-up-the-firewall>2.3 Setting up the firewall</h3>
<p>The last thing to do is to set up our firewall.
<p>Log in to the ISPConfig UI, and go to System -> Firewall. Then click "Add new firewall record".
<p>For the panel server, we have to open the following ports:
<p>TCP:
<pre>22,80,443,8080,8081</pre>
<p>No UDP ports have to be opened through the UI.
<p>We are also going to open port 3306, which is used for MySQL, but only from our local network for security reasons. To do so, run the following command from the CLI, after the change from the ISPConfig panel is propagated (when the red dot is gone):
<pre class=command>ufw allow from 10.0.64.0/24 to any port 3306 proto tcp</pre>
<p>Your panel is now set up and ready for use.
