# Ngnix-CookBooks
A step by step approach to configuring cookbooks to spin up ngnix on Chef nodes 
knife cookbook create nginx


knife cookbook create apt


vi ~/chef-repo/cookbooks/nginx/recipes/default.rb
					
cd cookbooks/nginx

cd recipes

vi default.rb


include_recipe "apt"

package 'nginx' do
  action :install
end

service 'nginx' do
  action [ :enable, :start ]
end

cookbook_file "/usr/share/nginx/www/index.html" do
  source "index.html"
  mode "0644"
end

Index File
cd ~/chef-repo/cookbooks/nginx/files/default

vi index.html

<html>
  <head>
    <title>Hello there</title>
  </head>
  <body>
    <h1>This is a test</h1>
    <p>I hope this works!</p>
  </body>
</html>


Metadata.rb file
This file is checked when the Chef server sends the run-list to the node, to see which other recipes should be added to the run-list.
vi ~/chef-repo/cookbooks/nginx/metadata.rb
At the bottom of the file, add these lines:
name             'nginx'
maintainer       'YOUR_COMPANY_NAME'
maintainer_email 'YOUR_EMAIL'
license          'All rights reserved'
description      'Installs/Configures nginx'
long_description IO.read(File.join(File.dirname(__FILE__), 'README.md'))
version          '0.1.0'

depends "apt"

With this, our apt cookbook will update the package database of our Nginx cookbook.

Adding the Cookbook to our Node
We can now upload the cookbooks to our Chef server from our workstation either by:
a.individually typing:
	knife cookbook upload apt
	knife cookbook upload nginx
or, 

b. upload everything by typing:
	knife cookbook upload –a

 Now, we can modify the run-list  by typing:
knife node edit name_of_node
To see the  available nodes, you can type:
knife node list
client1
For our purposes, when we type this, we get a file that looks like this:
knife node edit client1
{
  "name": "client1",
  "chef_environment": "_default",
  "normal": {
    "tags": [

    ]
  },
  "run_list": [

  ]
}
You may need to set your EDITOR environmental variable for this to work. You can do this by typing:
export EDITOR=name_of_editor
This simple JSON document describes some aspects of our node. We can see a "run_list" array, which is currently empty.
We can add our Nginx cookbook to that array using the format:
"recipe[name_of_recipe]"
When we are finished, our file should look like this:
{
  "name": "client1",
  "chef_environment": "_default",
  "normal": {
    "tags": [

    ]
  },
  "run_list": [
    "recipe[nginx]"
  ]
}
Save and close the file to implement the new settings.
Now, we can SSH into our node and run the Chef client command. This will cause the client to check into the Chef server and it will see the new run list.
sudo chef-client
Starting Chef Client, version 11.8.2
resolving cookbooks for run list: ["nginx"]
Synchronizing Cookbooks:
  - apt
  - nginx
Compiling Cookbooks...
Converging 4 resources
Recipe: apt::default
  * execute[apt-get update] action run
    - execute apt-get update

Recipe: nginx::default
  * package[nginx] action install (up to date)
  * service[nginx] action enable
    - enable service service[nginx]

  * service[nginx] action start (up to date)
  * cookbook_file[/usr/share/nginx/www/index.html] action create (up to date)
Chef Client finished, 2 resources updated
Our apt cookbook is sent over and run as well even though it is not in the run-list we created. That is because Chef intelligently resolves dependencies and modifies the actual run-list before executing it on the node.
We can verify that this works by going to our node's IP address or domain name:
http://node_domain_or_IP
You should see something that looks like this:
This is a test
I hope this works!
