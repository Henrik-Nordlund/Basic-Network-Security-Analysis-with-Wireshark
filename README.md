<b>Project description</b><br> 
I am a security analyst working for a company that wants to detect certain network packets on the server, from several protocols. The company wants to test the access to an internal database which is using the RADIUS protocol at the moment, and examine just how safe are websites that uses HTTP with Basic Auth or form-based verification. Finally, as a system administrator: is there a way to see encrypted traffic on our own network (HTTPS)? I am tasked to looking into these things.

<b>Objective 1 – capture network traffic and view captured traffic through a display filter</b><br>
Captured traffic had been stored in a pcap file generated in the tool Wireshark. The network protocol of interest was RADIUS. RADIUS, or Remote Authentication Dial-In User Services, is used for authentication of users logging in to a central database. By submitting bogus passwords to access the central database it became obvious that RADIUS is not secure at all - as can be viewed in image below. 

<img src ="https://github.com/Henrik-Nordlund/Basic-Network-Security-Analysis-with-Wireshark/blob/275fba478ef96a815d72139b7e17ee9d5949a8a0/RADIUS.PNG"/>

In addition simple capture of ongoing network was used, using the ethernet Applying DNS as a display filter which displayed the DNS traffic. Ethernet was the interface of interest.
<img src = "https://github.com/Henrik-Nordlund/Basic-Network-Security-Analysis-with-Wireshark/blob/f8e69bf6da86951a7c73d0cc8754bc40cdafb76e/Capturing%20from%20ethernet%203.PNG"/>
By utilizing the display filter, the DNS traffic could be filtered out. As shown below, DNS traffic is not encrypted.
<img src = "https://github.com/Henrik-Nordlund/Basic-Network-Security-Analysis-with-Wireshark/blob/f8e69bf6da86951a7c73d0cc8754bc40cdafb76e/Early%20DNS%20traffic.PNG"/>

<b>Objective 2 – Generate and capture RADIUS traffic</b><br>
A public RADIUS server was utilized and a username and a password was submitted. The RADIUS will then store those two things. 
<img src = "https://github.com/Henrik-Nordlund/Basic-Network-Security-Analysis-with-Wireshark/blob/f8e69bf6da86951a7c73d0cc8754bc40cdafb76e/Public%20Radius%20server.PNG"/>
In wireshark, to ultiize the capture filter even if one knows the interesting protocol one cannot simply type “RADIUS” there because it is not a display filter. But RADIUS uses the 1812 port, so that can be usedto capture the traffic of interest to us. A client to the RADIUS server was useful to send information to it. That traffic will be captured by wireshark.
<img src = "https://github.com/Henrik-Nordlund/Basic-Network-Security-Analysis-with-Wireshark/blob/f8e69bf6da86951a7c73d0cc8754bc40cdafb76e/NTRadPing%20access%20accept.PNG"/>

The correct username, but different passwords to the RADIUS server was sent using the client. In the attached image below different captured responses can be seen from the RADIUS server.
<img src = "https://github.com/Henrik-Nordlund/Basic-Network-Security-Analysis-with-Wireshark/blob/f8e69bf6da86951a7c73d0cc8754bc40cdafb76e/Capturing%20from%20port%201812%20-%20different%20responses.PNG"/>
As can be seen, RADIUS is not encrypted at all.
<imr src = "https://github.com/Henrik-Nordlund/Basic-Network-Security-Analysis-with-Wireshark/blob/f8e69bf6da86951a7c73d0cc8754bc40cdafb76e/Capturing%20from%20port%201812%20-%20username%20clearly%20visible.PNG"/>
However, the password is not shown in Wireshark. RADIUS uses the field “secret key” and the MD5 hashing algoritm to hide the password behind the server and the client.
Wireshark can be used to decrypt such a weak protected password. 
Edit -> Preferences -> Protocols -> RADIUS.
Shared secret, which can be found in the client.  
<img src = "https://github.com/Henrik-Nordlund/Basic-Network-Security-Analysis-with-Wireshark/blob/f8e69bf6da86951a7c73d0cc8754bc40cdafb76e/Capturing%20from%20port%201812%20-%20password%20with%20known%20secret.PNG"/>


<b>Objective 3 – Analyze a HTTP Basic Authentication</b><br>
HTTP is a clear text protocol.  A display filter was used, and not a capture filter. Specifically port 80 to capture only HTTP traffic.
A HTTP website was accessed, where the credentials was prompted for.

<img src = "https://github.com/Henrik-Nordlund/Basic-Network-Security-Analysis-with-Wireshark/blob/f8e69bf6da86951a7c73d0cc8754bc40cdafb76e/Http%20login.PNG"/>

As can be seen this is not secure since Wireshark can see exactly was transmitted.

<img src = "https://github.com/Henrik-Nordlund/Basic-Network-Security-Analysis-with-Wireshark/blob/f8e69bf6da86951a7c73d0cc8754bc40cdafb76e/Http%20login%20in%20Wireshark.PNG"/>
Upon a closer look, the credentials can be directly seen in Wireshark.
<img src = "https://github.com/Henrik-Nordlund/Basic-Network-Security-Analysis-with-Wireshark/blob/f8e69bf6da86951a7c73d0cc8754bc40cdafb76e/Http%20credentials.PNG"/>

<b>Objective 4 – Http Form-Based Authentication and DNS</b></br>
Form based HTTP authentication was created to negate the downsides of basic authentication. It does not send a GET request with the credentials, instead it sends a POST request and passes the credentials with it. Unlike basic authentication, Wireshark will find the credentials in the HTML field.

<img src = "https://github.com/Henrik-Nordlund/Basic-Network-Security-Analysis-with-Wireshark/blob/f8e69bf6da86951a7c73d0cc8754bc40cdafb76e/Display%20captured%20packets%20with%20HTTP%20traffic.PNG"/>

Normally, this would be within TLS, so it does not matter if it is not encrypted. But now the credentials are shown in the form field instead.
DNS traffic, utilizing port 53, are not encrypted either as the picture below demonstrates.

<img src = "https://github.com/Henrik-Nordlund/Basic-Network-Security-Analysis-with-Wireshark/blob/f8e69bf6da86951a7c73d0cc8754bc40cdafb76e/Capturing%20DNS%20traffic%20with%20port%2053.PNG"/>

<b>Objective 5 – Initiate, Capture and Analyze Telnet Sessions</b><br>
Telnet is unencrypted, it’s secure form is SSH. Telnet is a computer protocol built to interacting with remote devices, to access and manage a device.
Telnet uses port 23, and Wireshark was set to capture packets from that port.
PowerShell was used to generate telnet traffic with the following command: telnet tty.sdf.org
Inside Wireshark, rightclick on the first telnet package, choose Follow -> TCP Stream.

<img src ="https://github.com/Henrik-Nordlund/Basic-Network-Security-Analysis-with-Wireshark/blob/f8e69bf6da86951a7c73d0cc8754bc40cdafb76e/Wireshark%20follow%20TCP%20stream.PNG"/>
It clearly shows that telnet is not secure at all.

<b>Objective 6 – Capturing and analyzing SSH Sessions</b><br>
Both telnet and SSH are both used to remotely access a device, but SSH is encrypted and uses port 22. To secure the communication, SSH uses the security protocol TLS.
The differences between captured traffic using SSH relative to telnet was demonstrated by using host tty.sdf.org as a capture filter. PowerShell could then be used to directly access the site: telnet tty.sdf.org and ssh tty.sdf.org respectively.
Back in Wireshark, there is several conversations going on since the same website was accessed with both telnet and SSH. 
Statistics -> Conversations

<img src = "https://github.com/Henrik-Nordlund/Basic-Network-Security-Analysis-with-Wireshark/blob/f8e69bf6da86951a7c73d0cc8754bc40cdafb76e/Conversations%20capturing%20both%20telnnet%20and%20ssl.PNG"/>
Clicking in Follow Stream.. shows what any man in middle would see in respective case.

<img src = "https://github.com/Henrik-Nordlund/Basic-Network-Security-Analysis-with-Wireshark/blob/f8e69bf6da86951a7c73d0cc8754bc40cdafb76e/Telnet%20traffic%20is%20not%20encrypted.PNG"/>
<img src = "https://github.com/Henrik-Nordlund/Basic-Network-Security-Analysis-with-Wireshark/blob/f8e69bf6da86951a7c73d0cc8754bc40cdafb76e/SSH%20traffic%20is%20encrypted.PNG"/>

<b>Objective 7 – Generate, Capture, Analyze and the Decrypt HTTPS Traffic</b><br>
HTTP/S is just HTTP over TLS. HTTP operates over port 80 while HTTPS operates over port 443.
Too see encrypted traffic (HTTP/TLC) is not possible since that require a private key. But seeing encrypted traffic from one’s own machine is possible when using premaster secret key.
The browser (the client) uses this premaster secret key when generating the master secret key to encrypt all shown content.  If it is set locally, in the form of a SSL key log file, Wireshark will be able to decrypt the traffic between that local machine and the server.

Start -> Environment variables -> [System Properties -> Advanced] -> User variables for Administrator -> New 
Then I typed SSLKEYLOGFILE in the field Variable name.  In the field Variable name I typed the location of the file: C:\Users\Administrator\Desktop\ssl-keys.log
Back in Wireshark: Edit -> Preferences -> Protocols -> TLS
In the field (Pre)-Master Secret log file name field I typed in the address: C:\Users\Administrator\Desktop\ssl-keys.log
To view the conversion: Statistics -> Conversations
Clicking in Follow Stream.. shows what any man in middle would see in respective case.
And as be can seen below, it is now decrypted.
<img src = "https://github.com/Henrik-Nordlund/Basic-Network-Security-Analysis-with-Wireshark/blob/f8e69bf6da86951a7c73d0cc8754bc40cdafb76e/Unencypted%20HTTPS%20stream.PNG"/>

Even if you go back to where we see the packets it is more clear.

<img src = "https://github.com/Henrik-Nordlund/Basic-Network-Security-Analysis-with-Wireshark/blob/f8e69bf6da86951a7c73d0cc8754bc40cdafb76e/Clearer%20Packets.PNG"/>


<b>Summery</b><br>
In this project I went through both HTTP and HTTPS, as well as both Telnet and its more secure twin SSH and compared them to determine which is more secure. And even HTTPS can under certain circumstances be decrypted to great relief our systems administrator. I also had a look at RADIUS, an older protocol used for accessing a common database.

