---
title: "How I found my first CVE with Cross-Site Scripting(XSS)"
author: "Felipe Tapia Sasot"
date: 2022-02-13
categories: Cybersecurity
tags: CVE-2022-0449
---
<p>
As the new year started, I set as a goal for myself to start exploring web security and bug bounties programs. I always thought that getting a CVE was something only experienced security researchers, hardened with tons of experience, would be able to find bugs with a broad impact. However, as it turns out there are tons of low hanging fruit that are vulnerable to common bugs and can still impact the security or companies and individuals around the globe.</p>
<br>
<h1>The vulnerable world of WordPress plugins</h1>
<p>
WordPress is a useful service that is used around the world to host blogs and e-commerce sites among other things. WordPress instances can be expanded upon with plugins that allow for new implemented functionalities.</p><br>
<h1>Finding a low hanging fruit</h1>
<p>
If this is your first time looking for bugs in a program, the first place you must go to is the WordPress plugin store. Here you will find a lot of projects, some monetized some free, that may be vulnerable. It is unlikely that you will find any 0-day that will turn the internet upside down, but you can find bugs that will give you the confidence of pursuing your bug bounty journey. I suggest you spin up a docker instance of WordPress on your own computer to quickly be able to experiment on your own home lab.</p><br>
<h1>Setting up WordPress Docker on your own machine</h1>
<p> First, install docker with your package manager, here I am using Debian 11:</p><br>
```
apt install docker.io
```
To set up docker create this yaml file:
```
nano docker-compose.yaml
```
and insert this content:
```
version: "3.9"

services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: somewordpress
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress

  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    volumes:
      - wordpress_data:/var/www/html
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
volumes:
  db_data: {}
  wordpress_data: {}
```
then start the instance with this command
```
docker-compose up -d
```
<p>then navigate to localhost:8000 on a web browser in order to start setting up your web page.</p><br>
<h1>Installing a plug in</h1>
<p>WordPress plugins are usually easy to install and uninstall, and the source code is freely available for anyone to inspect.</p><br>

<p>
I found the Flexi - Guest Submit plugin, which at first I thought could be vulnerable to a Local File Inclusion vulnerability due to the nature of the app,
however, it ended up being vulnerable to a Cross-Site Scripting.
</p><br>
<h1>How XSS works</h1>
<p>
Reflected Cross-Site Scripting is a vulnerability in web apps where an user input in not well sanitized, and JavaScript code can be ran on the client.
That compromised page can then be used for social engineering attacks such as cookie stealing or malware download.
In summary, if you input something in a field, and you get back your input on the returned page, it might be vulnerable to XSS.</p><br>
<h1>Flexi guest submit app</h1>
The Flexi Guest submit is a plugin that allows the creation of a portal to submit media files, which can then be reflected in a nicely arranged portfolio.
<br>
The app contains a user-dashboard accessible at loclhost:8000/user-dashboard/
This dashboard contains a search field that allows to browse the users who posted media files.
Using a common payload, such as ```<img src=1 onerror=alert(1)>``` we get an alert box!
<img src="{{site.url}}/images/xssalert.jpg" style="display: block; margin: auto;" />
<br>
The final XSS url ends up being:
http://localhost:8000/user-dashboard/?search=keyword:<img%20src=1%20onerror=alert(1)>
There are many XSS payloads to manually test from in this repository:
https://github.com/payloadbox/xss-payload-list
<br>
The optional next step, in order to help the developer, would be to search through the PHP code for the reference to the vulnerable search field, using a code editor with Ctrl+ F, or using grep.
Here the file "\includes\user_dashboard\class-flexi-user-dashboard.php" contains the input field that accepts the unsanitized user input.
<img src="{{site.url}}/images/xssinputf.jpg" style="display: block; margin: auto;" />
<h1>Making a report</h1>
In order to make a valuable report, it is important to submit steps to replicate the issue, mitigations, and impact of the vulnerability. Once you have written a report, you can submit it to wp-scan.
https://wpscan.com/submit
They will verify the issue, and assign a CVE if it is confirmed.
<br>
<p>
Link to plugin:<a href="https://wordpress.org/plugins/flexi/">https://wordpress.org/plugins/flexi/</a><br>
Link to plugin source code:<a href="https://plugins.trac.wordpress.org/browser/flexi/">https://plugins.trac.wordpress.org/browser/flexi/</a><br>
Link to WPScan report:<a href="https://wpscan.com/vulnerability/3cc1bb3c-e124-43d3-bc84-a493561a1387"> https://wpscan.com/vulnerability/3cc1bb3c-e124-43d3-bc84-a493561a1387</a><br>
Link to Mitre CVE: <a href="https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-0449">https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2022-0449</a></p>
