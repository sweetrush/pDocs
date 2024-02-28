## Useful Nmap Commands 

```bash
nmap -sV -sC -oN [outputfile.extension] -p- [IP_toScan] 
```

# used when ICMP ping request is blocked 
```bash
nmap -T4 -Pn -p- -A [IP_toScan] 
```