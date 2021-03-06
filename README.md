# web-knock: allows only you to connect to your server

```
          ____ ____  _   _   _                       
  _   _  / ___/ ___|| | | | | |  _ __   __ _ ___ ___ 
 | | | | \___ \___ \| |_| | | | | '_ \ / _` / __/ __|
 | |_| |  ___) |__) |  _  | |_| | |_) | (_| \__ \__ \
  \__,_| |____/____/|_| |_| (_) | .__/ \__,_|___/___/  web-knock
                                |_|                  
```

## What is it

web-knock is a simple and free alternative to the practice of protecting admin port of your server with a paid fixed IP or dynamic DNS.

Even if you don't have a static IP or dynamic DNS, with this small script your server firewall will keep some ports on your server opened only to you. This can help to avoid brute-force attacks, by allowing access to authentication only to you.

**Note: this script is not an authentication method.** So please do not consider it a substitute to public key authentication, or PAM module for TOTP/2FA authentication, nor a substitute for brute force protection like fail2ban/SSHGuard. You are still encouraged to apply all the required security measures to protect your server.

## Use case

Protection standard "fail2ban" effectively blocks brute-force attacks, but in the process it also creates huge jails of IPs for large botnets with different IPs, by reading and updating this database of banned addresses at every connection. web-knock takes a different approach by allowing **only your IP** to connect to your VPS.

Static IP and dynamic DNS are the best solution for this, but they are not free (even Entrydns and Dnsexit services, which are free, allow only a limited number of requests per month), and *iptables* doesn't even supports Dynamic DNS addresses as firewall rule.

And even if you have a fixed IP at home, or already have a Dynamic DNS, **what if you are on the go with your mobile phone?**

With web-knock you can quickly add yourself to a whitelist, similarly as port knocking does, and without needing any specific client or extra configuration. Only a web browser is requried.

## How it works

It is simple as just visiting a web page, either with any **browser**, or with a **curl** script; you could even set up the web page address on your **modem** in the "custom dynamic DNS" page (i.e.: Fritz!Box supports that). Then web-knock will replace your previous IP address in the whitelist, with the your new one.

![](https://github.com/Linkinverse/web-knock/blob/master/media/screenshot.png)

You can add to the whitelist as many address as you want! **Using a second backup website is recommended**, in case the website goes offline.

All is done with a couple of simple and small BASH scripts using only POSIX commands (awk) without any additional third part software, and best of all: it's completely free and open-source, so you are free to customize the script to fit your needs. If you want to add some security feature feel free to fork it.

## Components

- **PHP script:** *index.php*

  (upload it to any web hosting: when visited it will create a plain-text file containing the IP of the device visiting the page)

  

- **Main script:** *web-portknock.sh*

  (simply calls the firewall update sub-scripts, you can configure this file with your web hosting address, and this to cron to poll for any changes to your IP)

  

- **Firewall update script:** *web-portknock-update.sh*

  (automatically replaces your previous IP in the firewall whitelist, if it has changed)
  

![](https://github.com/Linkinverse/web-knock/blob/master/media/flowchart.jpg)

### Note about security: please note that both your IP and the PHP script will be in an protected subfolder.

**Logs are saved here:** */var/log/youshallnotpass.log*

**Previous IP address are saved on your VPS here:** */etc/web-portknock/**.txt*

------

## Installation and configuration

No installation is required: just upload the included "index.php" on a web hosting (MySQL is not required), add "web-portknock.sh" to cron for root user, then edit it to include the correct URL where you uploaded "index.php"; that's all, you're ready to go. Further details follows.

### Requirements

* iptables
* curl
* awk

### On your web server / web hosting with PHP:

1. Upload *index.php* script on any free or paid web hosting of your choosing, in a subdirectory. MySQL database is **not** required.
2. **Add one of the following protections:**
    1. To avoid other people finding this subdirectory, make sure the file *.htaccess* on the webserver home, includes the following line: `Options -Indexes`
    2. Protect the your newly created subdirectory with a .htpasswd file
    3. Modify the PHP script to ask for a password (please note if you chose this method it can't be automatised)

### On the linux server you want to protect:

1. Edit *web-portknock.sh* to add the URL address where you uploaded the PHP script. Example of configuration:

   ```bash
   web-portknock-update.sh http://yourwebsite.com/myip/hard-to-guess-filename.txt
   ```

2. Add the script to *cron*, **for user root** like this:

   `export VISUAL=nano; crontab -e`

   And add the following line:

   ```
   */10    *   *       *       *       /root/web-portknock.sh >> /var/log/youshallnotpass.log 2>&1
   ```

   **Note**: `*/10` to update the IP in the whitelist every 10 minutes, it should be often enough, but change it according to your preferences. As opposed to the Dynamic DNS, there is no limit to the number of requests.*

3. There is no need to modify the file *web-portknock-update.sh*, unless you want to change the default ports to open. Default ports: 21, 22 (FTP, SSH)

### Verify the rules and connections

1. Check the current firewall rules anytime with: `iptables -n -v -L`
2. Connect using another IP to verify if the protection is working

------

## Usage example

1. Once you ploaded the PHP script on your web hosting, simply **visit the page** from your device, with any browser.

   (You can also automate this process with a script that uses curl to get the page, or also by putting the web address into your modem at the section *"Dynamic DNS"*, if supported)

   **Note:** if you used .htpasswd protection you have to paste the URL like this:

   http://username:password@yourwebaddress.com/subfolder/hard-to-guess-filename.txt

2. Wait for **cron** to execute the script **web-portknock.sh** you previously edited/customized with the URL where you uploaded the PHP script.

   (Suggested curl execution time for polling the new IP: 5-15min)

3. *web-portknock-update.sh* subscript will be called and replace (or add, if not present), a rule on iptables to let you in from the ports you selected. Everyone else will see the ports closed. Default ports: 21,22 (FTP, SSH), customize the script with the port you need if necessary.

4. Get the popcorn and enjoy the view of botnets trying to enter and fail miserably 😃🍿, you can monitor the failing attempts with the command: `tcptrack -i eth0`

![](https://github.com/Linkinverse/web-knock/blob/master/media/botnet-fail.gif)

![](https://github.com/Linkinverse/web-knock/blob/master/media/youshallnotpass.gif)

------

## Contact

### Bugs, feature requests, discussions?

Please open a bug tracking ticket here

### You just appreciate this program:

Send kudos to the original author Marco Trovato using this [contact form](http://ispace.altervista.org/msn/).

## Thanks

* [Pascalbrax](https://github.com/pascalbrax) who taught me the basics.
* My sleep disorder

## License

web-knock is licensed as GNU General Public License v3.0, it means you are free to use it, redistribute it and/or modify it, as long as you keep this license.

If you use this software please consider writing your honest opinion about microsoft or apple, on any place of your choosing (i.e.: any online community), and feel free to express your gratitude for decades of frustration, poorly developed/unsecure/bloated/retarded software like bugged RDP with infinite tries, shared NetBIOS exposed to WAN, various *embrace-extend-extinguish* strategies, and overpriced hardware with programmed obsolescence. ❤️

web-knock is distributed in the hope that it will be useful, but AS IS: WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
