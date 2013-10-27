---
layout: post
title: Cooking a Web-Server with Chef-Solo
description: "Cooking a Web-Servers with Chef-Solo"
modified: 2013-10-25
tags: chef, publishing
comments: false
share: true
---


This is a recipe on how to use Chef recipes.
This is my Ruby naive understanding of Chef.

Cooking with Chef feels like learning a DSL, and while I despise learning DSLs because of the shortness of their lifespans, I find cooking with Chef an necessity when it comes to managing a server.

The advantages is that if your server goes FOOBAR, you can easily start from scratch and get up and running again in 20minutes.

Statis is bad.

# Concepts
Cooking is simple, however, the concepts and the roles they play were the biggest stumbling block I experienced.


Chef was built to distribute to multiple nodes
Chef-Solo does not need a dedicated Chef Server
Chef-Solo is a handy means to putting up a web server. There are a few quirks between full blown Chef and Chef-Solo.


Knife is a tool, and it is what you will use to instruct Chef how to do it's job. Knife is a nice name, but a bit counter intuitive.

Knife Solo is what you will use to perform solo cooks.

The server needs to be prepared before you can cook



# Ruby
You will need Ruby. Chef is written in Ruby.

Use RVM to manage different versions of Ruby.


Have some faith and get a stable version of ruby:


{% highlight bash %}
\curl -L https://get.rvm.io | bash -s stable --ruby
    curl -L get.rvm.io | bash -s stable
{% endhighlight %}



It then asks you for your root password so that it can make changes to openssl.

And then you have to source it.

{% highlight bash %}
source /Users/indika/.rvm/scripts/rvm
{% endhighlight %}



Use Gem to install Bundler

{% highlight bash %}
gem install bundler
{% endhighlight %}

    


Create a gem file

    bundle install

Create a Cheffile

    librarian-chef install --clean


# The Kitchen

## ~/.chef

This is the default location where Chef looks for machine specific settings. This folder is not on the server.

Create a knife.rb configuration file. Mine was simple until I upgrading knife-solo.

    knife[:provisioning_path] = "/home/root/solo"
    
    cookbook_path [
        "../common-cookbooks",
        './cookbooks',
        "site-cookbooks"]
    
    role_path     nil
    data_bag_path "data_bags"
    encrypted_data_bag_secret "#{ENV['HOME']}/.chef/motion_secret"
    
This is necessary for knife solo because Chef was built to run on it's own dedicated server.





### Gemfile

Use a Gemfile to specify the dependencies for Chef.

    source 'https://rubygems.org'

    gem 'knife-solo'
    gem 'knife-solo_data_bag'
    gem 'librarian'
    gem 'multi_json'
    gem 'foodcritic'

And then use Bundler to obtain these dependencies.

    bundle install

I'm not sure exactly why Bundler is better than Gem to manage dependencies.
But I do know Bundler can maintain the specific version number required for the application (Chef).
And it also obtains the gems required by the required gems, and so forth.
And I do know that it simply just work.


#### A workspace for the Kitchen

Choose a directory and use Knife to create a scaffolding
    knife solo init .




### Cheffile

Cookbooks contain recipes that instruct Chef how to do stuff.
One of the reasons Chef appeals to me the most is because of the wealth of open source cookbooks available.

I've noticed three different managers for cookbooks
- Librarian-Chef
- Berkshelf
- knife-github-cookbooks

I started with Librarian-Chef and currently see no good reason to change.

Create the scaffolding for a Cheffile

    librarian-chef init



This is my Cheffile

    site 'http://community.opscode.com/api/v1'

    cookbook 'runit'
    cookbook 'cron', :git => 'git://github.com/opscode-cookbooks/cron.git'
    cookbook 'database', :git => 'git://github.com/opscode-cookbooks/database.git'
    cookbook 'logwatch', :git => 'git://github.com/opscode-cookbooks/logwatch.git'
    cookbook 'nginx', :git => 'git://github.com/opscode-cookbooks/nginx.git'
    cookbook 'phantomjs', :git => 'git://github.com/customink-webops/phantomjs.git'
    cookbook 'python', :git => 'git://github.com/opscode-cookbooks/python.git'
    cookbook 'postgresql', :git => 'git://github.com/opscode-cookbooks/postgresql.git'
    cookbook 'user', :git => 'git://github.com/fnichol/chef-user.git'


## Structure of the Kitchen

Your kitchen should look like:

    .
    ├── Gemfile
    ├── Gemfile.lock
    ├── cookbooks
    │   └── - public cookbooks get loaded in here
    │       -
    ├── data_bags
    ├── nodes
    │   └── - server configuration files
    │       -
    ├── roles
    │   └── - server configuration files
    │       - i don't know what these are yet
    └── site-cookbooks
    │   └── - cookbooks that you have written
            -

### Solo.rb
This seems to be deprecated.
solo.rb found, but since knife-solo v0.3.0 it is not used any more



# Bootstrapping the Server

In the ideal world, Chef can and will completely determine the architecture of the server. However, when it comes to practicalities, the server still needs to be bootstrapped before it can be cooke. It broadly involves:

- setting up a SSH connection, so that you  (meaning the software running on your computer) can securely communicate with it, and
- installing stuff like unattended upgrades, and any obviously required packages (I'm not going to be idealistic here on not having dependencies outside of the Chef configuration)
- the existence of Chef specific libraries.
- the existence of the secret file to decrypt data bags (more on this later)

## Configuring SSH

SSH is pretty much the standard when it comes to open a communication channel with a server. I'll assume you know how to set it up. Using a non-default SSH port is a good idea to deter random hackers.


## Remotely installing Chef

    knife solo prepare root@motion -p 2222

    … Setting up chef (11.6.2-1.ubuntu.12.04)

This installs Ruby, RubyGems and Chef on target mach. Chef needs to be installed on the server


## Create a secret file and SCP it

Vulnerable pieces should be encrypted. More about this listen. For now, simply generate a secret key and upload it via SCP.

    openssl rand -base64 512 > ~/.chef/motion_secret
    scp -P 2222 ~/.chef/motion_secret root@motion:/root/.chef/motion_secret



# Nodes

The .json configuration files is where the definition of a node (node) exists.
This is a good place to start interrogating Chef.

It contains of a bunch of node specific data. The most pertinent is the run list. The run list is where the action happens, and it appears the order of items in the list, is the order in which the action happens.


# Structure of a Cookbook

Create a cookbook

Then create a cookbook
    knife cookbook create ninjacircles -r md


    entity
    ├── CHANGELOG.md
    ├── README.md
    ├── attributes
    ├── definitions
    ├── files
    │   └── default
    ├── libraries
    ├── metadata.rb
    │   └── - information about the creator
    │       - has some depends statements
    ├── providers
    ├── recipes
    │   └── default.rb
    ├── resources
    └── templates
        └── default




## Includes and stuff like this

include recipe[rvm::user] in your run_list and add a user hash to the user_installs attribute list.

eg

    node['rvm']['user_installs'] = [
      { 'user'          => 'wiggle bottom',
        'default_ruby'  => 'rbx',
        'rubies'        => ['1.9.2', '1.8.7']
      }
    ]





# Databags

Knife solo uploads cookbooks, roles and data bags onto the target node.
This was not obvious to me at first.
It was possible that it would only sending instructions.

Databags are annoying, yet feel necessary. Errors only become apparent during the cook. However, they seem important and my struggled attempt to justify them are as follows:

- The Chef server could be compromised
- Can manage data bags in source control without plain text
- Perhaps the reason behind data bags is because private information can be intercepted between the Chef server and the node.

Nevertheless, I will use them to wire good security habit.

The basic idea is that keys are stored in .json files and encrypted versions are sent to the Chef server.

An annoying point to bear in mind is, the ID of the data bag must match the name of the file, minus the .json name. Otherwise, the convention fails!


## Encryption Keys

Use openssl to generate an encryption key.

    openssl rand -base64 512

And use this key to encrypt the data bag.

    knife solo data bag create indika bob
    --secret-file /Users/indika/.chef/encrypted_data_bag_secret
    --data-bag-path data_bags
    -e vim
    --json-file data_bags/indika/_bob.json

- It takes a secret file.
- A path to place the bag
- A preferred editor
- And a source bag

Notice how this creates the structure for me. And I fully specify the source.


This is the strategy:
Have a folder called data\_bags\_plain

Inside there, have an example folder

Now, there are two concepts,
the folder, and the file name.

I suppose the first name refers to the site-cookbook - well it could.

    secret = Chef::EncryptedDataBagItem
    .load_secret("/root/.chef/motion_secret")

    item = Chef::EncryptedDataBagItem.load("indika", "bob", secret)
    item = data_bag_item('indika', 'bob')
    puts item['password']
    url = "https://#{item['password']}/App.config"


# Roles


Can define using JSON or the Ruby DSL.
Don't really need them.



## My config

The way I have set things up is that I have private and example data bags. And I can switch between them with a symbolic link. This is so that I can share this repository on github.


## Code Snippets

Syntax highlighting via Pygments

{% highlight css %}
#container {
  float: left;
  margin: 0 -240px 0 0;
  width: 100%;
}
{% endhighlight %}

Non Pygments code example

    <div id="awesome">
        <p>This is great isn't it?</p>
    </div>



# Next


Somethings are harder to cook than others.
The Ruby community seems to be very splintered.


Ansible and Salt exist. So does Puppet.
What about OpenStack. People talk about OpenStack on OpenStack.

Chef for cooking a server.
And Docker for distributing programs.

Berkshelf instead of Librarian.
