# PyEmailTools

## Description
PyEmailTools can perform email analysis and email forgering. I am implementing an SMTP, IMAP and POP3 client to make this package easier to use.

## Requirements
This package require : 
 - python3
 - python3 Standard Library

## Installation
```bash
pip install PyEmailTools 
```

## Examples

### Simple usage

#### Command lines

##### Forger
```bash
EmailForgering -t "My Subject" -T "receiver.address@domain.com" -M "my.server.com" -O 587 -L -R "receiver.address@domain.com" -m "My message" -U "my.address@domain.com" -P "my_password" "my.address@domain.com"
```

##### Analysis
```bash
EmailAnalysis -G -S "test" -f "*.eml" -i -a -B -s "@*.com" "files" 
# print all IP and email address, for all files with the eml extension in the current directory, search string with "@<some characters>.com" in email and save it in file named "test<id>.eml"
```

#### Python3

##### Forger
```python
from PyEmailTools.Forger import Forger
from PyEmailTools.SmtpClient import SmtpClient
from getpass import getpass

password = getpass()
receiver = "receiver.address@domain.com"
sender = "my.address@domain.com"

email = Forger(sender, titre = "My Subject")
email.add_recipient(receiver)
email.add_part("My message.", "plain")
email.make_email() # Build the mail

smtpclient = SmtpClient(smtp = "my.server.com", port = 587, username = sender, password = password)
smtpclient.send(email, sender, [receiver])
```

##### Analysis
```python
from PyEmailTools.Reader import Reader

email = Reader("mail.eml")
email.make_email() # Read a email file

del email.email['To']
del email.email['Sender']
del email.email['From'] # Delete some headers values

email.email['To'] = "receiver.address@domain.com"
email.email['Sender'] = "my.address@domain.com" # Add/Change some headers values

email.make_email(email.email.as_bytes()) # Rebuild the email object from bytes
email.save_in_file("mail.eml") # Save change
email.print(part = True, attachements = True, email = email.email) # Analysis (with max verbosity level)

# To send the new email use the SmtpClient class (precedent example)
```

### Advanced usage

#### Command lines

##### Forger
```bash
EmailForgering -t "My Subject" -T "receiver.address@domain.com" -M "my.server.com" -R "receiver.address@domain.com" -m "My message" "my.address@domain.com" 
# Some server can send message without authentication.

EmailForgering -t "My Subject" -T "receiver.address@domain.com" -M "my.server.com" -O 587 -L -D -R "receiver.address@domain.com" -S "mail.eml" -H "<html><body><h1>Message</h1><p>My HTML message</p></body></html>" -p "Test" -N "my.address@domain.com" "my.address@domain.com" 
# Send HTML mail with special name, save it in a file and use a secure connection with debug mode.

EmailForgering -t "My Subject" -T "fake.receiver@domain.com" -M "my.server.com" -O 587 -L -A "CustomHostname" -D -R "receiver1.address@domain.com,receiver2.address@domain.com" -S "mail.eml" -H "<html><body><h1>Message</h1><p>My HTML message</p></body></html>" -p "Test" -k "keywords,test,email" -c "this is my comment" -F -l "en,it" -a "kali.jpg" -m "Simple second message" -d "2020-04-02 11:20:03" -i 3 -s 3 -r 3 -e ROT13 -E "2020-11-12 09:00:00" -N "my.user@domain.com" -x "Email HTML with image." "fake.sender@domain.com" 
# Use a fake receiver email (the real receivers can't see other real receivers address), add custom hostname, add keywords header, add comment header, add language, add attachment, add second body, change the sending date, add importance level, add sensibility level, add priority level, add the encrypted header, add expiring date and use your address to send this message but indicate a fake sender address.
```

##### Analysis
```bash
EmailAnalysis -K -p 1 -s "[0-9]{4,10}" -R -l 5 -H "From" -g "To" -i -a -N "my.server.com" -U "my.address@domain.com" -P "my_password" -L -D "imap" 
# Research 5 emails with a number code (regex : "[0-9]{4,10}") or with the header "From", print all emails with verbosity level 1, reverse the server email order (to get last emails), debug the imap ssl connection.

EmailAnalysis -k -p 4 -s "Dear" -N "my.server.com" -r 1 -H "From" -v "receiver.address@domain.com" -i -a -g "Subject" -O 995 -U "my.address@domain.com" -L -D "pop3" 
# Print (with verbosity level 4) emails with value of header "From" = "receiver.address@domain.com" or with the string "Dear", print Subject value, IP address and email address of all emails, perform 1 pop3 request on my.server.com on port 995 with username : my.address@domain.com (the script ask the password), debugging connection and using SSL.
```

#### Python3

##### Forger
```python
from PyEmailTools.Forger import Forger
from PyEmailTools.SmtpClient import SmtpClient

fake_receivers = "fake.receiver@domain.com"
receivers = ["receiver1.address@domain.com", "receiver1.address@domain.com"]
fake_sender = "fake.sender@domain.com"
sender = "my.address@domain.com"

email = Forger(fake_sender, titre = "My Subject", pseudo = "Custom Name", comments = "My comments", 
       keywords = ["test", "forger", "email", "spoof"], date = datetime(2019, 5, 3), encrypted = "ROT13", 
       expires = datetime(2021, 1, 1), importance = 3, sensitivity = 3, language = ["test", "forger", "email", "spoof"], 
       priority = 3, default_text = "Email HTML with image.")
email.add_recipient(fake_receivers)
email.add_image("image.jpg", "<html><body><p>Message with an image.</p>[image]</body></html>")
email.add_part("<html><body><p>Message without image.</p></body></html>", "html")
email.add_attachement("attachment.txt")
email.add_part("My message.", "plain")
email.email["Fake"] = "Add a fake header"
email.make_email()
email.save_in_file("mail.eml")
email.print(part = True, attachements = True, email = email.email)

smtpclient = SmtpClient(smtp = "my.server.com", port = 587, debug = True)
smtpclient.send(email, sender, receivers, "CustomHostname")
```

##### Analysis
```python
from PyEmailTools.Reader import Reader
from PyEmailTools.ImapClient import ImapClient

client = ImapClient(server="my.server.com", port=None, username="my.address@domain.com", password="my_password", ssl=True, debug=0)
for total, index, data in client.get_all_mail():
	email = Reader()
	email.make_email(data)
	
	print(f"""
Email {index} / {total}
\tFrom: {email.headers.get("From")}
\tTo: {email.headers.get("To")}
\tSubject: {email.headers.get("Subject")}
\tBody:
""")
	
	for name, text in email.part.items():
		if "txt" in name:
			text = text.replace("\n", "\n\t\t")
			print(f"\t\t{name} : {text}")

	if index > 3 :
		break
```

### Python executable

```bash
python3 PyEmailTools.pyz forger -t "My Subject" -T "receiver.address@domain.com" -M "my.server.com" -O 587 -L -R "receiver.address@domain.com" -m "My message" -U "my.address@domain.com" -P "my_password" "my.address@domain.com"

# OR
chmod u+x PyEmailTools.pyz # add execute rights
./PyEmailTools.pyz analysis -G -S "test" -f "*.eml" -i -a -B -s "@*.com" "files"  # execute file
```

### Python module (command line):

```bash
python3 -m PyEmailTools analysis -G -S "test" -f "*.eml" -i -a -B -s "@*.com" "files"
python3 -m PyEmailTools.Forger -t "My Subject" -T "receiver.address@domain.com" -M "my.server.com" -O 587 -L -R "receiver.address@domain.com" -m "My message" -U "my.address@domain.com" -P "my_password" "my.address@domain.com"
```

## Links
 - [Github Page](https://github.com/mauricelambert/PyEmailTools)
 - [Documentation Forger](https://mauricelambert.github.io/info/python/security/PyEmailTools/Forger.html)
 - [Documentation Email](https://mauricelambert.github.io/info/python/security/PyEmailTools/Email.html)
 - [Documentation Reader](https://mauricelambert.github.io/info/python/security/PyEmailTools/Reader.html)
 - [Documentation ImapClient](https://mauricelambert.github.io/info/python/security/PyEmailTools/ImapClient.html)
 - [Documentation PopClient](https://mauricelambert.github.io/info/python/security/PyEmailTools/PopClient.html)
 - [Documentation SmtpClient](https://mauricelambert.github.io/info/python/security/PyEmailTools/SmtpClient.html)
 - [Download as python executable](https://mauricelambert.github.io/info/python/security/PyEmailTools.pyz)
 - [Pypi package](https://pypi.org/project/PyEmailTools/)

## Licence
Licensed under the [GPL, version 3](https://www.gnu.org/licenses/).