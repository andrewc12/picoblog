---
Title: Generate reproducible MAC addresses
Description: A short code snippet
Date: 2014/11/04 23:40:58
Template: blog-post
---

##Generate reproducible MAC addresses 

I wanted to generate MAC addresses for LXC containers, this uses the md5 of the container name so the result will always be the same. Apparently were allowed to use 02, Here's a link on the subject http://serverfault.com/questions/40712/what-range-of-mac-addresses-can-i-safely-use-for-my-virtual-machines
```
container= put the name of the containing here
macaddr=$(echo $container|md5sum|sed 's/^\(..\)\(..\)\(..\)\(..\)\(..\).*$/02:\1:\2:\3:\4:\5/')
echo "lxc.network.hwaddr = $macaddr" >> /var/lib/lxc/$container/config
```
