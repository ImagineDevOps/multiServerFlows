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
<p>In the next step, we will install the webserver.<span id=ezoic-pub-ad-placeholder-113 class=ezoic-adpicker-ad></span>
</article>
<nav>
<div id=articlePrevNextContainer>
<div id=articlePrevLink></div>
<div id=articleNextLink><a href=/tutorial/ispconfig-multiserver-setup-debian-ubuntu/2/>Next >></a></div>
</div>
</nav>
<div style=height:30px;margin-top:20px>
<p style=text-align:right><img src="data:image/svg+xml,%3Csvg xmlns=%22http://www.w3.org/2000/svg%22 width=%2216%22 height=%2216%22%3E%3C/svg%3E" ezimgfmt="rs rscb5 src ng ngcb5" class=ezlazyload data-ezsrc=/images/pdficon_small.png> <a class=pdf-btn href=javascript:void(0);>view as pdf</a> | <img src="data:image/svg+xml,%3Csvg xmlns=%22http://www.w3.org/2000/svg%22 width=%2216%22 height=%2216%22%3E%3C/svg%3E" class=ezlazyload data-ezsrc=/images/print.gif> <a class=print-btn href=javascript:void(0);>print</a>
</div>
<div style=height:30px;margin-top:20px>
<div style=float:left><b>Share this page:</b></div>
<p style=text-align:center>
<a href="https://www.facebook.com/sharer.php?u=https%3A%2F%2Fwww.howtoforge.com%2Ftutorial%2Fispconfig-multiserver-setup-debian-ubuntu%2F" target=_blank rel=nofollow><img src="data:image/svg+xml,%3Csvg xmlns=%22http://www.w3.org/2000/svg%22 width=%22119%22 height=%2225%22%3E%3C/svg%3E" height=20 ezimgfmt="rs rscb5 src ng ngcb5" class=ezlazyload data-ezsrc=/images/socialmedia/btn-recommend.png></a>
<a href="https://twitter.com/intent/tweet?url=https%3A%2F%2Fwww.howtoforge.com%2Ftutorial%2Fispconfig-multiserver-setup-debian-ubuntu%2F&text=ISPConfig+Perfect+Multiserver+setup+on+Ubuntu+20.04+and+Debian+10&via=howtoforgecom&related=howtoforgecom" target=_blank rel=nofollow><img src="data:image/svg+xml,%3Csvg xmlns=%22http://www.w3.org/2000/svg%22 width=%2275%22 height=%2225%22%3E%3C/svg%3E" height=20 ezimgfmt="rs rscb5 src ng ngcb5" class=ezlazyload data-ezsrc=/images/socialmedia/btn-tweet.png></a>
<a href=https://twitter.com/howtoforgecom/ target=_blank rel=nofollow><img src="data:image/svg+xml,%3Csvg xmlns=%22http://www.w3.org/2000/svg%22 width=%2278%22 height=%2225%22%3E%3C/svg%3E" height=20 ezimgfmt="rs rscb5 src ng ngcb5" class=ezlazyload data-ezsrc=/images/socialmedia/btn-follow.png></a>
<a href="https://plus.google.com/share?url=https%3A%2F%2Fwww.howtoforge.com%2Ftutorial%2Fispconfig-multiserver-setup-debian-ubuntu%2F" target=_blank rel=nofollow><img src="data:image/svg+xml,%3Csvg xmlns=%22http://www.w3.org/2000/svg%22 width=%2240%22 height=%2225%22%3E%3C/svg%3E" height=20 ezimgfmt="rs rscb5 src ng ngcb5" class=ezlazyload data-ezsrc=/images/socialmedia/btn-gplus.png></a>
</div>
</div>
<div id=htfContentPagePaging>
<h2>Sub pages</h2>
<nav>
<ul>
<li><a href=/tutorial/ispconfig-multiserver-setup-debian-ubuntu/><b>ISPConfig Perfect Multiserver setup on Ubuntu 20.04 and Debian 10</b></a><li><a href=/tutorial/ispconfig-multiserver-setup-debian-ubuntu/2/>ISPConfig Perfect Multiserver setup on Ubuntu 20.04 and Debian 10 - Page 2</a><li><a href=/tutorial/ispconfig-multiserver-setup-debian-ubuntu/3/>ISPConfig Perfect Multiserver setup on Ubuntu 20.04 and Debian 10 - Page 3</a><li><a href=/tutorial/ispconfig-multiserver-setup-debian-ubuntu/4/>ISPConfig Perfect Multiserver setup on Ubuntu 20.04 and Debian 10 - Page 4</a><li><a href=/tutorial/ispconfig-multiserver-setup-debian-ubuntu/5/>ISPConfig Perfect Multiserver setup on Ubuntu 20.04 and Debian 10 - Page 5</a><li><a href=/tutorial/ispconfig-multiserver-setup-debian-ubuntu/6/>ISPConfig Perfect Multiserver setup on Ubuntu 20.04 and Debian 10 - Page 6</a><li><a href=/tutorial/ispconfig-multiserver-setup-debian-ubuntu/7/>ISPConfig Perfect Multiserver setup on Ubuntu 20.04 and Debian 10 - Page 7</a><li><a href=/tutorial/ispconfig-multiserver-setup-debian-ubuntu/8/>ISPConfig Perfect Multiserver setup on Ubuntu 20.04 and Debian 10 - Page 8</a> </ul>
</nav>
</div>
<div id=htfContentPagePaging>
<h2>Suggested articles</h2>
</div>
<div id=htfContentPageComment><span id=ezoic-pub-ad-placeholder-118 class=ezoic-adpicker-ad></span>
<a name=comments></a>
<h2>1 Comment(s)</h2>
<div id=commentform>
<div class=commentContainerHead style=font-size:18px>Add comment</div>
<div class=commentContainer>
<form method=post action=/tutorial/ispconfig-multiserver-setup-debian-ubuntu/#comments>
<input type=hidden name=reply_to id=reply-to-comment value=0>
<div id=commentFormDetail>
<div style=width:48%;float:left;margin-right:3%>
<label for=commentname>Name *</label>
<input id=commentname name=commentname value>
</div>
<div style=width:48%;float:left>
<label for=commentname>Email *</label>
<input id=commentemail name=commentemail value>
</div>
<div style=clear:both></div>
</div>
<textarea id=commentedit name=commentedit style=width:auto></textarea>
<div>
<div style=float:left>
<div class=g-recaptcha data-sitekey=6LdX9fUSAAAAALWwVoPdeXScMSketz4npF-0QcQ1></div>
<script data-cfasync="false" src="/cdn-cgi/scripts/5c5dd728/cloudflare-static/email-decode.min.js"></script><script type=text/ez-screx src="https://www.howtoforge.com/ezossp/https/www.google.com/recaptcha/api.js?hl=en&screx=1&sxcb=5a"></script>
</div>
<div style=margin-top:10px;margin-bottom:10px>
<input type=submit name=submit class="button primary" style=float:right value="Submit comment">
</div>
<div style=clear:both></div>
</div>
<script type=text/ez-screx>tinymce.init({selector:"textarea#commentedit",theme:"modern",height:100,apply_source_formatting:!0,remove_linebreaks:!1,menubar:!1,plugins:["link"],content_css:"https://www.howtoforge.com/themes/howtoforge/css/custom.css",toolbar:"undo redo | bold italic link"})</script>
</form>
</div>
</div>
<h3>Comments</h3>
<div class="commentContainerHead commentLevel0">
<div class=floatLeft><b>By:</b> Mario <b>at:</b> <i>2021-04-18 03:27:46</i></div>
<div class=floatRight><a href=# data-comment=32510 class=reply-to>Reply</a>  </div>
<div class=clearBoth></div>
</div>
<div id=comment32510 class="commentContainer commentLevel0">
<p><p>Excelent document ... Thanks ...<p>
</div>
</div>
<div class=breadBoxBottom>
<nav>
<fieldset class=breadcrumb>
<span class=crumbs>
<span class="crust homeCrumb" itemscope itemtype=http://data-vocabulary.org/Breadcrumb>
<a href=/ class=crumb rel=up itemprop=url><span itemprop=title>Home</span></a>
<span class=arrow><span></span></span>
</span>
<span class="crust selectedTabCrumb" itemscope itemtype=http://data-vocabulary.org/Breadcrumb>
<a href=/tutorial/ispconfig-multiserver-setup-debian-ubuntu/ class=crumb rel=up itemprop=url><span itemprop=title>ISPConfig Perfect Multiserver setup on Ubuntu 20.04 and Debian 10</span></a>
<span class=arrow><span>></span></span>
</span>
</span>
</fieldset>
</nav>
</div>
<script type=text/ez-screx>$(document).on('click','a.reply-to',function(d){var c,a,b;if(d.preventDefault(),c=$(this),a=c.attr('data-comment'),console.log(a),!a)return this;$('#commentform').insertAfter('#comment'+a),$('#reply-to-comment').val(a),b='commentedit',tinyMCE.get(b).remove(),tinyMCE.execCommand("mceAddEditor",!1,b),tinyMCE.activeEditor.focus()})</script>
<div style=margin-top:10px;margin-bottom:10px;text-align:center>
<center>
</center>
</div>
<form action="/community/login/login?redirect=/tutorial/ispconfig-multiserver-setup-debian-ubuntu/" method=post class="xenForm eAuth" id=login style=display:none>
<ul id=eAuthUnit>
<li><a href="/community/register/facebook?reg=1" class=fbLogin tabindex=110><span>Log in with Facebook</span></a>
<li><a href="/community/register/twitter?reg=1" class=twitterLogin tabindex=110><span>Log in with Twitter</span></a>
<li><span class="googleLogin GoogleLogin JsOnly" tabindex=110 data-client-id=198773272284-b6tfk0c28bje81p774prrcrkq9jb3o44.apps.googleusercontent.com data-redirect-url="/community/register/google?code=__CODE__&csrf=hVMXbW_3rnrmq4Em"><span>Log in with Google</span></span>
</ul>
<div class=ctrlWrapper>
<dl class=ctrlUnit>
<dt><label for=LoginControl>Your name or email address:</label>
<dd><input name=login id=LoginControl class=textCtrl tabindex=101>
</dl>
<dl class=ctrlUnit>
<dt>
<label for=ctrl_password>Do you already have an account?</label>
<dd>
<ul>
<li><label for=ctrl_not_registered><input type=radio name=register value=1 id=ctrl_not_registered tabindex=105>
No, create an account now.</label>
<li><label for=ctrl_registered><input type=radio name=register value=0 id=ctrl_registered tabindex=105 checked class=Disabler>
Yes, my password is:</label>
<li id=ctrl_registered_Disabler>
<input type=password name=password class=textCtrl id=ctrl_password tabindex=102>
<div class=lostPassword><a href=/community/lost-password/ tabindex=106>Forgot your password?</a></div>
</ul>
</dl>
<dl class="ctrlUnit submitUnit">
<dt>
<dd>
<input type=submit class="button primary" value="Log in" tabindex=104 data-loginphrase="Log in" data-signupphrase="Sign up">
<label for=ctrl_remember class=rememberPassword><input type=checkbox name=remember value=1 id=ctrl_remember tabindex=103> Stay logged in</label>
</dl>
</div>
<input type=hidden name=cookie_check value=1>
<input type=hidden name=redirect value=/community/>
<input type=hidden name=_xfToken>
</form>
</div>
</div>
<aside>
<div class=sidebar><span id=ezoic-pub-ad-placeholder-101 class=ezoic-adpicker-ad></span><span class="ezoic-ad box-1 box-1101 adtester-container adtester-container-101 ezoic-ad-adaptive" data-ez-name=howtoforge_com-box-1><span class="ezoic-ad box-1 box-1-multi-101 adtester-container adtester-container-101" data-ez-name=howtoforge_com-box-1><span id=div-gpt-ad-howtoforge_com-box-1-0 ezaw=300 ezah=262 style=position:relative;z-index:0;display:inline-block;padding:0;min-height:262px;min-width:300px class=ezoic-ad><script data-ezscrex=false data-cfasync=false style=display:none>typeof __ez_fad_position!='undefined'&&__ez_fad_position('div-gpt-ad-howtoforge_com-box-1-0')</script></span></span><span class="ezoic-ad box-1 box-1-multi-101 adtester-container adtester-container-101" data-ez-name=howtoforge_com-box-1><span id=div-gpt-ad-howtoforge_com-box-1-0_1 ezaw=300 ezah=262 style=position:relative;z-index:0;display:inline-block;padding:0;min-height:262px;min-width:300px class=ezoic-ad><script data-ezscrex=false data-cfasync=false style=display:none>typeof __ez_fad_position!='undefined'&&__ez_fad_position('div-gpt-ad-howtoforge_com-box-1-0_1')</script></span></span><span class="ezoic-ad box-1 box-1-multi-101 adtester-container adtester-container-101" data-ez-name=howtoforge_com-box-1><span id=div-gpt-ad-howtoforge_com-box-1-0_2 ezaw=300 ezah=262 style=position:relative;z-index:0;display:inline-block;padding:0;min-height:262px;min-width:300px class=ezoic-ad><script data-ezscrex=false data-cfasync=false style=display:none>typeof __ez_fad_position!='undefined'&&__ez_fad_position('div-gpt-ad-howtoforge_com-box-1-0_2')</script></span></span><span class="ezoic-ad box-1 box-1-multi-101 adtester-container adtester-container-101" data-ez-name=howtoforge_com-box-1><span id=div-gpt-ad-howtoforge_com-box-1-0_3 ezaw=300 ezah=262 style=position:relative;z-index:0;display:inline-block;padding:0;min-height:262px;min-width:300px class=ezoic-ad><script data-ezscrex=false data-cfasync=false style=display:none>typeof __ez_fad_position!='undefined'&&__ez_fad_position('div-gpt-ad-howtoforge_com-box-1-0_3')</script></span></span><style>.box-1-multi-101{border:none!important;display:block!important;float:none;line-height:0;margin-bottom:10px!important;margin-left:0!important;margin-right:0!important;margin-top:8px!important;min-height:250px;min-width:300px;padding:0;text-align:center!important}</style></span>
<div class="section loginButton">
<div class=secondaryContent>
<label for=LoginControl id=SignupButton><a href=/community/login/ class=inner>Sign up now!</a></label>
</div>
</div>
