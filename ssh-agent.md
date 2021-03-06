# ssh-agents 

I kept getting permission error (private) even though I set up my public/private keys correctly. 

Turns out that my ssh-agent did not have any identities linked. 

## How to fix this 

Get ssh-agent to start 
``` 
eval `ssh-agent`
```
This should return a `Agent pid {pid} `

Check that it is running 
```
echo $SSH_AGENT_SOCK
```
$SSH_AUTH_SOCK contains the path of the unix file socket that the agent uses for communication with other processes. This is essential for ssh-add. If you do not have that envrionment variable, see [3]

Check if you have any private keys accessible. 
```
ssh-add -l
```
To add in your keys, run 
```
ssh-add {path} 

Note: Your keys are usually contained in `~/.ssh/id_rsa*
```
Check that it's been successfully added with 
```
ssh-add -l
```

## Persistant ssh-agent identities 

I am using WSL so everytime I restart the computer, I have to read the identities.  

Whilst this is good security practice [4], it's quite annoying. Although this isn't persistent, it's better than nothing. 

Add this to `~/.bashrc` 
```shell
if [ -z "$(pgrep ssh-agent)" ]; then
   rm -rf /tmp/ssh-*
   eval $(ssh-agent -s) > /dev/null
else
   export SSH_AGENT_PID=$(pgrep ssh-agent)
   export SSH_AUTH_SOCK=$(find /tmp/ssh-* -name agent.*)
fi
``` 

And then add an alias 
```bash
alias key_add="eval \`ssh-agent\` > /dev/null 
&& echo $SSH_AGENT_SOCK > /dev/null 
&& ssh-add -t 30m ~/.ssh/id_rsa*"

alias key_del="ssh-add -D"
```

Note: Don't need `-t 30` (deletes after 30mins) but sEcUrItY.

See [5]

## References 

[1] https://www.ssh.com/ssh/agent  
[2] https://stackoverflow.com/questions/22154423/how-is-ssh-auth-sock-setup-and-used-by-ssh-agent  
[3] http://blog.joncairns.com/2013/12understanding-ssh-agent-and-ssh-add/  
[4] https://github.com/NetSPI/sshkey-grab  
[5] https://www.scivision.dev/ssh-agent-windows-linux/
