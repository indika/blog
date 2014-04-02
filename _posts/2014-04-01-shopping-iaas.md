---
layout: post
title: Shopping for Configuration Management Software [DRAFT]
description: "Shopping for configuration management software"
modified: 2014-04-01
tags: [chef, puppet, iaas, devops]
comments: false
share: true
disqus: y
---

Programs are written, not configured. Infrastructure is configured, not written. While both is a means to construct, the one is more fine grained, and the other is more immutable. There are many different solutions within this grey.

Chef is developed by Opscode, initially developed by Adam Jacob. Initially it was written purely in Ruby, but a year ago since Chef 11, the core was re-written in Erlang, and called ErChef. Erlang is a good language to handle scalability issues. It has been criticized for not having a good web UI. The biggest user of Chef is Facebook.

Puppet is developed by Puppet Labs, initially by Luke Kanies. It is about 9 years old since inception. It too is written in Ruby. It has some big players behind it. The code is simpler, and offers a Ruby DSL as well now.

The basic idea is that the agent node sniffs out the configuration on the target node, and sends this information back to the master, which then decides what to do.

Both Chef and Puppet have been funded between $30-40mil. Google, Cisco System and VMWare, amongst others have backed Puppet. With the extra money, both have started to provision Windows based machines.


Boxen is a toolchain built to provision Mac's, and it uses Puppet. It has been criticized that it uses non-default homebrew locations.

Salt and Ansible exist. Both are written in Python. Not sure what their edge is just yet.

OpenShift over OpenStack is driven by RedHat.

OpenStack is written is Python, and provides an API for instantiating virtual machine resources.
"OpenStack is a cloud operating system that provides support for provisioning large networks of virtual machines, pluggable and scalable network and IP management, and object and block storage"

"OpenStack is an Infrastructure as a Service (IaaS) cloud computing software project founded by Rackspace and NASA in 2010 and open sourced under the terms of the Apache License. The mission of the OpenStack project is to enable any organization to create and offer cloud computing services running on standard hardware"

OpenShift sits on top of OpenStack, and is written in Ruby. The well connected of the two, can help magical autoscaling. CloudForms sits on top of both of these, and help manage heterogenous cloud architectures.
This solution feels less flexible, but it is a turn-key one.






