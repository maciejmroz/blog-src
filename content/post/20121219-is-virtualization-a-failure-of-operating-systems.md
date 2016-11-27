+++
title = "Is virtualization a failure of operating systems?"
date = "2012-12-19"
slug = "is-virtualization-a-failure-of-operating-systems"
categories = [ "Technology", "Server Side" ]
tags = [ "Virualization"]
+++

One thing has hit me while watching of this year's LinuxCon Europe presentations - one of reasons virtualization exists in the first place is operating system inability to properly isolate the applications from each other. Look at it this way: the role of operating systems is abstracting the physical hardware from the applications. And what is the role of hypervisor? Abstracting the hardware from the opertaing system(s) … Which makes the operating system an application running under the hypervisor, doesn't it?

Now, let's think of it from other angle - why are we using virtualization? We are virtualizing applications, and operating system is only a required intermediary. Typical workload of virtual machine is exactly one application (in general case this is one-to-many relationship, because often single application spans multiple VMs). We do this partly to achieve good utilization of physical hardware, but also to achieve isolation at the level close to "physical box per every application". Not as secure, but conceptually equivalent.

In fact, this level of isolation is desireable on the desktop, too. VM to run untrusted USB stick? Seems like a good idea, and that's only one of possible use cases. If we are running entire operating system just to run one application, isn't it a waste? What can run under the hypervisor doesn't really have to be a complete operating system. There are projects like Erlang on Xen ([erlangonxen.org](http://erlangonxen.org)) or Mirage that seem to skip the OS and talk straight to the hypervisor API - I am not familiar with the implementation details of these projects, but that seems to be general idea. And that makes hypervisor … an operating system. An operating system that opens up plethora of new and exciting possibilities, and one much better fit for todays world - but still an operating system.