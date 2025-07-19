---
layout: post
title: Adding swap space to a running EC2 instance
description: Quick guide to adding swap space to an EC2 instance without stopping it, helping resolve memory issues on resource-constrained servers.
---

Sometimes you want to run some trivial software on an EC2 instance. You don’t really pay attention to what the requirements are or have any performance concerns. So you end up spinning up something with very minimal resources. The software hums along nicely and you think nothing of it. It runs for several days, unnoticed. Then you realize the software stops running for some reason requiring you to restart it. Since you somewhat need the software to run, it becomes a bit annoying. Not annoying enough for a full out solution, but annoying enough you want to have to stop intervening. 

Sometimes the lack of resources becomes a problem, especially in the memory department. Instead of spending more money to fix this, a quick and easy solution is to throw some swap on there to test it out. By default EC2 instances do not have swap enabled (for good reason), but since you only want to test things out before committing, it’s quite easy to add swap on the fly.

1. First confirm that there is no swap available by using the `free` command.
    ```
    $ free
                  total        used        free      shared  buff/cache   available
    Mem:         467100      291896        5172         788      170032       88392
    Swap:             0           0           0
    ```
2. Then prep a swap file, here is a 1gb file, and then turn it on.
    ```
    sudo /bin/dd if=/dev/zero of=/var/swap.1 bs=1M count=1024
    sudo /sbin/mkswap /var/swap.1
    sudo chmod 600 /var/swap.1
    sudo /sbin/swapon /var/swap.1
    ```
3. Confirm that the OS sees the swap file by running the `free` command again.
    ```
    $ free
                  total        used        free      shared  buff/cache   available
    Mem:         467100      293124        5200         728      168776       87424
    Swap:       1048572         256     1048316
    ```

If all goes well, and your software was in fact failing because of a memory issue, the OS should start to use this swap file when it starts to run out of memory.

This isn’t a recommended permanent solution, but an easy way to find out if memory is the root of your issue. If it is, I’d recommend allocating a larger EC2 machine, or permanently enabling swap.
