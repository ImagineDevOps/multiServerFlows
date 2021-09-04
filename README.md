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
  
  <h1>ISPConfig Perfect Multiserver setup on Ubuntu 20.04 and Debian 10 - Page 2</h1>
<div class=contributeEdit style=float:right;width:450px;max-width:100%>
<div id=tocContainer>
<h3>On this page</h3>
<ol class=toc>
<li><a href=#-installing-the-webserver>3 Installing the webserver</a>
<ol>
<li><a href=#-configure-the-hostname>3.1 Configure the hostname</a>
<li><a href=#-installing-ispconfig>3.2 Installing ISPConfig</a>
<li><a href=#-setting-up-the-firewall>3.3 Setting up the firewall</a>
</ol>
</ol>
</div>
</div>
<h2 id=-installing-the-webserver>3 Installing the webserver</h2>
<p>Log in as root or run </p>
<pre class=command>su -</pre>
<p>to become root user on your server before you proceed. <strong>IMPORTANT</strong>: You must use 'su -' and not just 'su', otherwise your PATH variable is set wrong by Debian.</p?
<h3 id=-configure-the-hostname>3.1 Configure the hostname</h3>
<p>The hostname of your server should be a subdomain like "web01.example.com". Do not use a domain name without a subdomain part like "example.com" as hostname as this will cause problems later with your mail setup. First, you should check the hostname in <span class=system>/etc/hosts</span> and change it when necessary. The line should be: "IP Address - space - full hostname incl. domain - space - subdomain part". For our hostname web01.example.com, the file shall look like this:<div style="width:336px;float:left;margin:10px 15px 10px 0;background-color:#fff">
</div>
<pre class=command>nano /etc/hosts</pre>
<pre>127.0.0.1 localhost.localdomain   localhost<br># This line should be changed on every node to the correct servername:<br>127.0.1.1 web01.example.com web01<br># These lines are the same on every node:
10.0.64.12 panel.example.com panel<br>10.0.64.13 web01.example.com web01<br>10.0.64.14 mx1.example.com mx1<br>10.0.64.15 mx2.example.com mx2<br>10.0.64.16 ns1.example.com ns1<br>10.0.64.17 ns2.example.com ns2<br>10.0.64.18 webmail.example.com webmail

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters</pre>
<p>
<p>As you can see, we added the hostnames of our other servers aswell, so they can communicate over the internal network later.
<p>Then edit the /etc/hostname file:</p>
<p>It shall contain only the subdomain part, in our case:
<pre>web01</pre>
<p>Finally, reboot the server to apply the change:
<pre class=command><span>systemctl reboot</span></pre>
<p>Log in again and check if the hostname is correct now with these commands:
<pre class=command>hostname<br>hostname -f</pre>
<p>The output shall be like this:

  <pre class=system><a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="dcaeb3b3a89cabb9beeced">[email&#160;protected]</a>:~$ hostname<br>web01<br><a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="295b46465d695e4c4b1918">[email&#160;protected]</a>:~$ hostname -f<br>web01.example.com</pre>
<h3 id=-installing-ispconfig>3.2 Installing ISPConfig</h3>
<p>Now we can run the autoinstaller for all packages and ISPConfig:
<pre>wget -O - https://get.ispconfig.org | sh -s -- --no-mail --no-dns --interactive</pre>
<p>After some time, you will see:
<pre class=command>WARNING! This script will reconfigure your complete server!<br>It should be run on a freshly installed server and all current configuration that you have done will most likely be lost!<br>Type &#39;yes&#39; if you really want to continue:</pre>
<p>Answer "yes" and hit enter. The installer will now start.
<p>When the installation and configuration of the packages is done, the root password for MySQL on web01 will be shown. Write this down (along with the servername, to prevent any confusion later).
<p>Now we will have to answer some questions as we are using interactive mode. This is necessary as this server will be added to your multiserver setup.<span id=ezoic-pub-ad-placeholder-110 class=ezoic-adpicker-ad></span><span class="ezoic-ad box-4 box-4110 adtester-container adtester-container-110" data-ez-name=howtoforge_com-box-4><span id=div-gpt-ad-howtoforge_com-box-4-0 ezaw=580 ezah=400 style=position:relative;z-index:0;display:inline-block;padding:0;width:100%;max-width:1200px;margin-left:auto!important;margin-right:auto!important;min-height:400px;min-width:580px class=ezoic-ad><script data-cfasync="false" src="/cdn-cgi/scripts/5c5dd728/cloudflare-static/email-decode.min.js"></script><script data-ezscrex=false data-cfasync=false style=display:none>typeof __ez_fad_position!='undefined'&&__ez_fad_position('div-gpt-ad-howtoforge_com-box-4-0')</script></span></span>
<pre>[INFO] Installing ISPConfig3.<br>[INFO] Your MySQL root password is: kl3994aMsfkkeE<br><br><br>--------------------------------------------------------------------------------<br> _____ ___________   _____              __ _         ____<br>|_   _/  ___| ___ \ /  __ \            / _(_)       /__  \<br>  | | \ `--.| |_/ / | /  \/ ___  _ __ | |_ _  __ _    _/ /<br>  | |  `--. \  __/  | |    / _ \| &#39;_ \|  _| |/ _` |  |_ |<br> _| |_/\__/ / |     | \__/\ (_) | | | | | | | (_| | ___\ \<br> \___/\____/\_|      \____/\___/|_| |_|_| |_|\__, | \____/<br>                                              __/ |<br>                                             |___/ <br>--------------------------------------------------------------------------------<br><br><br>&gt;&gt; Initial configuration  <br><br>Operating System: Debian 10.0 (Buster) or compatible<br><br>    Following will be a few questions for primary configuration so be careful.<br>    Default values are in [brackets] and can be accepted with &lt;ENTER&gt;.<br>    Tap in &#34;quit&#34; (without the quotes) to stop the installer.<br><br><br>Select language (en,de) [en]: <span class=highlight>&lt;-- Hit enter</span><br><br>Installation mode (standard,expert) [standard]: <span class=highlight>&lt;-- expert</span><br><br>Full qualified hostname (FQDN) of the server, eg server1.domain.tld  [web01.example.com]: <span class=highlight>&lt;-- Hit Enter</span><br><br>MySQL server hostname [localhost]: <span class=highlight>&lt;-- Hit Enter</span><br><br>MySQL server port [3306]: <span class=highlight>&lt;-- Hit Enter</span><br><br>MySQL root username [root]: <span class=highlight>&lt;-- Hit Enter</span><br><br>MySQL root password []: <span class=highlight>&lt;-- Enter the MySQL password the script just gave you</span><br><br>MySQL database to create [dbispconfig]: <span class=highlight>&lt;-- Hit Enter</span><br><br>MySQL charset [utf8]: <span class=highlight>&lt;-- Hit Enter</span><br><br>The next two questions are about the internal ISPConfig database user and password.<br>It is recommended to accept the defaults which are &#39;ispconfig&#39; as username and a random password.<br>If you use a different password, use only numbers and chars for the password.<br><br>ISPConfig mysql database username [ispconfig]: <span class=highlight>&lt;-- Hit Enter</span><br><br>ISPConfig mysql database password [aakl203920459853sak20284204]: <span class=highlight>&lt;-- Hit Enter</span><br><br>Shall this server join an existing ISPConfig multiserver setup (y,n) [n]: <span class=highlight>&lt;-- y</span><br><br>MySQL master server hostname []: <span class=highlight>&lt;-- panel.example.com</span><br><br>MySQL master server port []: <span class=highlight>&lt;-- Hit Enter</span><br><br>MySQL master server root username [root]: <span class=highlight>&lt;-- Hit Enter</span><br><br>MySQL master server root password []: <span class=highlight>&lt;-- the password you gave the external root user on the master server.</span><br><br>MySQL master server database name [dbispconfig]: <span class=highlight>&lt;-- Hit Enter</span><br><br>Adding ISPConfig server record to database.<br><br>Configure Mail (y,n) [y]: <span class=highlight>&lt;-- n</span><br><br>Configuring Jailkit<br>Configuring Pureftpd<br>Configure DNS Server (y,n) [y]: <span class=highlight>&lt;-- n</span><br><br>The Web Server option has to be enabled when you want run a web server or when this node shall host the ISPConfig interface.<br>Configure Web Server (y,n) [y]: <span class=highlight>&lt;-- Hit Enter</span><br><br>Configuring Apache<br>Configuring vlogger<br>[WARN] autodetect for OpenVZ failed<br>Force configure OpenVZ (y,n) [n]: <span class=highlight>&lt;-- Hit Enter</span><br><br>Skipping OpenVZ<br><br>Configure Firewall Server (y,n) [y]: <span class=highlight>&lt;-- Hit Enter</span><br><br>Configuring Ubuntu Firewall<br>[WARN] autodetect for Metronome XMPP Server failed<br>Force configure Metronome XMPP Server (y,n) [n]: <span class=highlight>&lt;-- Hit Enter</span><br><br>Skipping Metronome XMPP Server<br><br>Configuring Fail2ban<br>Install ISPConfig Web Interface (y,n) [n]: <span class=highlight>&lt;-- Hit Enter</span><br><br>Do you want to create SSL certs for your server? (y,n) [y]: <span class=highlight>&lt;-- Hit Enter</span><br><br>Checking / creating certificate for web01.example.com<br>Using certificate path /etc/letsencrypt/live/web01.example.com<br>Using apache for certificate validation<br>Symlink ISPConfig SSL certs to Postfix? (y,n) [y]: <span class=highlight>&lt;-- Hit Enter</span><br><br>Symlink ISPConfig SSL certs to Pure-FTPd? Creating dhparam file may take some time. (y,n) [y]: <span class=highlight>&lt;-- Hit Enter</span><br><br>Generating DH parameters, 2048 bit long safe prime, generator 2<br>This is going to take a long time<br>......................+...........................................+...............<br>Configuring Apps vhost<br>Configuring DBServer<br>Installing ISPConfig crontab<br>Detect IP addresses<br>Restarting services ...<br>Installation completed.<br>[INFO] Adding php versions to ISPConfig.<br>[INFO] Checking all services are running.<br>[INFO] mysql: OK<br>[INFO] clamav-daemon: OK<br>[INFO] postfix: OK<br>[INFO] bind9: OK<br>[INFO] pureftpd: OK<br>[INFO] apache2: OK<br>[INFO] Installation ready.<br>[INFO] Your MySQL root password is: kl3994aMsfkkeE<br>[INFO] Warning: Please delete the log files in /tmp/ispconfig-ai/var/log/setup-* once you don&#39;t need them anymore because they contain your passwords!</pre>
<p><em>Note: if you want to redirect example.com/webmail to webmail.example.com, follow <a href=https://www.howtoforge.com/community/threads/redirect-webmail-to-webmail-example-com.86368/ target=_blank rel=noopener>this guide</a>.</em>
<p>To set this server as default for your websites and databases, log in to ISPConfig and go to System -> Main config. Select web01.example.com as default server.<em><br></em>
<h3 id=-setting-up-the-firewall>3.3 Setting up the firewall</h3>
<p>The last thing to do is to set up our firewall.
<p>Log in to the ISPConfig UI, and go to System -> Firewall. Then click "Add new firewall record".
<p>Make sure you select the correct server. For our webserver, we have to open the following ports:<span id=ezoic-pub-ad-placeholder-111 class=ezoic-adpicker-ad></span>
<p>TCP:
<pre>20,21,22,80,443</pre>
<p>No UDP ports have to be opened through the UI.
<p>We are also going to open port 3306, which is used for MySQL, but only from our local network for security reasons. To do so, run the following command from the CLI, after the change from the ISPConfig panel is propagated (when the red dot is gone):
<pre class=command>ufw allow from 10.0.64.0/24 to any port 3306 proto tcp</pre>
<p>Your webserver is now ready to use. In the next step, we will install the first mailserver.
</article>
<nav>
<h1>ISPConfig Perfect Multiserver setup on Ubuntu 20.04 and Debian 10 - Page 3</h1>
<div class=contributeEdit style=float:right;width:450px;max-width:100%>
<div id=tocContainer>
<h3>On this page</h3>
<ol class=toc>
<li><a href=#-installing-the-first-mailserver>4 Installing the first mailserver</a>
<ol>
<li><a href=#-configure-the-hostname>4.1 Configure the hostname</a>
<li><a href=#-installing-ispconfig>4.2 Installing ISPConfig</a>
<li><a href=#-setting-up-replication-for-dovecot>4.3 Setting up replication for Dovecot</a>
<li><a href=#-setting-up-synchronisation-for-rspamd>4.4 Setting up synchronisation for Rspamd</a>
<li><a href=#-securing-the-mailserver-with-a-valid-ssl-certificate>4.5 Securing the mailserver with a valid SSL certificate</a>
<li><a href=#-setting-up-the-firewall>4.6 Setting up the firewall</a>
</ol>
</ol>
</div>
</div>
<h2 id=-installing-the-first-mailserver>4 Installing the first mailserver</h2>
<p>Log in as root or run<span id=ezoic-pub-ad-placeholder-106 class=ezoic-adpicker-ad></span><span class="ezoic-ad box-3 box-3106 adtester-container adtester-container-106" data-ez-name=howtoforge_com-box-3><span id=div-gpt-ad-howtoforge_com-box-3-0 ezaw=336 ezah=280 style=position:relative;z-index:0;display:inline-block;padding:0;min-height:280px;min-width:336px class=ezoic-ad><script data-ezscrex=false data-cfasync=false style=display:none>typeof __ez_fad_position!='undefined'&&__ez_fad_position('div-gpt-ad-howtoforge_com-box-3-0')</script></span></span>
<pre class=command>su -</pre>
<p>to become root user on your server before you proceed. <strong>IMPORTANT</strong>: You must use 'su -' and not just 'su', otherwise your PATH variable is set wrong by Debian.<span id=ezoic-pub-ad-placeholder-121 class=ezoic-adpicker-ad></span><span class="ezoic-ad medrectangle-3 medrectangle-3121 adtester-container adtester-container-121" data-ez-name=howtoforge_com-medrectangle-3><span id=div-gpt-ad-howtoforge_com-medrectangle-3-0 ezaw=728 ezah=90 style=position:relative;z-index:0;display:inline-block;padding:0;min-height:90px;min-width:728px class=ezoic-ad><script data-ezscrex=false data-cfasync=false style=display:none>typeof __ez_fad_position!='undefined'&&__ez_fad_position('div-gpt-ad-howtoforge_com-medrectangle-3-0')</script></span></span>
<h3 id=-configure-the-hostname>4.1 Configure the hostname</h3>
<p>The hostname of your server should be a subdomain like "mx1.example.com". Do not use a domain name without a subdomain part like "example.com" as hostname as this will cause problems later with your mail setup. First, you should check the hostname in <span class=system>/etc/hosts</span> and change it when necessary. The line should be: "IP Address - space - full hostname incl. domain - space - subdomain part". For our hostname mx1.example.com, the file shall look like this:
<pre class=command>nano /etc/hosts</pre>
<pre>127.0.0.1 localhost.localdomain   localhost<br># This line should be changed on every node to the correct servername:<br>127.0.1.1 mx1.example.com mx1<br># These lines are the same on every node:
10.0.64.12 panel.example.com panel<br>10.0.64.13 web01.example.com web01<br>10.0.64.14 mx1.example.com mx1<br>10.0.64.15 mx2.example.com mx2<br>10.0.64.16 ns1.example.com ns1<br>10.0.64.17 ns2.example.com ns2<br>10.0.64.18 webmail.example.com webmail

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters</pre>
<p>As you can see, we added the hostnames of our other servers aswell, so they can communicate over the internal network later.<div style="width:336px;float:left;margin:10px 15px 10px 0;background-color:#fff">
</div>
<p>Then edit the /etc/hostname file:
<pre class=command>nano /etc/hostname</pre>
<p>It shall contain only the subdomain part, in our case:<span id=ezoic-pub-ad-placeholder-130 class=ezoic-adpicker-ad></span><span class="ezoic-ad link-h-large-1 link-h-large-1130 adtester-container adtester-container-130" data-ez-name=howtoforge_com-link-h-large-1><div id=howtoforge_com-link-h-large-1 class=ezo_link_unit_a><ins class=adsbygoogle style=display:inline-block;width:728px;height:15px data-ad-client=ca-pub-3043223216276099 data-ad-slot=2663697646 data-ad-channel=7741601259></ins><script>(adsbygoogle=window.adsbygoogle||[]).push({})</script></div></span>
<pre>mx1</pre>
<p>Finally, reboot the server to apply the change:
<pre class=command><span>systemctl reboot</span></pre>
<p>Log in again and check if the hostname is correct now with these commands:
<pre class=command>hostname<br>hostname -f</pre>
<p>The output shall be like this:
<pre class=system><a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="ff8d90908bbf9287ce">[email&#160;protected]</a>:~$ hostname<br>mx1<br><a href="/cdn-cgi/l/email-protection" class="__cf_email__" data-cfemail="65170a0a1125081d54">[email&#160;protected]</a>:~$ hostname -f<br>mx1.example.com</pre>
<h3 id=-installing-ispconfig>4.2 Installing ISPConfig</h3>
<p>Now we can run the autoinstaller for all packages and ISPConfig:<span id=ezoic-pub-ad-placeholder-108 class=ezoic-adpicker-ad></span><span class="ezoic-ad medrectangle-4 medrectangle-4108 adtester-container adtester-container-108" data-ez-name=howtoforge_com-medrectangle-4><span id=div-gpt-ad-howtoforge_com-medrectangle-4-0 ezaw=336 ezah=280 style=position:relative;z-index:0;display:inline-block;padding:0;width:100%;max-width:1200px;margin-left:auto!important;margin-right:auto!important;min-height:280px;min-width:336px class=ezoic-ad><script data-cfasync="false" src="/cdn-cgi/scripts/5c5dd728/cloudflare-static/email-decode.min.js"></script><script data-ezscrex=false data-cfasync=false style=display:none>typeof __ez_fad_position!='undefined'&&__ez_fad_position('div-gpt-ad-howtoforge_com-medrectangle-4-0')</script></span></span>
<pre>wget -O - https://get.ispconfig.org | sh -s -- --no-dns --no-roundcube --no-mailman --use-php=system --use-unbound --interactive</pre>
<p>After some time, you will see:
<pre class=command>WARNING! This script will reconfigure your complete server!<br>It should be run on a freshly installed server and all current configuration that you have done will most likely be lost!<br>Type &#39;yes&#39; if you really want to continue:</pre>
<p>Answer "yes" and hit enter. The installer will now start.
<p>When the installation and configuration of the packages is done, the root password for MySQL on mx1 will be shown. Write this down (along with the servername, to prevent any confusion later).
<p>Now we will have to answer some questions as we are using interactive mode. This is necessary as this server will be added to your multiserver setup.
<pre>[INFO] Installing ISPConfig3.<br>[INFO] Your MySQL root password is: kl3994aMsfkkeE<br><br><br>--------------------------------------------------------------------------------<br> _____ ___________   _____              __ _         ____<br>|_   _/  ___| ___ \ /  __ \            / _(_)       /__  \<br>  | | \ `--.| |_/ / | /  \/ ___  _ __ | |_ _  __ _    _/ /<br>  | |  `--. \  __/  | |    / _ \| &#39;_ \|  _| |/ _` |  |_ |<br> _| |_/\__/ / |     | \__/\ (_) | | | | | | | (_| | ___\ \<br> \___/\____/\_|      \____/\___/|_| |_|_| |_|\__, | \____/<br>                                              __/ |<br>                                             |___/ <br>--------------------------------------------------------------------------------<br><br><br>&gt;&gt; Initial configuration  <br><br>Operating System: Debian 10.0 (Buster) or compatible<br><br>    Following will be a few questions for primary configuration so be careful.<br>    Default values are in [brackets] and can be accepted with &lt;ENTER&gt;.<br>    Tap in &#34;quit&#34; (without the quotes) to stop the installer.<br><br><br>Select language (en,de) [en]: <span class=highlight>&lt;-- Hit enter</span><br><br>Installation mode (standard,expert) [standard]: <span class=highlight>&lt;-- expert</span><br><br>Full qualified hostname (FQDN) of the server, eg server1.domain.tld  [mx1.example.com]: <span class=highlight>&lt;-- Hit Enter</span><br><br>MySQL server hostname [localhost]: <span class=highlight>&lt;-- Hit Enter</span><br><br>MySQL server port [3306]: <span class=highlight>&lt;-- Hit Enter</span><br><br>MySQL root username [root]: <span class=highlight>&lt;-- Hit Enter</span><br><br>MySQL root password []: <span class=highlight>&lt;-- Enter the MySQL password the script just gave you</span><br><br>MySQL database to create [dbispconfig]: <span class=highlight>&lt;-- Hit Enter</span><br><br>MySQL charset [utf8]: <span class=highlight>&lt;-- Hit Enter</span><br><br>The next two questions are about the internal ISPConfig database user and password.<br>It is recommended to accept the defaults which are &#39;ispconfig&#39; as username and a random password.<br>If you use a different password, use only numbers and chars for the password.<br><br>ISPConfig mysql database username [ispconfig]: <span class=highlight>&lt;-- Hit Enter</span><br><br>ISPConfig mysql database password [aakl203920459853sak20284204]: <span class=highlight>&lt;-- Hit Enter</span><br><br>Shall this server join an existing ISPConfig multiserver setup (y,n) [n]: <span class=highlight>&lt;-- y</span><br><br>MySQL master server hostname []: <span class=highlight>&lt;-- panel.example.com</span><br><br>MySQL master server port []: <span class=highlight>&lt;-- Hit Enter</span><br><br>MySQL master server root username [root]: <span class=highlight>&lt;-- Hit Enter</span><br><br>MySQL master server root password []: <span class=highlight>&lt;-- the password you gave the external root user on the master server.</span><br><br>MySQL master server database name [dbispconfig]: <span class=highlight>&lt;-- Hit Enter</span><br><br>Adding ISPConfig server record to database.<br><br>Configure Mail (y,n) [y]: <span class=highlight>&lt;-- Hit enter<br></span><br>Configuring Postgrey<br>Configuring Postfix<br>Generating a RSA private key<br>......................................................................++++<br>....................++++<br>writing new private key to &#39;smtpd.key&#39;<br>-----<br>You are about to be asked to enter information that will be incorporated<br>into your certificate request.<br>What you are about to enter is what is called a Distinguished Name or a DN.<br>There are quite a few fields but you can leave some blank<br>For some fields there will be a default value,<br>If you enter &#39;.&#39;, the field will be left blank.<br>-----<br>Country Name (2 letter code) [AU]: <span class=highlight>&lt;-- Hit enter</span><br>State or Province Name (full name) [Some-State]: <span class=highlight>&lt;-- Hit enter</span><br>Locality Name (eg, city) []: <span class=highlight>&lt;-- Hit enter</span><br>Organization Name (eg, company) [Internet Widgits Pty Ltd]: <span class=highlight>&lt;-- Hit enter</span><br>Organizational Unit Name (eg, section) []: <span class=highlight>&lt;-- Hit enter</span><br>Common Name (e.g. server FQDN or YOUR name) []: <span class=highlight>&lt;-- Hit enter</span><br>Email Address []: &lt;-- Hit enter<br>[WARN] autodetect for Mailman failed<br>Force configure Mailman (y,n) [n]: <span class=highlight>&lt;-- Hit enter</span><br><br>Skipping Mailman<br><br>Configuring Dovecot<br>Creating new DHParams file, this takes several minutes. Do not interrupt the script.<br>Configuring Spamassassin<br>[WARN] autodetect for Amavisd failed<br>Force configure Amavisd (y,n) [n]: <span class=highlight>&lt;-- Hit enter</span><br><br>Skipping Amavisd<br><br>Configuring Rspamd<br>Configuring Getmail<br>Configuring Jailkit<br>Configuring Pureftpd<br>Configure DNS Server (y,n) [y]: <span class=highlight>&lt;-- n</span><br><br>The Web Server option has to be enabled when you want run a web server or when this node shall host the ISPConfig interface.<br>Configure Web Server (y,n) [y]: <span class=highlight>&lt;-- Hit enter</span><br><br>Configuring Apache<br>Configuring vlogger<br>[WARN] autodetect for OpenVZ failed<br>Force configure OpenVZ (y,n) [n]: <span class=highlight>&lt;-- Hit Enter</span><br><br>Skipping OpenVZ<br><br>Configure Firewall Server (y,n) [y]: <span class=highlight>&lt;-- Hit Enter</span><br><br>Configuring Ubuntu Firewall<br>[WARN] autodetect for Metronome XMPP Server failed<br>Force configure Metronome XMPP Server (y,n) [n]: <span class=highlight>&lt;-- Hit Enter</span><br><br>Skipping Metronome XMPP Server<br><br>Configuring Fail2ban<br>Install ISPConfig Web Interface (y,n) [n]: <span class=highlight>&lt;-- Hit Enter</span><br><br>Do you want to create SSL certs for your server? (y,n) [y]: <span class=highlight>&lt;-- Hit Enter</span><br><br>Checking / creating certificate for mx1.example.com<br>Using certificate path /etc/letsencrypt/live/mx1.example.com<br>Using apache for certificate validation<br>Symlink ISPConfig SSL certs to Postfix? (y,n) [y]: <span class=highlight>&lt;-- Hit Enter</span><br><br>Symlink ISPConfig SSL certs to Pure-FTPd? Creating dhparam file may take some time. (y,n) [y]: <span class=highlight>&lt;-- Hit Enter</span><br><br>Generating DH parameters, 2048 bit long safe prime, generator 2<br>This is going to take a long time<br>......................+...........................................+...............<br>Configuring Apps vhost<br>Configuring DBServer<br>Installing ISPConfig crontab<br>no crontab for getmail<br>Detect IP addresses<br>Restarting services ...<br>Installation completed.<br>[INFO] Adding php versions to ISPConfig.<br>[INFO] Checking all services are running.<br>[INFO] mysql: OK<br>[INFO] clamav-daemon: OK<br>[INFO] postfix: OK<br>[INFO] bind9: OK<br>[INFO] pureftpd: OK<br>[INFO] apache2: OK<br>[INFO] rspamd: OK<br>[INFO] redis-server: OK<br>[INFO] dovecot: OK<br>[INFO] Installation ready.<br>[INFO] Your MySQL root password is: kl3994aMsfkkeE<br>[INFO] Warning: Please delete the log files in /tmp/ispconfig-ai/var/log/setup-* once you don&#39;t need them anymore because they contain your passwords!</pre>
<h3 id=-setting-up-replication-for-dovecot>4.3 Setting up replication for Dovecot</h3>
<p>We are going to use dsync to synchronise the emails between mx1 and mx2, for redundancy.<span id=ezoic-pub-ad-placeholder-110 class=ezoic-adpicker-ad></span><span class="ezoic-ad box-4 box-4110 adtester-container adtester-container-110" data-ez-name=howtoforge_com-box-4><span id=div-gpt-ad-howtoforge_com-box-4-0 ezaw=580 ezah=400 style=position:relative;z-index:0;display:inline-block;padding:0;width:100%;max-width:1200px;margin-left:auto!important;margin-right:auto!important;min-height:400px;min-width:580px class=ezoic-ad><script data-ezscrex=false data-cfasync=false style=display:none>typeof __ez_fad_position!='undefined'&&__ez_fad_position('div-gpt-ad-howtoforge_com-box-4-0')</script></span></span>
<p>The custom settings for dovecot have to be stored in /usr/local/ispconfig/server/conf-custom/install/dovecot_custom.conf.master so you don't lose them when updating ISPConfig.
<pre class=command>nano /usr/local/ispconfig/server/conf-custom/install/dovecot_custom.conf.master</pre>
<p>Put the following in there:
<pre>protocol imap {<br>  mail_plugins = $mail_plugins quota imap_quota notify replication<br>}<br>protocol pop3 {<br>  mail_plugins = $mail_plugins quota notify replication<br>}<br>protocol lda {<br>  mail_plugins = $mail_plugins sieve quota notify replication<br>}<br>protocol lmtp {<br>  mail_plugins = $mail_plugins sieve quota notify replication<br>}<br>doveadm_password = /(:&amp;p-J:4e%?\@Q-;VSE#3Dmfm[fVK&amp;r-mx1<br>doveadm_port = 12345<br>replication_max_conns = 50<br><br>ssl_client_ca_dir = /etc/ssl/certs<br><br># Replicator process should be started at startup, so it can start replicating users immediately:<br>service replicator {<br>  process_min_avail = 1<br>}<br><br># The mail processes need to have access to the replication-notify fifo and socket.<br>service aggregator {<br>    fifo_listener replication-notify-fifo {<br>        user = vmail<br>        mode = 0666<br>    }<br><br>    unix_listener replication-notify {<br>        user = vmail<br>        mode = 0666<br>    }<br>}<br><br># Enable doveadm replicator commands<br>service replicator {<br>    unix_listener replicator-doveadm {<br>        mode = 0666<br>    }<br>}<br><br># Create a listener for doveadm-server<br>service doveadm {<br>    user = vmail<br>    inet_listener {<br>        port = 12345<br>        ssl = yes<br>    }<br>}<br>service config {<br>    unix_listener config {<br>        user = vmail<br>    }<br>}<br><br>plugin {<br>    mail_replica = tcps:mx2.example.com<br>}</pre>
<p>Replace the doveadm_password with your own password - make it a long and random string for security reasons.
<p>Replace
<pre>mail_replica = tcps:mx2.example.com</pre>
<p>with your own hostname of mx2.<span id=ezoic-pub-ad-placeholder-111 class=ezoic-adpicker-ad></span>
<p>To apply these changes, copy the file to the dovecot folder and restart dovecot:
<pre class=command>cp /usr/local/ispconfig/server/conf-custom/install/dovecot_custom.conf.master /etc/dovecot/conf.d/99-ispconfig-custom-config.conf<br>systemctl restart dovecot</pre>
<h3 id=-setting-up-synchronisation-for-rspamd>4.4 Setting up synchronisation for Rspamd</h3>
<p>Rspamd has a nice web UI. We are going to set up our servers as neighbours so you can view the settings, stats, and other data in one panel.
<pre class=command>nano /etc/rspamd/local.d/options.inc</pre>
<p>Add these lines:
<pre>neighbours {<br>    mx1 {<br>        host = &#34;https://<span class=highlight>mx1.example.com</span>:443&#34;;<br>        path = &#34;/rspamd/&#34;;<br>    }<br>    mx2 { <br>        host = &#34;https://<span class=highlight>mx2.example.com</span>:443&#34;;<br>        path = &#34;/rspamd/&#34;;<br>    }<br>}</pre>
<p>Replace the hostnames with the correct hostnames for your main and secondary nameserver.
<p>Now we are going to add a website for this server in ISPConfig. Go to Sites -> Add new website and add the website "mx1.example.com". Make sure you create this on the correct server (mx1.example.com). Disable auto-subdomain, and enable SSL + Let's Encrypt. Then, go to options, and paste this in the Apache directives section:<span id=ezoic-pub-ad-placeholder-112 class=ezoic-adpicker-ad></span>
<pre>RewriteEngine On<br>RewriteRule ^/rspamd$ /rspamd/ [R,L]<br>RewriteRule ^/rspamd/(.*) http://127.0.0.1:11334/$1 [P]<br>Header set Access-Control-Allow-Origin https://<span class=highlight>mx2.example.com</span></pre>
<p>Replace mx2.example.com with the hostname of the secondary nameserver.
<p>Lastly, we have to enable some Apache modules and restart our services:
<pre class=command>a2enmod proxy_balancer proxy_http<br>systemctl restart rspamd<br>systemctl restart apache2</pre>
<h3 id=-securing-the-mailserver-with-a-valid-ssl-certificate>4.5 Securing the mailserver with a valid SSL certificate</h3>
<p>For a working DSYNC and Roundcube setup, you need to have a valid SSL certificiate in place for the used hostnames. This certificate can not be self signed. If you are going to use additional hostnames for this server, like imap.example.com and smtp.example.com, or if the installer could not create a valid certificate when installing, follow this guide to set up a valid certificate for your mailserver: <a href=https://www.howtoforge.com/securing-your-ispconfig-3-managed-mailserver-with-a-valid-lets-encrypt-certificate/ target=_blank rel=noopener>https://www.howtoforge.com/securing-your-ispconfig-3-managed-mailserver-with-a-valid-lets-encrypt-certificate/</a>
<h3 id=-setting-up-the-firewall>4.6 Setting up the firewall</h3>
<p>The last thing to do is to set up our firewall.
<p>Log in to the ISPConfig UI, and go to System -> Firewall. Then click "Add new firewall record".
<p>Make sure you select the correct server. For our mailserver, we have to open the following ports:
<p>TCP:
<pre>22,25,80,110,143,443,465,587,993,995</pre>
<p>No UDP ports have to be opened through the UI.
<p>We are also going to open port 3306, which is used for MySQL, and port 12345, which is used for dsync, but only from our local network for security reasons. To do so, run the following command from the CLI, after the change from the ISPConfig panel is propagated (when the red dot is gone):
<pre class=command>ufw allow from 10.0.64.0/24 to any port 3306 proto tcp<br>ufw allow from 10.0.64.0/24 to any port 12345 proto tcp</pre>
