# Lab #2,22110031, Bien Xuan Huy, INSE330380E_01FIE
# Task 1: Transfer files between computers  
**Question 1**: 
Conduct transfering a single plaintext file between 2 computers, 
Using openssl to implementing measures manually to ensure file integerity and authenticity at sending side, 
then veryfing at receiving side. 

**Answer 1**:

### Idea: Use HMAC as it ensures both integrity and authenticity.

- Integrity: HMAC uses a cryptographic hash (e.g., SHA-256) with a secret key, ensuring any data alteration changes the HMAC, detecting tampering.
- Authenticity: Only people have the secret key can generate or verify the HMAC, preventing forgery and confirming the data's source.

### Step 1: Install needed packages

To achieve the goal of this lab we first need to install some packages on both containers.

```
apt update
apt install net-tools
apt install openssl
apt install netcat-traditional
```

Explain each option:
- apt install net-tools: In order to use *ifconfig* to check IP adress.
- apt install openssl: Use openssl to achieve secure goal.
- apt install netcat-traditional: Use netcat to send - receive files.

### Step 2: Check IP address of receiver and conduct files to send.

To check IP address of receiver, in the receive container, use command ifconfig.

<img width="500" alt="Screenshot" src="https://github.com/leonart-delux/informationsecurity-labs/blob/015a2ffa6a8ce4d2917eb34a2a1ace3325d0a77c/lab2/image/checkReceiveIP.jpg">

The IP address of receiver in network is: 172.17.0.3.

In container of sender, I create a file named *greet.txt* by using this command

```
echo "Hi from Sender!" > greet.txt
```

Then I use command of openssl to creat a HMAC file named *send.hmac* which contains MAC of *greet.txt*

```
openssl dgst -sha256 -mac HMAC -macopt key:secret123 -out send.hmac greet.txt
```

Secret ket is: secret123

Result in sender:

<img width="500" alt="Screenshot" src="https://github.com/leonart-delux/informationsecurity-labs/blob/e516b181f69b99453c76220c543d9907cfa740e4/lab2/image/sendMAC.jpg">

### Step 3: Send and receive file

In container of receiver, use command of netcat to start receving file:

```
nc -l -p <port> > received_file.txt
```

And in container of sender, use this command to send file:

```
cat file.txt | nc <receiver_ip> <port>
```

We need to send 2 files, so this cycle must to be looped 2 times. Note that the receiver need to start receiving before sender sends file.

The port I will use is 3032.

Result in receiver:

<img width="500" alt="Screenshot" src="https://github.com/leonart-delux/informationsecurity-labs/blob/82ae1c3416ed038d16f142177226553061bb37e0/lab2/image/receiveMAC.jpg">

### Step 4: Check MAC.

In container of receiver, use openssl again to create MAC of *greet.txt*

```
openssl dgst -sha256 -mac HMAC -macopt key:secret123 -out receive.hmac greet.txt
```
 Use diff command to check the difference between *send.hmac* and *receive.hmac*

```
diff send.hmac receive.hmac
```

Result shows nothing mean no differences.

<img width="500" alt="Screenshot" src="https://github.com/leonart-delux/informationsecurity-labs/blob/7b0fc763ffe18971aa2c9ed53d42a09b14e22798/lab2/image/checkDiffMAC.jpg">

We can manually recheck again by **cat** command

<img width="500" alt="Screenshot" src="https://github.com/leonart-delux/informationsecurity-labs/blob/7b0fc763ffe18971aa2c9ed53d42a09b14e22798/lab2/image/checkStep2MAC.jpg">

As we can easily see 2 files are the same.

Conclusion: Mission success.
 
# Task 2: Transfering encrypted file and decrypt it with hybrid encryption. 
**Question 1**:
Conduct transfering a file (deliberately choosen by you) between 2 computers. 
The file is symmetrically encrypted/decrypted by exchanging secret key which is encrypted using RSA. 
All steps are made manually with openssl at the terminal of each computer.

**Answer 1**:

### Step 1: Prepare RSA conditions.

In container of receiver, use these 2 commands of openssl to create public key and private key using RSA algorithm

```
openssl genpkey -algorithm RSA -out private_key.pem
openssl rsa -pubout -in private_key.pem -out public_key.pem
```

Here's the result:

<img width="500" alt="Screenshot" src="https://github.com/leonart-delux/informationsecurity-labs/blob/519cd03bd73a458c0834715ca0f5cbdeaa1d7160/lab2/image/pub_priv_key.jpg">

Then use netcat to send the public key from receiver to sender. As I did this job in previous task, here I will quickly do following commands.

```
// In container of sender
nc -l -p 3032 > receiver_pub_key.pem

// In container of receiver
cat public_key.pem | nc 172.17.0.2 3032
```

We can easily get IP address of sender by using command **ifconfig** in container of sender. 

Here's the result in container of sender:

<img width="500" alt="Screenshot" src="https://github.com/leonart-delux/informationsecurity-labs/blob/519cd03bd73a458c0834715ca0f5cbdeaa1d7160/lab2/image/senderReceivePubkey.jpg">

### Step 2: Send secret key and encrypted file

Back to container of sender, now I will again send encrypted version of *greet.txt* (which I created in previous task). First I will encypt *greet.txt* using AES in CBC mode with key is *secret0724*. The result will be saved in *greet.enc*.

```
openssl enc -aes-256-cbc -in greet.txt -out greet.enc -k secret0724
```

Then I will use RSA algorithm to encrypt the secret key which saved in *secret.txt* using public key of receiver. The result will be saved in *secret_key.enc*.

```
openssl rsautl -encrypt -inkey receiver_pub_key.pem -pubin -in secret.txt -out secret_key.enc
```

Now I will send *secret_key.enc* and *greet.enc* from sender to receiver using netcat

```
// Transfer encrypted secret key
// In receiver
nc -l -p 3032 > enc_key.txt

// In sender
cat secret_key.enc | nc 172.17.0.3 3032

// Transfer ciphertext
// In receiver
nc -l -p 3032 > enc_file.txt

// In sender
cat greet.enc | nc 172.17.0.3 3032
```


### Step 3: Decryption.

In receiver, use this command to decrypt the secret key of ciphertext

```
openssl rsautl -decrypt -inkey private_key.pem -in enc_key.txt -out secret_key.txt
```

Check secret key

<img width="500" alt="Screenshot" src="https://github.com/leonart-delux/informationsecurity-labs/blob/05092f8a631bb85daba53132ef887a31cb254338/lab2/image/revealSecKey.jpg">

As this lab is under our control, we know that this key is the right one.

Then, use the decrypted secret key to decrypt the ciphertext

```
openssl enc -d -aes-256-cbc -in enc_file.txt -out dec_file.txt -k $(cat secret_key.txt)
```

Here's the result.

<img width="500" alt="Screenshot" src="https://github.com/leonart-delux/informationsecurity-labs/blob/c69943bf9d5fe40390fcdca9a7135b3c650be579/lab2/image/result.jpg">

As we can see the content matches.

Conclusion: Mission completed.

# Task 3: Firewall configuration
**Question 1**:
From VMs of previous tasks, install iptables and configure one of the 2 VMs as a web and ssh server. Demonstrate your ability to block/unblock http, icmp, ssh requests from the other host.

**Answer 1**:

### Step 1: Set up environment on server

Randomly choose a VM to be a web server.

First we need to install needed packages:

```
sudo apt install iptables
sudo apt install apache2
sudo apt install openssh-server
```

Explain each package:
- iptables: In order to configure the firewall.
- apache2: The server we will use.
- openssh-server: Turn the server into SSH server, accepting allowed SSH requests.

Start and check status of server with these 2 commands.

```
sudo service apache2 start
sudo service apache2 status
```

Result:

<img width="500" alt="Screenshot" src="https://github.com/leonart-delux/informationsecurity-labs/blob/7938273d87f036043cf0752e23eafc617cc445d1/lab2/image/apacheStart.jpg">

Server is running normally.

Start and check status of SSH server with these 2 commands.

```
sudo service ssh start
sudo service ssh status
```

Result:

<img width="500" alt="Screenshot" src="https://github.com/leonart-delux/informationsecurity-labs/blob/7938273d87f036043cf0752e23eafc617cc445d1/lab2/image/sshStart.jpg">

SSH is running normally.

### Step 2: Test connection

In the remain VM, which is now the client, we need to install *curl* package to test HTTP connection. *openssh-client* package is also required to connect to server using SSH.

```
sudo apt install curl
sudo apt install openssh-client
```

Use *ifconfig* in server terminal, acknowledge the IP address of server is 172.17.0.3. 

<img width="500" alt="Screenshot" src="https://github.com/leonart-delux/informationsecurity-labs/blob/8b720bc24ceca74d382862ed6afe27a113c39f3c/lab2/image/serverIP.jpg">

#### HTTP

Use curl command to test HTTP connection with server:

```
curl -I http://172.17.0.3
```

Option -I is used to display the HTTP header only (not the entire HTML file).

The connection is fine:

<img width="500" alt="Screenshot" src="https://github.com/leonart-delux/informationsecurity-labs/blob/0c84d7aff9263388faeb55491a774fa3e108ece4/lab2/image/HTTP.jpg">

#### ICMP

Use this command to test the ICMP connection with server:

```
ping 172.17.0.3
```

The connection is fine:

<img width="500" alt="Screenshot" src="https://github.com/leonart-delux/informationsecurity-labs/blob/0c84d7aff9263388faeb55491a774fa3e108ece4/lab2/image/ICMPConn.jpg">

#### SSH

As SSH is for secure remote control, we need to create an user and set a password for that user on server first.

I will create a user with username and password are user72 and 123, respectively.

<img width="500" alt="Screenshot" src="https://github.com/leonart-delux/informationsecurity-labs/blob/55696abe00df74c89accc921faaabefe6dca33f2/lab2/image/addUser.jpg">

Then in client, I use command ``` ssh user72@172.17.0.3 ``` to connect to user72 on server (IP 172.17.0.3 of server is specified) and enter password:

<img width="500" alt="Screenshot" src="https://github.com/leonart-delux/informationsecurity-labs/blob/55696abe00df74c89accc921faaabefe6dca33f2/lab2/image/sshConn.jpg">

As you can see, the connection is fine.

### Step 3: Close service on server

Now I will block *HTTP*, *ICMP*, *SSH* requests by using *iptables*.

Using these commands:

```
sudo iptables -A INPUT -p tcp --dport 80 -j DROP
sudo iptables -A INPUT -p icmp -j DROP
sudo iptables -A INPUT -p tcp --dport 22 -j DROP
```

Explain:
- -A INPUT: Add new rule for all incoming requests from other hosts.
- -p <protocol>:  Which port will apply the rule.
- --dport <port>: This rule is applied on specified port only (80 for HTTP, 22 for SSH).
- -j DROP: Drop all data packages match this rule.

As the ICMP works on network layer, there's no port specified in rule.

By running these commands, server will ignore all *HTTP*, *ICMP*, *SSH* requests.

Rule table now:

<img width="500" alt="Screenshot" src="https://github.com/leonart-delux/informationsecurity-labs/blob/55696abe00df74c89accc921faaabefe6dca33f2/lab2/image/iptables.jpg">

Now I will try to connect to server from client using protocols that blocked.

With HTTP:

<img width="500" alt="Screenshot" src="https://github.com/leonart-delux/informationsecurity-labs/blob/74e24ce8a9cb4993575d7b5522be089e139ba3d1/lab2/image/httpClose.jpg">

With ICMP:

<img width="500" alt="Screenshot" src="https://github.com/leonart-delux/informationsecurity-labs/blob/74e24ce8a9cb4993575d7b5522be089e139ba3d1/lab2/image/icmpClose.jpg">

With SSH:

<img width="500" alt="Screenshot" src="https://github.com/leonart-delux/informationsecurity-labs/blob/55696abe00df74c89accc921faaabefe6dca33f2/lab2/image/sshClose.jpg">

As you can see the server no longer response to our requests.

### Step 4: Unblock requests.

Now I will unblock *HTTP*, *ICMP* requests by using these commands:

```
sudo iptables -D INPUT -p tcp --dport 80 -j DROP
sudo iptables -D INPUT -p icmp -j DROP
sudo iptables -D INPUT -p tcp --dport 22 -j DROP
```

The unlock command is pretty same as block command, the different detail is that -D replaces -A, which means that this command deletes the rule instead of adding.

Retest connection:

With HTTP:

<img width="500" alt="Screenshot" src="https://github.com/leonart-delux/informationsecurity-labs/blob/251c73b42bf097483bc9db747afc7e1357d0f148/lab2/image/httpUnblock.jpg">

With ICMP:

<img width="500" alt="Screenshot" src="https://github.com/leonart-delux/informationsecurity-labs/blob/251c73b42bf097483bc9db747afc7e1357d0f148/lab2/image/imcpUnblock.jpg">

With SSH:

<img width="500" alt="Screenshot" src="https://github.com/leonart-delux/informationsecurity-labs/blob/55696abe00df74c89accc921faaabefe6dca33f2/lab2/image/sshReconnect.jpg">

The connections from those requests are available again.

Conclusion: Misson success.






