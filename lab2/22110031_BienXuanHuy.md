# Lab #2,22110031, Bien Xuan Huy, INSE330380E_01FIE
# Task 1: Transfer files between computers  
**Question 1**: 
Conduct transfering a single plaintext file between 2 computers, 
Using openssl to implementing measures manually to ensure file integerity and authenticity at sending side, 
then veryfing at receiving side. 

**Answer 1**:

### Idea: Use HMAC to ensure both integrity and authenticity.

### Step 1: Install needed packages

To achieve the goal of this lab we first need to install some packages on both containers.

```
apt update
apt install net-tools
apt install openssl
apt install netcat-traditional
```

Explain each options:
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
 
# Task 2: Transfering encrypted file and decrypt it with hybrid encryption. 
**Question 1**:
Conduct transfering a file (deliberately choosen by you) between 2 computers. 
The file is symmetrically encrypted/decrypted by exchanging secret key which is encrypted using RSA. 
All steps are made manually with openssl at the terminal of each computer.

**Answer 1**:


# Task 3: Firewall configuration
**Question 1**:
From VMs of previous tasks, install iptables and configure one of the 2 VMs as a web and ssh server. Demonstrate your ability to block/unblock http, icmp, ssh requests from the other host.

**Answer 1**:



