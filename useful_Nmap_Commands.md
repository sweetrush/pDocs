## Useful Nmap Commands 
Install nmap in Ubuntu or Debian
```sh
sudo apt install nmap 
```


```bash
nmap sT [IP_toScan][/24] # for Scanning the range of the IP  
```


```bash
nmap -sV -sC -oN [outputfile.extension] -p- [IP_toScan] 
```

# used when ICMP ping request is blocked 
```bash
nmap -T4 -Pn -p- -A [IP_toScan] 
```

```sh
nmap -Pn -T4 --top-ports [Porttoscan] -A [IPAddressTOVictum] 
```

```sh
nmap -p msrpc,http,apex-mesh  [IPAddressTOVictum] 
```

