```sh
#!/bin/bash
 
DEST_USER=$1  
PASSWORD=$2  
HOSTS_FILE=$3  
if [ $# -ne 3 ]; then  
    echo "Usage:"  
    echo "$0 remoteUser remotePassword hostsFile"  
    exit 1  
fi  
  
SSH_DIR=~/.ssh  
echo ===========================  

# 1. prepare  directory .ssh  
mkdir $SSH_DIR  
chmod 700 $SSH_DIR 
  
# 2. generat id_rsa and id_rsa.pub
expect <<EOF
spawn ssh-keygen -b 1024 -t rsa
expect {
  "*Overwrite*" {send "y\r";exp_continue}
  "*key*" {send "\r";exp_continue}
  "*passphrase*" {send "\r";exp_continue}
  "*again:" {send "\r";exp_continue}
}
EOF
  
# 3. generat file authorized_keys  
cat $SSH_DIR/id_rsa.pub>>$SSH_DIR/authorized_keys  
  
# 4. chmod 600 for file authorized_keys  
chmod 600 $SSH_DIR/authorized_keys  

# 5. copy all files to other hosts  
for ip in `cat ${HOSTS_FILE}|awk -F ' ' '{print $1}'|grep -v '127.0.0.1'|grep -v '::1'`
do  
echo $ip
expect <<EOF
spawn scp -r ${SSH_DIR} ${DEST_USER}@${ip}:~/
expect {
  "*continue*" {send "yes\r";exp_continue}
  "*password*" {send "${PASSWORD}\r";exp_continue}
}
EOF

done
```

