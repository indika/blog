---
layout: post
title: Cooking a Web Server with Chef-Solo
description: "Cooking a Web Server with Chef-Solo"
modified: 2014-01-11
tags: [chef, chef-solo, web server, devops]
comments: false
share: true
disqus: y
---



Stasis is unstable. Chaos always seeps through.
My web server was so well configured, that it left me in a position reluctant to make any changes.
Thus, this component fell into stasis.
It eventually broke, and all development came to a halt, until it was resolved.

With Chef, you can declare your server architecture, and cook it as many times as you like. It operates idempotently. Re-initializing and re-cooking your server often is the best counter against stasis.

Learning to cook with Chef feels like learning a DSL.
I despise learning DSLs because I'm learning a language that operates at a layer of abstraction above what I actually want to do. Nevertheless, I find cooking with Chef a necessity now and would not want to go back.

If you are like me, and like to experiment, then often things can go fubar.
It is comforting and encouraging knowing that everything can be reconstructed in half an hour.

Despite being conceptually simple, getting started was the biggest obstacle I experienced - and hence the purpose behind this article.


# Chef and Knife
Chef offers a means of declaring a server architecture that is agnostic to the underlying platform.

Its original intention is to cook a farm of servers, with one server acting as the master.
Chef-Solo differs in the way that it does not need a dedicated Chef Server.
One of the quirks between full-blown Chef and Chef-Solo is that configuration files have to be present on your local machine.

Knife is a tool. This part is clear from the metaphor.
Intuitively, I thought that knife was a tool used by Chef to perform the cook.
I was wrong. Knife is a tool that you use to instruct Chef to do its job.
Knife is a nice name, but a bit counter intuitive. "Knife Solo" is the specific version of the tool required for solo cooks.

In addition, before the target node can be cooked, it needs to be prepared by Knife Solo.



# Ruby
You will need Ruby to cook with Chef. RVM is a means to manage different versions of Ruby.
RVM has been criticised for achieving too much, and that RBEnv is more focussed.
However, RVM works for me for now.
I had to have some faith when installing RVM, granting curl root privileges:

{% highlight bash %}
\curl -L https://get.rvm.io | bash -s stable --ruby
    curl -L get.rvm.io | bash -s stable
{% endhighlight %}

The root privileges are required to make changes to openssl. Should I not be a bit more paranoid here?


After it is installed, it has to be activated.

{% highlight bash %}
source /Users/indika/.rvm/scripts/rvm
{% endhighlight %}


Bundler is awesome (according to the community). Use it to manage the gems required for the client side of the Chef.
Use Gem to obtain Bundler.

{% highlight bash %}
gem install bundler
{% endhighlight %}

Now that the Ruby dependencies have been set up, we are ready to create a kitchen.



# Creating a Kitchen

The kitchen is the structure you create on the client side before you cook your server.

## ~/.chef

This is the default location where Knife looks for machine specific settings.
This folder is not on the server, as I initially imagined. This is my *knife.rb* configuration file.

{% highlight ruby %}
knife[:provisioning_path] = "/home/root/solo"

cookbook_path [
    './cookbooks',
    'site-cookbooks']

role_path     "roles"
data_bag_path "data_bags"
encrypted_data_bag_secret "#{ENV['HOME']}/.chef/motion_secret"
{% endhighlight %}


My guess is that this file might be necessary for knife solo, because Chef was built to run on its own dedicated server.


## A workspace for the Kitchen

Create a directory for your kitchen.

### Bundler and the Gemfile

The Ruby library dependencies for Chef are specified in a Gemfile.


{% highlight ruby %}
source 'https://rubygems.org'

gem 'knife-solo'
gem 'knife-solo_data_bag'
gem 'librarian'
gem 'multi_json'
gem 'foodcritic'
{% endhighlight %}


Bundler is used to obtain these dependencies.


{% highlight bash %}
bundle install
{% endhighlight %}


Bundler is considered better than Gem to manage dependencies. I'm not sure why. I do know Bundler can maintain the specific version number required for the application (Chef).
It also obtains the gems required by the required gems, and so forth.
It simply just works.


### Scaffolding


Choose a directory and use Knife to create a scaffolding.

{% highlight bash %}
knife solo init .
{% endhighlight %}

The scaffolding of your kitchen should look like:


{% highlight text %}
├── Gemfile
├── Gemfile.lock
├── cookbooks
│   └── - public cookbooks get loaded in here
├── data_bags
│   └── - secret stuff
├── nodes
│   └── - configuration of particular nodes
├── roles
│   └── - common configurations that can be applied
│         across multiple nodes
└── site-cookbooks
    └── - cookbooks that you have written
{% endhighlight %}


### Librarian-Chef and the Cheffile

Cookbooks contain recipes that instruct Chef how to do stuff.
One of the reasons Chef appeals to me the most is because of the wealth of open source cookbooks available.

I've noticed three different managers for cookbooks:

* Librarian-Chef
* Berkshelf
* knife-github-cookbooks

I started with Librarian-Chef and currently see no good reason to change. Use Librarian-Chef to initialise a Cheffile

{% highlight bash %}
librarian-chef init
{% endhighlight %}





Here is a sample of my Cheffile:

{% highlight ruby %}
site 'http://community.opscode.com/api/v1'

cookbook 'runit'
cookbook 'cron', :git => 'git://github.com/opscode-cookbooks/cron.git'
cookbook 'database', :git => 'git://github.com/opscode-cookbooks/database.git'
cookbook 'logwatch', :git => 'git://github.com/opscode-cookbooks/logwatch.git'
cookbook 'nginx', :git => 'git://github.com/opscode-cookbooks/nginx.git'
cookbook 'python', :git => 'git://github.com/opscode-cookbooks/python.git'
cookbook 'postgresql', :git => 'git://github.com/opscode-cookbooks/postgresql.git'
cookbook 'user', :git => 'git://github.com/fnichol/chef-user.git'
{% endhighlight %}


And then user librarian-chef to fetch the cookbooks:

{% highlight bash %}
librarian-chef install --clean
{% endhighlight %}






### Solo.rb
This seems to be deprecated, since knife-solo v0.3.0.



# Bootstrapping the Server

Before you can cook your server, it has to be prepared for cooking.
In an ideal world, Chef can and will completely determine the architecture of the server. However, when it comes to practicalities, the server still needs to be bootstrapped before it can be cooked. It broadly involves:

- setting up a SSH connection, so that you (meaning the software running on your computer) can securely communicate with it, and
- installing unattended upgrades, and any obviously required packages (I'm not going to be idealistic here on not having dependencies outside the Chef configuration)
- the existence of Chef specific libraries.
- the existence of the secret file to decrypt data bags (more on this later)

## Configuring SSH

SSH is pretty much the standard when it comes to opening a secure communication channel with a server. I'll assume you know how to set it up.

There has been a recent debate on whether it is [a bad idea](http://www.adayinthelifeof.nl/2012/03/12/why-putting-ssh-on-another-port-than-22-is-bad-idea) or
[a good idea](http://www.danielmiessler.com/blog/putting-ssh-another-port-good-idea)
to use a non-default port. I prefer to use a non-default port because I cannot deal with the noise of random hackers.


## Remotely installing Chef


{% highlight bash %}
knife solo prepare root@motion -p 2222

… Setting up chef (11.6.2-1.ubuntu.12.04)
{% endhighlight %}


This installs Ruby, RubyGems and Chef on target machine. Chef needs to be installed on the server


## Create a secret file and SCP it

Vulnerable pieces should be encrypted. More about this later. For now, simply generate a secret key and upload it via SCP.


{% highlight bash %}
openssl rand -base64 512 > ~/.chef/motion_secret
scp -P 2222 ~/.chef/motion_secret root@motion:/root/.chef/motion_secret
{% endhighlight %}




# Nodes

A particular node on your server farm is defined by .json file. I believe that the file must have the same name as its alias in your /etc/hosts. This is a good place to start interrogating Chef.

It appears to be a collection of attributes - data that is specific to the node. The most pertinent attribute is the run list. The run list is where the action happens - it contains a list of recipes that will be applied. The order of which recipes appear in the run_list is preserved during the cook.


# Creating your own Cookbook

Create a cookbook

{% highlight bash %}
knife cookbook create entity -r md
{% endhighlight %}

For some reason, this command places my cookbook, not into the site-cookbooks directory, but into the cookbooks directory. Anyway, it'll look a little bit like this:


{% highlight text %}
entity
├── CHANGELOG.md
├── README.md
├── attributes
├── definitions
├── files
│   └── default
├── libraries
├── metadata.rb
│   └── - information about the author
│       - dependecies to other cookbooks exist here
├── providers
├── recipes
│   └── default.rb
├── resources
└── templates
    └── default

{% endhighlight %}






## Cookbook Dependencies

According to the official documentation:
 *Declaring cookbook dependencies is not required with chef-solo.*
I have a hard time understanding why this is.

There are two types of dependency statements:

- include_recipe 'java'
    - include_recipe "java" or include_recipe "apache2::mod_ssl"
    - These can be found at the top of a recipe
    - the resources found in that recipe will be inserted (in the exact same order) at that point

- depends 'java'
    - This can be found in the metadata of a cookbook
    - I assume that this cookbook may be automatically obtained, however, I wonder how the exact URL is resolved.






<!-- ## Logging

The warning logging works for me, but not the other ones.

- Chef::Log.info('FINDMEFINDME')
- Chef::Log.warn('i warn you')
- Chef::Log.fatal!('something bad')

Perhaps I need to increase the logging verbosity.
 -->



# Databags

Knife-solo uploads cookbooks, roles and data bags onto the target node.
This was not obvious to me at first asI assumed initially that it only sends instructions.
Hence, sensitive data needs to be encrypted before being stored on the target node.

Databags are annoying, yet they feel necessary. Debugging broken bags are slow. However, they seem important and my struggled attempt to justify them are as follows:

- The Chef server could be compromised
- Data bags can be stored in source control without plain text
- Private information can be intercepted between the Chef server and the node.

Nevertheless, I will use them to wire in good security habits.

The basic idea is that keys are stored in .json files, and encrypted versions are sent to the Chef server.

An annoying point to bear in mind is, the id of the data bag must match the name of the file, minus the .json name, otherwise the convention fails.




## Encryption Keys

Use openssl to generate an encryption key.

{% highlight bash %}

openssl rand -base64 512

{% endhighlight %}



Use this key to encrypt the data bag.

{% highlight bash %}

knife solo data bag create indika mybag
--secret-file /Users/indika/.chef/encrypted_data_bag_secret
--data-bag-path data_bags
-e vim
--json-file data_bags/indika/_mybag.json

{% endhighlight %}


This command takes a secret file, the path to place the bag, a preferred text editor and a source bag. Notice how this creates the data bag structure for me.


{% highlight json %}
{
    "id": "users",
    "users": [
        {
            "user_name": "publish",
            "tag": "publish",
            "comment": "User for publishing content",
            "use_ssh": true,
            "ssh_key": "secret ssh key"
        }
    ]
}
{% endhighlight %}

The databag can be accessed from within the recipe like this:


{% highlight ruby %}

secret = Chef::EncryptedDataBagItem.load_secret("/root/.chef/motion_secret")

entity_users_bag = Chef::EncryptedDataBagItem.load("entity", "users", secret)
entity_users_bag['users'].each do |user_item|
    puts user_item['user_name']
end

{% endhighlight %}




# Roles

Roles are a means of applying common functionality to multiple nodes.
I safely ignored roles for a long time since I was cooking a single node, until I needed to cook logstash.

So, how do I create a role?
Create a role file in the roles folder.

The name of the role is logstash_server.
So the name of the role file is logstash_server.rb

The role file has an identifier, a runlist, default and override attributes.

Roles can be defined using JSON or the Ruby DSL.

I am going to add a recipe to this role, particularly logstash::server

Now how do I apply this role to a node?
and finally, JSON data passed to chef-solo:

The role can then be applied to the node by adding it to the run list

{% highlight json %}

{ "run_list": "role[logstash_server]" }

{% endhighlight %}

Roles are assigned to recipes.
I need to assign the newly created role to the recipe logstash::server


# Cooking

Finally, once the kitchen has been created, and the target node has been prepared, it is ready to be cooked.

{% highlight bash %}

knife solo cook username@host

{% endhighlight %}

All the recipes in the runlist will be cooked in the order provided. It is unlikely that your first cook will be successful. However, every subsequent correction takes you further in building a stable web server - one which is not subjected to stasis.


# Next

Chef is not the only framework cooking a server. Puppet, Ansible and Salt exist.
Soon I will investigate into why Berkshelf is preferred over Librarian.

<!-- LWRPs and understanding attribute precedence seem important. -->

Useful links:

[Essential Cookbook Recipes](http://docs.opscode.com/essentials_cookbook_recipes.html)

[Sample Kitchen](https://github.com/indika/kitchen)

