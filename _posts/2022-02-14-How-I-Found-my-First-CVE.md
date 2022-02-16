---
title: "How I found my first CVE with Cross-Site Scripting(XSS)"
date: 2022-02-13
categories: Cybersecurity
---

As 2022 started, I set as a goal for myself to start exploring web security, and bug bounties programs. I always thought that getting a CVE was something only experienced security researchers, hardened with tons of experience, would be able to find these rare eleven character labels. However, as it turns out there are tons of low hanging fruit that are vulnerable to common bugs, and can still impact the security or companies and individuals around the globe.
<br>
<h1>The vulnerable world of WordPress plugins</h1>
WordPress is a useful service that is used around the world to host blogs and e-commerce sites among other things. WordPress instances can be expanded upon with plugins that allow for new implemented functionalities.
<h1>Finding a low hanging fruit</h1>
If this is your first time looking for bugs in a program, the first place you have to go to is the WordPress plugin store. Here you will find a lot of projects, some monetized some free, that may be vulnerable.
I suggest you spin up a docker instance of Wordress on your own computer in order to quickly be able to experiment on your own homelab.
<h1>Setting up WordPress Docker on your own machine</h1>
install docker with your package manager
blabla apt install docker.io
To set up docker create this yaml file
bla bla bla
then start the instance with this command
bla bla bla
then navigate to localhost:8000 on a web browser in order to start setting up your web page.
<h1>Installing a plug in</h1>
Wordpress plugins are usually easy to install and uninstall, and the source code is freely available for anyone to inspect.
I found the Flexi - Guest Submit plugin, which at first I thought could be vulnerable to a Local File Inclusion vulnerability due to the nature of the app,
but ended up being vulnerable to a Cross-Site Scripting.
<h1>How XSS works</h1>
Reflected Cross-Site Scripting is a vulnerability in web apps where an user input in not well sanitized, and javascript code can be ran on the client.
That compromised page can then be used for social engineering attacks such as cookie stealing or malware download.
In summary, if you input something in a field, and you get back your input on the returned page, it might be vulnerable to XSS.
<h1>Flexi guest submit app</h1>
The flexi Guest submit is a plugin that allows the creation of a portal to submit media files, which can then be reflected in a nicely arranged portfolio.
The app contains a user-dashboard accessible at loclhost:8000/user-dashboard/
This dashboard contains a search field that allows to browse the users who posted media files.
Using a common payload, such as <img src=1 onerror=alert(1)> we get an alert box!
the final XSS url ends up being localhost:8000/user-dashboard/?search=keyword:<img%20src=1%20onerror=alert(1)>
The optional next step, in order to help the developper, would be to search through the php code for the reference to the vulnerable search field, using a code editor with Ctrl+ F, or using grep.
Here the file \includes\user_dashboard\class-flexi-user-dashboard.php contains the input field that accepts the unsanitized user input.
<h1>Making a report</h1>
In order to make a valuable report, it is important to submit steps to replicate the issue, mitigations, and impact of the vulnerability. Once you have written a report, you can submit it a wp-scan,
which will verify the issue, and assign a CVE if it is confirmed.
