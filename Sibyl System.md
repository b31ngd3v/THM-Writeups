# Sibyl System: THM CTF

![date](https://img.shields.io/badge/date-02.07.2021-brightgreen.svg)  
![solved in time of CTF](https://img.shields.io/badge/solved-in%20time%20of%20CTF-brightgreen.svg)  
![linux category](https://img.shields.io/badge/category-linux-lightgrey.svg)
![score](https://img.shields.io/badge/score-200+-blue.svg)
![solves](https://img.shields.io/badge/solves-1+-brightgreen.svg)

## Description
The theme of the room is based on an Anime named Psycho Pass.
There are two flags in the machine!  Root the machine and prove your understanding of the fundamentals! This is a virtual machine meant for beginners. Acquiring both flags will require some basic knowledge of Linux and privilege escalation methods.

If you face any problem, contact [b31ngd3v](mailto:b31ngd3v@gmail.com)

## Before we start
Make sure -
1. You're connected to the vpn. **```(Important)```**
2. Your adblocker is disabled. [uBlock Origin, adblocker etc.] **```(Important)``` (or the site will not load.)**
3. You're using Chrome browser. **```(Highly Recommended)``` (for better experience)**


## Enumeration
> Q : Scan the machine, how many ports are open?

Open up your terminal and fire up nmap. Run nmap with ```nmap -sC -sV -oA nmap/ss 10.10.226.130```

![alt text](https://github.com/b31ngD3v/THM-Writeups/blob/main/images/image.png?raw=true "nmap result")

Hmm, looks like the http server is running. We should check what is there.

![alt text](https://github.com/b31ngD3v/THM-Writeups/blob/main/images/Screenshot%20at%202021-07-02%2023-17-23.png?raw=true "port 80")

> Q : What is the hidden directory?

I'm going to use gobuster to discover the hidden directory. Run ```gobuster dir -u http://10.10.226.130/ -w directory-list-2.3-medium.txt --no-error```

![alt text](https://github.com/b31ngD3v/THM-Writeups/blob/main/images/Screenshot%20at%202021-07-02%2023-22-41.png?raw=true "gobuster result")
![alt text](https://github.com/b31ngD3v/THM-Writeups/blob/main/images/Screenshot%20at%202021-07-02%2023-26-55.png?raw=true "hidden directory")

I tried to login with admin:admin but it failed. Then I tried to intercept requests, but It doesn't sending any requests to verify given credentials.

![alt text](https://github.com/b31ngD3v/THM-Writeups/blob/main/images/Screenshot%20at%202021-07-02%2023-27-38.png?raw=true "login")

After inspecting the Login button, I found a function named login.

![alt text](https://github.com/b31ngD3v/THM-Writeups/blob/main/images/Screenshot%20at%202021-07-02%2023-28-12.png?raw=true "login")

I searched the function in the html file, and found the function along with username and hashed password.

![alt text](https://github.com/b31ngD3v/THM-Writeups/blob/main/images/Screenshot%20at%202021-07-02%2023-30-15.png?raw=true "login")

> Q : What is the password?

The hashed password was easy to crack. I just headed over to crackstation and submited the hash and it returned the password in plain text.

![alt text](https://github.com/b31ngD3v/THM-Writeups/blob/main/images/Screenshot%20at%202021-07-02%2023-31-42.png?raw=true "crackstation")

## Capture The Flag

![alt text](https://github.com/b31ngD3v/THM-Writeups/blob/main/images/Screenshot%20at%202021-07-03%2000-53-58.png?raw=true "dash")

Now we are logged in as chief. Let's check what permissions we have with ```sudo -l```

![alt text](https://github.com/b31ngD3v/THM-Writeups/blob/main/images/Screenshot%20at%202021-07-03%2000-56-04.png?raw=true "permissions")

Note. (ALL, !root) this looks suspicious, there is a chance that sudo is vulnerable. We'll check it soon.

![alt text](https://github.com/b31ngD3v/THM-Writeups/blob/main/images/Screenshot%20at%202021-07-03%2001-28-51.png?raw=true "users")

Sibyl System has two users, chief and b31ngd3v. Let's check if we somthing in the home directory of this user.

![alt text](https://github.com/b31ngD3v/THM-Writeups/blob/main/images/Screenshot%20at%202021-07-03%2000-55-24.png?raw=true "home")

Yes, we have a python file named `clickjacking-tester.py`. We only have read permission so we can't write init.

```
# Contributor(s): nigella (@nig)

from urllib.request import urlopen
from sys import argv, exit
from termcolor import cprint

__author__ = 'D4Vinci'

def check(url):
    ''' check given URL is vulnerable or not '''

    try:
        if 'http' not in url: url = 'http://' + url

        data = urlopen(url)
        headers = data.info()

        if not 'X-Frame-Options' in headers: return True

    except: return False


def create_poc(url):
    ''' create HTML page of given URL '''

    code = '''
<html>
   <head><title>Clickjack test page</title></head>
   <body>
     <p>Website is vulnerable to clickjacking!</p>
     <iframe src='{}' width='500' height='500'></iframe>
   </body>
</html>
    '''.format(url)

    with open(url + '.html', 'w') as f:
        f.write(code)
        f.close()


def main():
    ''' Everything comes together '''

    try: sites = open(argv[1], 'r').readlines()
    except: cprint('[*] Usage: python(3) clickjacking_tester.py <file_name>', 'magenta'); exit(0)

    for site in sites[0:]:
        cprint('\n[*] Checking ' + site, 'blue')
        status = check(site)

        if status:
            cprint(' [+] Website is vulnerable!', 'green')
            create_poc(site.split('\n')[0])
            cprint(' [*] Created a poc and saved to <URL>.html', 'green')

        elif not status: cprint(' [-] Website is not vulnerable!', 'blue')
        else: cprint('Every single thing is crashed, Python got mad, dude wtf you just did?', 'magenta')

if __name__ == '__main__': main()```

## Detailed solution
Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation *ullamco laboris* nisi ut aliquip ex ea commodo consequat.
```

Telecolor in not a default package in python, so if we crate a file names termcolor.py in the same directory python will run our file instade of the package files. Let's try this.

```echo "import os;os.system('/bin/bash')" > /home/chief/termcolor.py```

Now run the `clickjacking-tester.py` file as b31ngd3v

```sudo -u b31ngd3v python3 /home/chief/clickjacking-tester.py```

![alt text](https://github.com/b31ngD3v/THM-Writeups/blob/main/images/Screenshot%20at%202021-07-03%2000-59-26.png?raw=true "success")

> Q : User Flag

![alt text](https://github.com/b31ngD3v/THM-Writeups/blob/main/images/Screenshot%20at%202021-07-03%2001-00-34.png?raw=true "user.txt")

Now we'll check if sudo is vulnerable or not. (I'm taking about CVE-2019-14287)

![alt text](https://github.com/b31ngD3v/THM-Writeups/blob/main/images/Screenshot%20at%202021-07-03%2001-56-29.png?raw=true "CVE-2019-14287 check")

No, the sudo is not vulnerable. Now let's check if user b31ngd3v is a member of any group or not.

![alt text](https://github.com/b31ngD3v/THM-Writeups/blob/main/images/Screenshot%20at%202021-07-03%2001-03-12.png?raw=true "id")

Yes, we are a member of docker group. Now head over to gtfobins and search for docker.

![alt text](https://github.com/b31ngD3v/THM-Writeups/blob/main/images/Screenshot%20at%202021-07-03%2001-03-45.png?raw=true "gtfobins")

Copy and paste the following line.

> Q : Root Flag

![alt text](https://github.com/b31ngD3v/THM-Writeups/blob/main/images/Screenshot%20at%202021-07-03%2001-05-28.png?raw=true "sudo access")


## Success

![alt text](https://github.com/b31ngD3v/THM-Writeups/blob/main/images/ezgif-2-76bb9beda24d.gif?raw=true "hacking statue of liberty")
