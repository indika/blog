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

One typically does not configure a program. One writes a program. The distinction is that you have greater control over the granularity of what you are creating.

When it comes to configuration, you meet something immutable. It may be immutable because it is constraint, or you trust it and don't want to change it's behaivour.


What are some of the solutions available?


Chef is developed by Opscode, initially developed by Adam. It was written in Ruby, but recently since Chef 11, the core was re-written in Erlang, and called ErChef. Erlang is a good language to handle scalability issues. It has been criticized for not having a good web UI.

Puppet is developed by Puppet Labs, initially by Luke Kanies. It too is written in Ruby. It has some big players behind it. The code is simpler, and offers a Ruby DSL as well now.

Both Chef and Puppet have been funded between $30-40mil. VMWare is backing Puppet.

Boxen is a toolchain built to provision Mac's, and it uses Puppet. It has been criticized that it uses non-default homebrew locations.

Salt and Ansible exist. Both are written in Python. Not sure what their edge is just yet.

OpenShift over OpenStack is driven by RedHat. OpenStack is written is Python, and provides an API for instantiating virtual machine resources. OpenShift sits on top of OpenStack, and is written in Ruby. The well connected of the two, can help magical autoscaling. This solution feels less flexible, but it is a turn-key one.






