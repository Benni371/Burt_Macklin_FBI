# Story

Burt Macklin, FBI Agent, has been working on building a foolproof case against Sasha Fang, leader of the infamous Fang Gang. Sasha's rise to infamy came through her manufacturing breakthrough of Pawnee Purple, a new drug that could bring Pawnee's culture and people to their knees. Burt was on the verge of piecing together the final bits of evidence to put her and her whole operating away for good. Unfortunately, Macklin hasn't been seen for 3 days and is considered MIA. Fortunately, Macklin was smart. He has hidden his evidence on 3 separate computers. The agency has taken an image of these machines and has dispersed them to us. Its now up to us to collect the evidence on our way through his machines and retrace Burt's last known footsteps. Lets pray that we find him......alive.

Due to the agencies increased focus on finding Burt, we let our guard down and the Fang Gang broke into the internal network. They managed to steal key evidence files in the Fang Gang Case and have exfiltrated them from our systems. Unfortunately we didn't take backups but we managed to obtain packet captures of the traffic that occurred close to the time of the breach. Find how they got in, recover the files and prepare an incident report. Boss is going to want to hear about this........

Disclaimer: We were originally going to have Burt as a CIA agent since its more related to computer crime/espionage etc but advertising was hard because we would have to change Andys jacket. So some stuff says CIA some stuff says FBI. In the end he is an government agent.
# Part 1 - Disk Analysis
## Burt_01
### Overview
The aim of this box is to hep you learn some easy file forensics. Finding hidden files, deobfuscating code, recognizing certain patterns, etc. Not too difficult but gives you a taste of what's to come.
- You should find the following items
	- A purchase history (evidence)
	- An encrypted zip folder (password not found on this machine)
	- A journal (clues)
	- Credentials to the next machine
### Solve
- Credentials: `burtm:macklin_CIA`
- The desktop is a clue that the you should probably look through the home folder
	- `ll` will reveal the `.investigation` folder
	- `cd` into the folder and `ll` again to find hidden evidence
		- the purchase history shows that sasha is working with a local pharmacist to make her drug
		- `.evidence.zip` is an encrypted file with more evidence. Password is found on the next machine
		- `.journal_01` is base64 decoded. Upon decoding it (`cat .journal_01 | base64 -d`) you will find a little story as well as the credentials to the next machine

## Burt_02
### Overview
This machine is windows based and is a great way to learn about windows forensics. This one is a little harder and employs the use of autopsy to help investigate the things that have happened on the computer.
- You should find the following items
	- An email detailing the deals that Sasha has mad with the local pharmacist
	- A journal_01 that has some hidden clues
	- The password to the previous encrypted file
	- The password to the next machine
	- Some more evidence files
### Solve
- Credentials: `burtm:onthecase`
- Once signed in you will notice two 3 items. Autopsy, HintsHelp.txt and burty.
	- Autopsy will launch and will show `burty`'s case file along with the image that was taken
	- HintsHelp.txt will help you see what you need to find
- First step would be to open autopsy. It should open the current case referencing burty. 
	- It might be a little slow so give it some time. If it doesnt launch, navigate to `C:\Users\burtm\Downloads` and find the autopsy installer. Run it and it should give you an option to repair
	- To open the already indexed case. Click open case and navigate to `burty` and select the case file
- EVEN IF AUTOPSY doesnt work you can still find all the information by looking through the windows computer. It just makes it easier
- Once its working start looking through the files
	- Interesting places would be
		- Recent documents
		- Powershell history
		- Deleted Files
- You should be able to find the journal_03 or journal_02 from one of these steps. Once you have found the journal read through it carefully. It should give you some clues
	- Journal_02 is in the videos folder. 
		- Hint is: Evidence is in a `Halter Net Day the Stir Aim`. If you have ever played mad gab this might be easy for you. If not look it up and you'll get the gist. Its saying `Alternate Data Stream`
		- Inside the journal_02 file I have hid the email we are looking for. Use powershell to recover it `Get-Item -Stream * ./journal_02`
	- Journal_03 is in the recycle bin
		- This contains the password for the zip file from the first machine `april_is_hot`
	- The third journal also clues in that the next password is contained in powershell history. This can be retrieved by looking at powershell history in the terminal or by using this command `notepad (Get-PSReadlineOption).HistorySavePath`
		- The password is `april_ludgate_has_a_crush_on_me`
## Burt_03
### Overview
### Solve

# Part 2 - Network Analysis
## PCAP1
### Overview
This pcap deals with how the network intrusion happened. Turns out the FBI is not as secure as we would hope it would be. 
- Through this pcap you should be able to discover/learn the following:
	- How the attackers got initial access?
	- How many files they downloaded?
	- Recover the files (5)
	- The password to the next pcap file
### Solve
- Just take a look through the pcap it will become very evident very quickly that there is something going on with FTP
	- If you follow a tcp stream for one of the FTP packets you should be able to see that `jeff_baker` is the user and that there is a service that is bruteforcing his password
	- At one point you will be able to see a successful login with his credentials in plain text. So the attackers gained initial access by bruteforcing ftp
- From there you will be able to see some get commands being issued to grab files from the server.
	- To extract the downloaded files:
		- filter by `ftp-data`
		- Right click on one of the packets and click `Follow -> TCP Stream`
		- In the new window change the type to raw
		- Click `Save As` to export the files
- Use the password you got from the ftp traffic to unlock the next pcap
## PCAP2
### Overview
This pcap is analyzing the additional exfiltration of files done by the fang gang. 
In this pcap you should be able to find/answer the following:
- How did they get in this time?
- What did they do?
- How were they able to access burts files?
- How did they exfiltrate the files? What did they do to them?
- Decrypt the evidence folder based on the network traffic you have seen
### Solve
- The pcap is relatively large so when you think about things being exfiltrated, you should think that its found in some type of stream. HTTP, SSH, FTP are good places to look
- If you analyze HTTP traffic you should find an API request endpoint that contains a MD5 has of the password they used to encrypt the zip folder. Cracking that hash leads to the password `tinkerbell1` (this can be done through hashcat or crackstation)
- The PNG image details how they were able to access burts files. Bad group permissions
- Once you have found the password and collected all the pieces of evidence write out an incident report with the things that you have found.


## Resources
the PCAP files are included in the resources folder. Burt_01 is hosted here: https://byu.box.com/s/5b6t8rka4kxzjnqhqt36t9kp80177b82. Burt_02 and Burt_03 images are not included due to their size if you would like access to them please contact me at jobs@benshkies.dev and we can figure out a way to transfer them.
