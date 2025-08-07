SMTP is a technical standard for transmitting electronic mail over the network. Like other networking protocols, SMTP allows computers and servers to exchange data regardless of their underlying hardware or software. Just as the use of standardized form of addressing an envelope allows the postal service to operate, SMTP standardizes the way email travels from sender to recipient, making widespread email delivery possible.

### How does SMTP work?
All networking protocol follow a predefined process for exchanging data. SMTP defines a process for exchanging data between an email client and a mail server. An email client is what a user interacts with: the computer or web application where they access and send email. A main server is a specialized computer for sending, receiving, and forwarding emails; users do not interact directly with main servers.

Here is a summary of what passes between the email client and the mail server for an email to begin sending:
- SMTP connection opened: Since SMTP uses the [[Transmission Control Protocol (TCP)]] as its transport protocol, this first step begins with a TCP connection between client and server. Next the email client begins the email sending process with a specialized "hello" command (HELO or EHLO).
- Email data transferred: the client sends the server a series of commands accompanied with the actual content of the email: the email header (including its destination and subject line), the email body, and any additional components.
- Mail Transfer Agent (MTA): The server runs a program called a Mail Transfer Agent (MTA). the MTA checks the domain of the recipient's email address, and if it differs from the sender's, it queries the [[Domain Name System (DNS)]] to find the recipient's [[IP address]]. This is like a post office looking up a main recipient's zip code.
- Connection closed: the client alerts the server when transmission of data is complete, and the server closes the connection. at this point the server will not receive additional email data from the client unless the client opens a new SMTP connection.

### SMTP commands
SMTP command are predefined text-based instructions that tell a client or server what to do and how to handle any accompanying data. Think of them as buttons the client can press to get the server to accept data correctly.
- `HELO/EHLO`: These commands say "hello" and start off the SMTP connection between client and server. `HELO` is the basic version of this command; "`EHLO`" is for a specialized type of SMTP.
- `MAIL FROM`: This tells the server who is sending the email.
- `RCPT TO`: This command is for listing the email's recipients. A client can send this command multiple times if there are multiple recipients. In the example above, Alice's emails client would send "RCPT TO:<bob@example.com>".
- `DATA`: This precedes the content of the email, like: 

```
DATA
Date: Mon, 4 April 2022
From: Alice alice@example.com
Subject: Eggs benedict casserole
To: Bob bob@example.com

Hi Bob,
I will bring the eggs benedict casserole recipe on Friday.
-Alice
```

- `RSET`: This command resets the connection, removing all previously transferred information without closing the SMTP connection. `RSET` is used if the client sent incorrect information.
- `QUIT`: This ends the connection.

### What is an SMTP server?
An SMTP server is a mail server that can send and receive emails using the SMTP protocol. Email clients connect directly with the email provider's SMTP server to begin sending an email. Several different software programs run on an SMTP server:


https://www.cloudflare.com/learning/email-security/what-is-smtp/