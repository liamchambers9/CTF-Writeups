Introduction
For this lab the objective is to find two flags. My setup includes using the virtualbox hypervisor
with the Mercury.ova file installed and imported. I’m using Kali-Red as my attack machine
running on a separate VM on my hypervisor.
Index
Step 1 - Scanning
Step 2 - Enumeration
Step 3- Exploitation
Step 4 - Initial access & Escalating Privileges
Lab 2 Mercury 2
Step 1
I will start by fetching the IP address on the victim machine. Since the IP address is on the same
network as my Kali-Red attack machine this shouldn't be a smooth process. The command I used
here is sudo arp-scan -l

*Results of the sudo arp-scan -l command*
I will now use nmap to help me with identifying the three IP addresses that I’ve found. Starting
with the first one I’ve noticed it says pfSense.home.arpa, It’s safe to assume that that’s the
firewall. The second IP address listed resulted in All 1000 scanned ports in an ignored state.
*Results of the nmap scans*
Lab 2 Mercury 3
Step 2
The scanning results revealed to me that two services are running on the virtual machine. These
two services are http port 8080 and ssh service port 22. I’ll preceded checking the webpage in the
browser by inputting this command into the browser: http://172.30.0.108:8080/
* http://172.30.0.108:8080/ command*
When I entered this command the results were “Hello. This site is currently in development
please check back later.”
Further, I used Dirb to scan the web directories, it scans the hidden as well as available web
directories. The command I entered was “dirb http://172.30.0.108/”
Lab 2 Mercury 4
*The image above is the dirb command*
As a result of the Directory scan, we obtained the /robots.txt directory. I then went exploring
further into this directory.
*The image above is the blank robots.txt page*
The robots.txt file was empty. I attempted to uncover additional files and directories using
various other tools, but my efforts were unsuccessful. As I kept troubleshooting, I inserted an
asterisk (*) into the URL. This action triggered an error page, which unexpectedly revealed a
new path: /mercuryfacts.
Lab 2 Mercury 5
The command that was entered looked like this “273.30.0.208:8080/*”
*Page that rendered when asterisks was inputted*
When I opened the mercury facts directory, I found a hyperlink consisting a fact. I then clicked
on load a fact.
Lab 2 Mercury 6
*Mercury Facts rendered page*
When I accessed the provided link, I encountered a page displaying Fact id:1. This led me to
suspect that the information was being pulled from a database. Given this structure, I
hypothesized that the page might be susceptible to SQL injection attacks. I started to investigate
this hypothesis further.
Step 3
Since I believed the page could be vulnerable to SQL injection, I went and used the “--dbs”
Lab 2 Mercury 7
which enumerates the database, and “--batch” which allows SQLMap to run continuously
without pausing to ask for decisions. The command looked like this:
sql -u http://172.30.0.108:8080/mercuryfacts/ –dbs –batch
*sql -u http://172.30.0.108:8080/mercuryfacts/ –dbs –batch command*
Following the successful data extraction, we uncovered two databases. Upon examination, the
database named “mercury” appeared to contain more relevant information.
*image documents encountering two databases*
Lab 2 Mercury 8
Having confirmed the page’s vulnerability to SQL injection, I proceeded to extract all available
contents from the mercury database using the following command:
sqlmap-u http://172.30.0.108:8080/mercuryfacts -D mercury --dumpall --batch
*Image shows command input into shell*
The extraction revealed four entries in a table named “users”. When I used the –dumball option
in my command prompt I was able to get a list of all databases, all tables content along with user
names and password. And the fourth entry seems peculiar.
*Image shows the four usernames and passwords in the ‘users’ table *
Moving further the previous port scan results that I used in the beginning gave me two open
ports, one of the ports was ssh. I will use ssh service to login into the user “webmaster” using
this command: ssh webmaster@172.30.0.108
Lab 2 Mercury 9
Once logged in my interface looked like this
*Image of ssh connection to webmaster*
When I put in the password obtained from the extracted entries, I successfully logged in as the
webmaster user. I made sure to confirm my access. I used the ‘id’ command to verify the user
and group names, along with their numeric IDs (UID or group ID) for the current user and other
users on the server. I then employed the ‘ls’ command to display the contents of the directory.
This revealed the presence of a file named user_flag.txt. I then proceeded to view it’s contents
using the ‘cat’ command, which unveiled the first user flag.
Lab 2 Mercury 10
* Content in the user_flag.txt file *
After this I’ll open the directory mercury_proj/ by using the command
cd mercury_proj/ > ls > cat notes.txt
*Image shows “cat notes.txt” command*
*base 64 used to give plaintext*
Now I need to convert the base64 hash into plain text. I will use the echo command as depicted
in my next image. Now I see the password for the user linuxmaster in plaintext.
Step 4
I then used the previously enumerated password, and successfully logged in as the linuxmaster
Lab 2 Mercury 11
user. I then proceeded to check the sudo privileges for this account. My investigation revealed
that linuxmaster has the ability to execute a specific bash script, /usr/bin/check_syslog.sh, with
root privileges. Notably, this execution occurs in a preserved environment due to the SETENV
tag. This configuration could potentially be leveraged for privilege escalation.
*Image shows the SETENV tag*
I then used the head command reading the script, the script was written to execute the tail
program for reading last 10 syslog entries. The command was: head -n 5
/usr/bin/check_syslog.sh
*Image of input for head -n 5 /usr/bin/check_syslog.sh command*
The check_syslog.sh script can be executed within the preserve environment, which presents an
opportunity to exploit the PATH environment variable. My approach involved creating a
symbolic link to the vim editor through the tail command, followed by modifying the
environment variable. This process was accomplished using the commands
ln -s /usr/bin/vim tail
export PATH=$(pwd) :$PATH
Lab 2 Mercury 12
After I executed the aforementioned commands, the next step is to run check_syslog.sh in a
--preserve environment. This action will creates a link between the vim editor and the tail
program, which will open the syslog.sh script in vi editor mode. I achieved this using this
command: sudo --preserve-env=PATH /usr/bin/check_syslog.sh
*Image shows command that was entered*
I then entered cat root_flag.txt
*Image of Mercury competition*
