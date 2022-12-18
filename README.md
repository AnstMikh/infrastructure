# 2 - Working with remote servers

**git branch name:** jbrowser

## Theory [2]

* [0.4] What are [computer ports](https://www.cloudflare.com/learning/network-layer/what-is-a-computer-port/) on a high level? How many ports are there on a typical computer?

**Computer ports** are software-based unique identifiers helping an operating system to divide information streams. Port is like a door and OS has several doors. It sends and receives different kinds of information through different predefined doors. When a package of information is sent from a specific computer port it has  the information of a port through which it should be recieved in a target computer. And the target computer is 'listening' to a certian door to get this kind of information.

Port is an integer number ranging from 0 to 65535, but for TCP zero port is reserved and cannot be used while in UDP it means the absence of port. Not all the ports are used in a typical computer. About 18 ports are intensively used and are commonly known:

|Port number|Protocol used|
|---|---|
|20|File Transfer Protocol (FTP) Data Transfer|
|21|File Transfer Protocol (FTP) Command Control|
|22|Secure Shell (SSH) Secure Login|
|23|Telnet remote login service, unencrypted text messages|
|25|Simple Mail Transfer Protocol (SMTP) email delivery|
|53|Domain Name System (DNS) service|
|67, 68|Dynamic Host Configuration Protocol (DHCP)|
|80|Hypertext Transfer Protocol (HTTP) used in the World Wide Web|
|110|Post Office Protocol (POP3)|
|119|Network News Transfer Protocol (NNTP)|
|123|Network Time Protocol (NTP)
|143|Internet Message Access Protocol (IMAP) Management of digital mail|
|161|Simple Network Management Protocol (SNMP)|
|194|Internet Relay Chat (IRC)|
|443|HTTP Secure (HTTPS) HTTP over TLS/SSL|
|546, 547|DHCPv6 IPv6 version of DHCP|

* [0.4] What is the difference between http, https, ssh, and other protocols? In what sense are they similar? Name default ports for several data transfer protocols.

All the client-server protocols determine the semantics of client-server interractions. They work on the application level and hardware does not know anything about them. When data are transferred with a protocol it means that protocol-specific blocks of information are added to the data to be recognizable by a network participant. The protocol-specific blocks contain some meta-information about the version of the protocol, date, time, status, etc. The content of the block is what differs one protocol from another.

One more difference is encryption while protocol is being transfered. HTTP, FTP have no encryption while HTTPS, SSH or SFTP have.

Of course, they are used for different tasks. Roughly speaking, HTML pages are trensfered with HTTP/HTTPS protocols and SSH, FTP, FTPS, SFTP provide remote work in file systems.  

The list of default ports is presented above. The most famous are port 80 for http, 443 for https and 20/21 for FTP.

* [0.4] Explain briefly: (1) what is IP, (2) what IPs are called 'white'/public, (3) and what happens when you enter 'google.com' into the web browser. 

**IP** is a unique identifier of a device in a cumputer network (global or local). It works with the help of IP protocol on the hardware-based network layer. It have two versions: IPv4 looks like this: `192.0.2.235`; IPv6 looks like this: `2001:0db8:11a3:09d7:1f34:8a2e:07a0:765d`.

**Public IP** address is an IP address available to Internet users, as opposed to a **private IP** address, which belongs to a local network. Several IP numbers are reserved to be used in local networks.

When I make a 'google.com' request into my web browser, this URL goes to the nearest available DNS server. There, 'google.com' is resolved to IP address of the Google website and the IP is sent back to my browser. After that, my browser makes a request using the obtained IP to the Internet. The Google web server receives my request and replies with HTML page.

* [0.4] What is Nginx? How does it work on the high level? List several alternative web servers.

**Nginx** is a most widespread web server. It listens to the ports 80 (for http) and 443 (for https), processes http/https requests and replies to these requests or proxies them to other servers or other ports of the same server. It is a software server which must be configured on a hardware server (or a regular computer) to work correctly.

**Apache** is one of the most famous alternatives of nginx, to my opinion, but there are also LiteSpeed Web Server, Microsoft-IIS, etc. 

* [0.4] What is SSH, and for what is it typically used? Explain two ways to authenticate in an SSH server in detail.

**SSH** is a network protocol allowing remote operating system management. It is of application level and it uses encryption during information transfer.

There are two ways to authenticate in an SSH server.

1. Authentication by password. In this case, we send a request to an SSH server to connect. At this moment, a unique fingerprint is created and saved on our machine. It looks like this `12:f8:7e:78:61:b4:bf:e2:de:24:15:96:4e:d4:72:53`. The fingerprint helps to identify our special client-server connection. It helps to prevent the connection of computer with the same IP address to the SSH server because such a computer does not have the fingerprint. SSH server has a database with clients IPs and their passwords. When connection is initiated, SSH server requires to enter a password. When password is entered and server checked it the connection starts.

2. Authentication by key couple. **Private and public keys** are generated. The private key is stored in a client computer and should not be released to the public. Public key is open to everyone and makes no sense without the private one. Actually, public key is something like a lock. Everyone sees it but only a private key holder can open it. To start a connection client sends its public key to an SSH server to recognize the client. The fingerprint is created as mentioned in 'Authentication by password'. If the server finds match of the public key it sends an information encrypted with the help of the public key. It cannot decode the information without a private key. It just sends the encrypted information to the client who decodes it with its private key.

## Problem [6.5]

A real-life situation that occurred to me several times over the years.

Imagine wrapping up a large bioinformatics project and wanting to share raw data with your colleagues in a friendly and straightforward format. The best option would be to use an online genome browser and host your data remotely, so it is easily accessible by anyone with a valid link. This is exactly what we will be doing here.

*Please consider doing this HW using Linux since setting up the SSH client on Windows is painful, and I won't be able to help you.*

Steps:
* [2] Create a new virtual machine in the Yandex/Mail/etc cloud (order at least 10GB of free disk space). Generate SSH key pair and use it to connect to your server.

Create a new virtual machine in the Yandex Cloud, chose these paramenets:

|OS|Type|Size|vCPU|RAM|
|---|---|---|---|---|
|Ubuntu 22.04|HDD|20 Gb|2|6 Gb

Create private and public keys with the command `ssh-keygen -t ed25519` as recommended [here](https://cloud.yandex.ru/docs/compute/operations/vm-connect/ssh). Press Enter three times to skip all.

Print the public key: `cat ~/.ssh/id_ed25519.pub`. That's how it looks:

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIyMEc6lQmNceItLElloyFPeXmGPvhxN9oeBO7x/P+QP snitkin@snitkin-NBR-WAX9
```

Copy and enter the key in the corresponding field to create a new virtual machine.

Connect to server by user name (`dvsnitkin`) and IP (`158.160.19.14`):

`ssh dvsnitkin@158.160.19.14`

* [1] Download the latest human genome assembly (GRCh38) from the Ensemble FTP server ([fasta](https://ftp.ensembl.org/pub/release-108/fasta/homo_sapiens/dna/Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz), [GFF3](https://ftp.ensembl.org/pub/release-108/gff3/homo_sapiens/Homo_sapiens.GRCh38.108.gff3.gz)). Index the fasta using samtools (`samtools faidx`) and GFF3 using tabix.

Download the human genome and unzip:

```
wget https://ftp.ensembl.org/pub/release-108/fasta/homo_sapiens/dna/Homo_sapiens.GRCh38.dna.primary_assembly.fa.gz
wget https://ftp.ensembl.org/pub/release-108/gff3/homo_sapiens/Homo_sapiens.GRCh38.108.gff3.gz
gunzip *
```

Install samtools and tabix.

```
sudo apt-get install -y samtools
sudo apt-get install -y tabix
```

Index FASTA files:

```
samtools faidx Homo_sapiens.GRCh38.dna.primary_assembly.fa
```

Sort, zip the GFF3 file as prescribed [here](http://www.htslib.org/doc/tabix.html) and index with tabix:

```
in_gff=Homo_sapiens.GRCh38.108.gff3
(grep "^#" $in_gff; grep -v "^#" $in_gff | sort -t"`printf '\t'`" -k1,1 -k4,4n) | bgzip > sorted.$in_gff.gz;
tabix -p gff sorted.$in_gff.gz;
```

* [1] Select and download BED files for three ChIP-seq and one ATAC-seq experiment from the ENCODE (use one tissue/cell line). Sort, bgzip, and index them using tabix.

Cell line: **A549**

<u>ATAC-seq</u>: https://www.encodeproject.org/experiments/ENCSR288YMH/

<u>ChIP-seq</u>

**JUNB**

https://www.encodeproject.org/experiments/ENCSR656VWZ/

**CTCF**

https://www.encodeproject.org/experiments/ENCSR432AXE/

**EP300**

https://www.encodeproject.org/experiments/ENCSR044IFH/

Download three ChIP-seq and one ATAC-seq BED files.

```
wget https://www.encodeproject.org/files/ENCFF619XXH/@@download/ENCFF619XXH.bed.gz -O atac.bed.gz
wget https://www.encodeproject.org/files/ENCFF536CDN/@@download/ENCFF536CDN.bed.gz -O JUNB.bed.gz
wget https://www.encodeproject.org/files/ENCFF824SNC/@@download/ENCFF824SNC.bed.gz -O CTCF.bed.gz
wget https://www.encodeproject.org/files/ENCFF448IBS/@@download/ENCFF448IBS.bed.gz -O EP300.bed.gz
gunzip *.bed.gz
```

Sort, bgzip and index the BED files with tabix:

```
for c in atac JUNB CTCF EP300
do
    (grep "^#" $c.bed; grep -v "^#" $c.bed | sort -t"`printf '\t'`" -k1,1 -k4,4n) | bgzip > sorted.$c.bed.gz;
    tabix -p bed sorted.$c.bed.gz;
done
```


* [1] Download and install [JBrowse 2](https://jbrowse.org/jb2/). Create a new jbrowse [repository](https://jbrowse.org/jb2/docs/cli/#jbrowse-create-localpath) in `/mnt/JBrowse/` (or some other folder).

Install conda:

```
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
```

Install JBrowse 2 via conda:

```
miniconda3/bin/conda install -c bioconda jbrowse2
```

Create a jbrowse repository:

```
sudo /home/dvsnitkin/miniconda3/bin/jbrowse create /var/www/html/jbrowse/
```

* [0.25] Install nginx and amend its config(/etc/nginx/nginx.conf) to contain the following section:
```
http {
  # Don't touch other options!
  # ........
  # ........

  # Add this:
  server {
		listen 80;
		index index.html;

		location /jbrowse/ {
			alias /mnt/JBrowse/;	
		}
	}

  # ........
}
```

Install nginx:

```
sudo apt-get install -y nginx
```

Change the config:

```
sudo nano /etc/nginx/nginx.conf
```

I add only this code because I don't need alias for JBrowse to be accessed:

```
server {
    listen 80;
    index index.html;
}
```


* [0.25] Restart the nginx (reload its config) and make sure that you can access the browser using a link like this: `http://64.129.58.13/jbrowse/`. Here `64.129.58.13` is your public IP address.

Reload nginx:

```
sudo nginx -s reload
```

The link `http://158.160.19.14/jbrowse/` is now working.

* [1] Add your files to the genome browser and verify that everything works as intended. Don't forget to [index](https://jbrowse.org/jb2/docs/cli/#jbrowse-text-index) the genome annotation, so you could later search by gene names.

Add the genome assembly:

```
sudo /home/dvsnitkin/miniconda3/bin/jbrowse add-assembly /home/dvsnitkin/Homo_sapiens.GRCh38.dna.primary_assembly.fa --load copy --out /var/www/html/jbrowse/
```

Index the genome annotation:

```
sudo /home/dvsnitkin/miniconda3/bin/jbrowse text-index --file=/home/dvsnitkin/sorted.Homo_sapiens.GRCh38.108.gff3.gz
```

Add tracks:

```
for c in atac JUNB CTCF EP300
do
    sudo /home/dvsnitkin/miniconda3/bin/jbrowse add-track sorted.$c.bed.gz --load copy --out /var/www/html/jbrowse
done
```

**Remember to put a persistent link to a session with all your BED files and the genome annotation in the report. I must be able to access it without problems.**

## Extra points [1.5]

* [1] Create a Docker container for running JBrowse 2. It should be a self-contained application, listening on the default HTTP port. Users must be able to mount directories with custom configs and access them later without any problems. 

Hint: to specify the config, use the config=PATH query parameter. E.g. `http://64.129.58.13/jbrowse/?config=my_folder%2Fconfig.json` where `my_folder%2Fconfig.json` is the [escaped](https://en.wikipedia.org/wiki/Percent-encoding) path to the config file.

* [0.5] Give an in-depth explanation of the OSI model and how the TCP/IP stack works. Don't copy-paste descriptions from the internet; paraphrase and shorten as much as possible (imagine writing a cheat sheet for yourself).
