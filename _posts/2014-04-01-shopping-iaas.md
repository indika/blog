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

Programs are written, not configured. Infrastructure is configured, not written. While both is a means to construct, the one is more fine grained, and the other is more immutable. There are many different solutions within this grey. This grey is about ease versus control.

I've been shopping for solutions to code my infrastructure.

First, there was, CFEngine.

Puppet is developed by Puppet Labs, initially by Luke Kanies. It is about 9 years old since inception. It too is written in Ruby. It has some big players behind it. Initially it offered a simple DSL to write the configuration, and now the configurations can be written in Ruby as well.

Chef is developed by Opscode, initially developed by Adam Jacob. Initially it was written purely in Ruby, but a year ago since Chef 11, the core was re-written in Erlang, and called ErChef. Erlang is a good environment to handle concurrency. Chef has been criticized for not having a good web UI. The most well known user of Chef is Facebook.

The basic idea is that the agent node sniffs out the configuration on the target node, and sends this information back to the master, which then decides what to do.

Both Chef and Puppet have been funded between $30-40mil. Google, Cisco System and VMWare, amongst others have backed Puppet. With the extra money, both have started to provision Windows based machines.

VMWare has backed Puppet, but both can integrate with VMWare products. Puppet and Chef have support for a variety of hardware.

Boxen is a toolchain built to provision Mac's, and it uses Puppet. It has been criticized that it uses non-default homebrew locations.

Salt and Ansible exist. Both are written in Python.

Salt configurations are written in YAML, and the execution is imperative, and not declarative. I'm postulation that the advantage is that you have fine "grain" control. Resolves some issue about orchestration. Salt uses ZeroMQ to send messages. And has a very "free" philosophy - not paid hosting plans.

Ansible is a lot more lightweigtht. Ansible remotely copies and executes generated Python scripts directly on the managed instances using SSH. Therefore, the only requirements for managing instances with Ansible are Python and SSH.

OpenStack is an IaaS, with the mission of enabling any organisation to create and offer cloud computing services running on standard hardware. This project was founded by Rackspace and NASA in 2010. It is a cloud operating system - meaning that you can set up a cloud, and not an OS that runs on your hardware that is thin. OpenStack is written in Python. The dashboard is written in Django.

OpenShift can sit on top of OpenStack nicely, and is written in Ruby. The well connected of the two, can help magical autoscaling.

CloudForms sits on top of both of these, and help manage heterogenous cloud architectures. This solution feels less flexible, but it is a turn-key one. OpenShift over OpenStack is advocated by RedHat.

The space seems large enough to not just have one clear winner. Each solution might be tairored for a different target market. Puppet and Chef seem to do very similar things. If you have a bunch of hardware, and you want to configure it, then perhaps Puppet is better for you. If you are a developer, and want to extend your reach, then perhaps Chef is better for you. If you are a big corporate and you want a cloud based solution, then perhaps OpenShift over OpenStack is better for you.

There are some finer points about scaling that I want to research.




