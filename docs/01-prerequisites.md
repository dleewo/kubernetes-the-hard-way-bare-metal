> This is a partial copy of Kelsey Hightower's [Prerequisites](https://github.com/kelseyhightower/kubernetes-the-hard-way/blob/master/docs/01-prerequisites.md) document.  The steps are identical for bare metal installs
---
<br />

# Prerequisites

## Hypervisor

You will need a hypervior on which you can provision the 7 necessary virtual machines.

Here are some options:

- [VMWare Workstation](https://www.vmware.com/products/workstation-pro.html)
- [VMWare Fusion](https://www.vmware.com/products/fusion.html) (for Mac OSX Users)
- [Oracle VirtualBox](https://www.virtualbox.org/)
- [KVM](https://www.linux-kvm.org/page/Main_Page)
- [VMWare ESXi](https://www.vmware.com/products/esxi-and-esx.html)

I use VMWare ESXI running on an unsed PC with a bunch of RAM.  You can get a free license for personal use.

If you wish to use actual machines, these [Celeron and Atom based mini-PCs](https://www.amazon.com/gp/product/B07ZYCZJVQ/ref=ppx_yo_dt_b_asin_title_o02_s00?ie=UTF8&psc=1) are an attraction option at just around US$110 each.



## Running Commands in Parallel with tmux

[tmux](https://github.com/tmux/tmux/wiki) can be used to run commands on multiple compute instances at the same time. Labs in this tutorial may require running the same commands across multiple compute instances, in those cases consider using tmux and splitting a window into multiple panes with synchronize-panes enabled to speed up the provisioning process.

> The use of tmux is optional and not required to complete this tutorial.

![tmux screenshot](/images/tmux-screenshot.png)

> Enable synchronize-panes by pressing `ctrl+b` followed by `shift+:`. Next type `set synchronize-panes on` at the prompt. To disable synchronization: `set synchronize-panes off`.

Next: [Installing the Client Tools](02-client-tools.md)