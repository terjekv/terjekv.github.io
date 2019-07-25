---
title: Ansible for fun and fright
layout: post
categories:
- computer science
---
Ansible has its advantages. For more ephemeral assets, I quite like its client-less design and the way it often works as a substitute for complex scripts. Ansible is also full of active development, with a rich and vibrant community that contributes code in a myriad of ways. This makes Ansible one of those tools it is so easy to reach for…

…and then shoot yourself in the foot.

This week was one of those moments. 

The task was annoying. I needed to copy a directory from one node to another. They both run MacOS, so my immediate thought was simple enough. Create a temporary SSH key on the sender node, copy the public key over to the receiver, and add the key to `~/.ssh/authorized_keys` of the relevant user on the receiver node.

A quick look through the Ansible documentation provided me with a set of tools that are designed to do a lot of this work. One can use [delegate\_to][1] to run things on a specific node, copying is offered via the [synchronize][2] module, and there is a module called [authorized\_key][3] that is designed to manipulate the relevant file. Joy! A solid toolbox is already on offer!

Except. Uhm. No. Trying to get synchronize to use the temporary ssh key (equivalent to `rsync -e “ssh -i /path/to/key”`) fell through. Okay. One can use `shell` and manually construct the rsync command. Not a big deal.

[Authorized\_key][4] however, that module was a big deal. An undocumented feature of its use is that it parses your current file to rebuild it when you add a key. However, when doing so, it creates a dict with the keys as, well, the keys. The consequences of that (immediately obvious) design choice is that if you have a key with multiple option strings, like distinct from-statments, all but one disappear when the file is rebuilt. This specific setup exists on our MacOS machines due to a bug where from=“hostname" didn't work, but from=“ipv4addr" did. Since we dual stack, we also have from=“ipv6addr". 

Of course, after adding the temporary key, the file only contained the from=“ipv4addr” option for the key. This broke all future Ansible scripts as the machine in question tried to connect via IPv6, and then got told to sod off. 

After a bit of panic — I really didn’t want to have to physically find the machine and log into it — the situation resolved itself by adding a -4 to ssh, but, well, ouch. It also doesn’t help that one of the nodes I was testing on had a broken IPv6 route setup, so if I had run the code there, I’d have to gain physical access to the machine. A humorous piece of irony is that the broken IPv6 configuration was performed via Ansible.  

The solution to adding the ssh keys? The [lineinfile][5] module. Cache the ssh key line and add/remove it via lineinfile. It makes no assumptions about the rest of the file, so it will work[tm].


[1]:	https://docs.ansible.com/ansible/latest/user_guide/playbooks_delegation.html#delegation "Ansible documentation : Delegation "
[2]:	https://docs.ansible.com/ansible/latest/modules/synchronize_module.html "Ansible documentation : The syncronize module"
[3]:	https://docs.ansible.com/ansible/latest/modules/authorized_key_module.html "Ansible documentation : The authorized_key module"
[4]:	https://docs.ansible.com/ansible/latest/modules/synchronize_module.html
[5]:	https://docs.ansible.com/ansible/latest/modules/lineinfile_module.html "Ansible documentation : Lineinfile"