# Digital Ocean Rails Droplet Deploy(Easy Way)

This is intended to be a quick and easy guide for setting up a rails app on a D.O. rails application droplet. This guide avoids dealing with any sort of unicorn or nginx configuration. When you create the droplet be sure to add your ssh key. 
It is encouraged to use the figaro gem to handle environment variables as it allows for easy configuration.

##App setup:

Configure the databse.yml file in your app for production (replace  example_app_name_production)


```
production:
  <<: *default
  database: example_app_name_production
  username: rails
  password: <%= ENV['APP_DATABASE_PASSWORD'] %>


```

##Droplet Setup:

SSH into your server: 

```
ssh root@example_ip_here
```

##Swapfile Setup
First lets make a swapfile to handle memory issues.
This is not completly necessary at first but if you have memory issues and do not want to upgrade it may be a good temporary solution. You can paste the entire script below, here is a summary:



* Make the swapfile
* Change the swap permissions
* Use the swapfile
* Make the swapfile persistent
* Change the swappiness and cache
* Make the changes persistent
 you can also see more in depth instructions [here](https://www.digitalocean.com/community/tutorials/how-to-add-swap-on-ubuntu-14-04)

```
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo "/swapfile   none    swap    sw    0   0" | sudo tee -a /etc/fstab
sudo sysctl vm.swappiness=10
sudo sysctl vm.vfs_cache_pressure=50
echo vm.swappiness=10 | sudo tee -a /etc/sysctl.conf
echo vm.vfs_cache_pressure=50 | sudo tee -a /etc/sysctl.conf
```


***
Take note of the rails user password from the welcome banner/login screen.

Run Updates and install important ruby dependencies that may be missing.. This may take a while.

```
apt-get update
apt-get install ruby-dev zlib1g-dev --yes
apt-get install git -- yes
gem update
```

While updates are running you can open another shell instance and copy your ssh key to the rails user( you may need to scroll up to look for the password -- the credentials are listed as sftp)

```
cat ~/.ssh/id_rsa.pub | ssh rails@example_ip_here "mkdir -p ~/.ssh && cat >>  ~/.ssh/authorized_keys"

```

Once updates are complete in shell window one... 

The droplet by default has a "rails" user and postgresss "rails" user. Give the rails user sudo privelages:

```
adduser rails sudo
```

Trash the example project server.

```
rm -rf /home/rails/rails_project/*
```

Export the databse password to the bash profile. You may need to scroll up to when you first logged in to see the password listed under “postgres database credentials” (you will also need your rails sftp password for sudo)

```
sudo echo "export APP_DATABASE_PASSWORD=example_rails_pg_user_password" >> /home/rails/.profile
```
Make sure that the password has been succesfully exported-- you should see the password after running this command.
```
cat /home/rails/.profile
```

##Clonning the Repo


The following commands should be run as rails user:

```
su - rails
```

Clone your repo with ssh:

```
git clone example.project.ssh.url.here
```

move the files into the rails_project directory:

```
rsync -a example.project.folder.name.here/ ./rails_project/
```

remove the repo (you have moved it in the previous step)


```
rm -rf example.project.folder.name.here
```

cd into your project folder

```
cd example.project.folder.name.here
```

##Final Setup

run rake secret to generate a secret token for verifying cookies:

```
rake secret
```

export this key to the bash profile:

```
echo "export SECRET_KEY_BASE=example.secret.token" >> /home/rails/.profile
```

bundle install
```
bundle install --path vendor/bundle
```

Set up rails db

```
RAILS_ENV=production rake db:create && rake db:migrate && rake db:seed
```

Precompile assets and clean

```
RAILS_ENV=production bundle exec rake assets:precompile assets:clean
```

###Figaro setup (skip if not necessary)
At this point if you have any other environment variables such as api keys run the figaro gem install
```
bundle exec figaro install
```
if you need to edit application yml ...
```
nano config/application.yml
```
***


##Restart your app
exit the rails user: 

```
exit
```

restart unicorn:

```
service unicorn restart
``` 

navigate to your droplet ip adress and you should be good to go.


##updating your app

ssh in and stop unicorn(you may not need to do this but oftentimes it can help with memory usage)

```
ssh root@example.ip.here
```

```
service unicorn stop
```

switch to rails user and enter project file

```
su - rails
cd rails_project
```

git pull:

```
git pull origin master
```

run bundle:

```
bundle
```

migrate if necessary:

```
RAILS_ENV=production rake db:migrate
```

precompile and clean assets

```
RAILS_ENV=production bundle exec rake assets:precompile assets:clean
```

exit and restart unicorn:

```
exit
service unicorn restart
```


Special Thanks to Kenaniah Cerny for providing the foundation of this guide.








