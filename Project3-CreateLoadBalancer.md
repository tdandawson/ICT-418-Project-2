# Create a Load Balancer

## Set default region and zone for all resources.

First, I went to my gcloud instance to check and confirm the region and zone it was set to.

I then set the default region for all resources:
```
gcloud config set compute/region us-west4
```

Then I set the default zone for all resources:
```
gcloud config set compute/zone us-west4-b
```

## Create multiple web server instances

First, I created a virtual machine www1 in my default zone using the following code:
```
 gcloud compute instances create www1 \
    --zone=Zone \
    --tags-network-lb-tag \
    --machine-type=e2-small \
    --image-family=debian-11 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
      apt-get update
      apt-get install apache2 -y
      service apache2 restart
      echo "
<h3>Web Server: www1</h3>" | tee /var/www/html/index.html'
```

Note: I accidentally typed `tags-network-lb-tag \` instead of `tags=network-lb-tag \` so I had to go back and redo with the correct syntax:
```
gcloud compute instances create www1 \
    --zone=Zone \
    --tags=network-lb-tag \
    --machine-type=e2-small \
    --image-family=debian-11 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
      apt-get update
      apt-get install apache2 -y
      service apache2 restart
      echo "
<h3>Web Server: www1</h3>" | tee /var/www/html/index.html'
```

Second, I created a virtual machine www2 in my default zone using the following code:
```
gcloud compute instances create www2 \
    --zone=Zone \
    --tags=network-lb-tag \
    --machine-type=e2-small \
    --image-family=debian-11 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
      apt-get update
      apt-get install apache2 -y
      service apache2 restart
      echo "
<h3>Web Server: www2</h3>" | tee /var/www/html/index.html'
```

Third, I created a virtual machine www3 in my default zone using the following code:
```
  gcloud compute instances create www3 \
    --zone=Zone  \
    --tags=network-lb-tag \
    --machine-type=e2-small \
    --image-family=debian-11 \
    --image-project=debian-cloud \
    --metadata=startup-script='#!/bin/bash
      apt-get update
      apt-get install apache2 -y
      service apache2 restart
      echo "
<h3>Web Server: www3</h3>" | tee /var/www/html/index.html'
```

I then created a firewall rule to allow external traffic to the VM instance:
```
gcloud compute firewall-rules create www-firewall-network-lb \
    --target-tags network-lb-tag --allow tcp:80
```

I ran the following command to list my instances:
```
gcloud compute instances list
```

Finally, I verified that each instance was running with `curl` for each VM:
```
curl http://[IP_ADDRESS]
```

## Configure the load balancing server

First, I created a static external IP address for my load balancer:
```
gcloud compute addresses create network-lb-ip-1 \
  --region us-west4
```

Second, I added a legacy HTTP help check resource:
```
gcloud compute http-health-checks create basic-check
```

Third, I added a target pool in the same region as my instances:
```
gcloud compute target-pools create www-pool \
> --instances www1,www2,www3
```

NOTE: I received an error of `ERROR: (gcloud.compute.target-pools.create) unrecognized arguments:
  --instances
  www1,www2,www3` and I thought it was because I used `--instances` instead of `--instance` so I ran it again with `--instance`:
```
gcloud compute target-pools create www-pool \
> --instance www1,www2,www3
```

However, I then realized that I was mixing up the codes from steps 3 and 4 in the Task 3 section of our notes.

After realizing my mistake I used the correct command to add a target pool in the same region as my instances:
```
gcloud compute target-pools create www-pool \
--region us-west4 --http-health-check basic-check
```

After successfully adding a target pool, I added the instances to the pool:
```
gcloud compute target-pools add-instances www-pool \
> --instances www1,www2,www3
```

Finally, after working through my mistakes, I could add a forwarding rule:
```
gcloud compute forwarding-rules create www-rule \
> --region us-west4 \
> --ports 80 \
> --address network-lb-ip-1 \
> --target-pool www-pool
```

## Sending traffic to my instances

Now that the load balancing service is configured, I could start sending traffic to the forwarding rule and watch the traffic be dispersed to different instances.

First, I entered the following command to view the external IP address of the www-rule forwarding rule used by the loader:
```
gcloud compute forwarding-rules describe www-rule --region us-west4
```

Second, I tried to access the external IP address:
```
 IPADDRESS=$(gcloud compute forwarding-rules describe www-rule --region us-west4 --format="json" | jq -r .IPAddress)
```

NOTE: I ran into an error of `zsh: command not found: jq` which led me down a rabbit hole of errors and mistakes.

I will show the commands I entered and the errors I received below (Skip to the bottom of the code block for a short summary):
```
tristandawson@Tristans-MBP-2 ~ % sudo apt-get update
Password:
sudo: apt-get: command not found
tristandawson@Tristans-MBP-2 ~ % sudo brew update
Error: Running Homebrew as root is extremely dangerous and no longer supported.
As Homebrew does not drop privileges on installation you would be giving all
build scripts full access to your system.
tristandawson@Tristans-MBP-2 ~ % apt-get update
zsh: command not found: apt-get
tristandawson@Tristans-MBP-2 ~ % sudo brew install jq
Error: Running Homebrew as root is extremely dangerous and no longer supported.
As Homebrew does not drop privileges on installation you would be giving all
build scripts full access to your system.
tristandawson@Tristans-MBP-2 ~ % cd /
tristandawson@Tristans-MBP-2 / % sudo brew install jq
Error: Running Homebrew as root is extremely dangerous and no longer supported.
As Homebrew does not drop privileges on installation you would be giving all
build scripts full access to your system.
tristandawson@Tristans-MBP-2 / % sudo apt update
The operation couldn’t be completed. Unable to locate a Java Runtime.
Please visit http://www.java.com for information on installing Java.

tristandawson@Tristans-MBP-2 / % cd
tristandawson@Tristans-MBP-2 ~ % sudo apt update
The operation couldn’t be completed. Unable to locate a Java Runtime.
Please visit http://www.java.com for information on installing Java.

tristandawson@Tristans-MBP-2 ~ % sudo apt install jq
The operation couldn’t be completed. Unable to locate a Java Runtime.
Please visit http://www.java.com for information on installing Java.

tristandawson@Tristans-MBP-2 ~ % sudo chown -R "$USER":admin /usr/local
chown: /usr/local: Operation not permitted
tristandawson@Tristans-MBP-2 ~ % sudo chown -R "$USER":admin /Library/Caches/Homebrew
chown: /Library/Caches/Homebrew: No such file or directory
tristandawson@Tristans-MBP-2 ~ % sudo apt install jq
The operation couldn’t be completed. Unable to locate a Java Runtime.
Please visit http://www.java.com for information on installing Java.

tristandawson@Tristans-MBP-2 ~ % sudo dnf install jq
Password:
sudo: dnf: command not found
tristandawson@Tristans-MBP-2 ~ % sudo install jq
usage: install [-bCcpSsv] [-B suffix] [-f flags] [-g group] [-m mode]
               [-o owner] file1 file2
       install [-bCcpSsv] [-B suffix] [-f flags] [-g group] [-m mode]
               [-o owner] file1 ... fileN directory
       install -d [-v] [-g group] [-m mode] [-o owner] directory ...
tristandawson@Tristans-MBP-2 ~ % git -C "/usr/local/Homebrew/Library/Taps/homebrew/homebrew-cask" fetch --unshallow
fatal: cannot change to '/usr/local/Homebrew/Library/Taps/homebrew/homebrew-cask': No such file or directory
tristandawson@Tristans-MBP-2 ~ % sudo apt install jq
The operation couldn’t be completed. Unable to locate a Java Runtime.
Please visit http://www.java.com for information on installing Java.

tristandawson@Tristans-MBP-2 ~ % sudo apt install jq
The operation couldn’t be completed. Unable to locate a Java Runtime that supports apt.
Please visit http://www.java.com for information on installing Java.

tristandawson@Tristans-MBP-2 ~ % which java
/usr/bin/java
tristandawson@Tristans-MBP-2 ~ % java -version
java version "1.8.0_391"
Java(TM) SE Runtime Environment (build 1.8.0_391-b13)
Java HotSpot(TM) 64-Bit Server VM (build 25.391-b13, mixed mode)
tristandawson@Tristans-MBP-2 ~ % sudo systemctl status java
sudo: systemctl: command not found
tristandawson@Tristans-MBP-2 ~ % sudo install jq
usage: install [-bCcpSsv] [-B suffix] [-f flags] [-g group] [-m mode]
               [-o owner] file1 file2
       install [-bCcpSsv] [-B suffix] [-f flags] [-g group] [-m mode]
               [-o owner] file1 ... fileN directory
       install -d [-v] [-g group] [-m mode] [-o owner] directory ...
tristandawson@Tristans-MBP-2 ~ % sudo apt install jq                                 
The operation couldn’t be completed. Unable to locate a Java Runtime that supports apt.
Please visit http://www.java.com for information on installing Java.

tristandawson@Tristans-MBP-2 ~ % IPADDRESS=$(gcloud compute forwarding-rules describe www-rule --region us-west4 --format="json" | jq -r .IPAddress)
zsh: command not found: jq
WARNING:  Python 3.5-3.7 will be deprecated on August 8th, 2023. Please use Python version 3.8 and up.

If you have a compatible Python interpreter installed, you can use it by setting
the CLOUDSDK_PYTHON environment variable to point to it.

tristandawson@Tristans-MBP-2 ~ % ls
Applications		Movies			bin
Desktop			Music			google-cloud-sdk
Documents		Pictures		learn-the-commandline
Downloads		Public
Library			Zotero
tristandawson@Tristans-MBP-2 ~ % cd /
tristandawson@Tristans-MBP-2 / % ;s
zsh: command not found: s
tristandawson@Tristans-MBP-2 / % 's
quote> ls
quote> exit
quote> \q
quote> q
quote> '
zsh: command not found: s\nls\nexit\n\q\nq\n
tristandawson@Tristans-MBP-2 / % ls
Applications	Volumes		etc		sbin
Library		bin		home		tmp
System		cores		opt		usr
Users		dev		private		var
tristandawson@Tristans-MBP-2 / % cd bin
tristandawson@Tristans-MBP-2 /bin % ls
[		dd		launchctl	pwd		tcsh
bash		df		link		realpath	test
cat		echo		ln		rm		unlink
chmod		ed		ls		rmdir		wait4path
cp		expr		mkdir		sh		zsh
csh		hostname	mv		sleep
dash		kill		pax		stty
date		ksh		ps		sync
tristandawson@Tristans-MBP-2 /bin % cd zsh
cd: not a directory: zsh
tristandawson@Tristans-MBP-2 /bin % cd
tristandawson@Tristans-MBP-2 ~ % ls .zshrc
.zshrc
tristandawson@Tristans-MBP-2 ~ % which java
/usr/bin/java
tristandawson@Tristans-MBP-2 ~ % nano .zshrc
tristandawson@Tristans-MBP-2 ~ % echo $JAVA_HOME

tristandawson@Tristans-MBP-2 ~ % IPADDRESS=$(gcloud compute forwarding-rules describe www-rule --region us-west4 --format="json" | jq -r .IPAddress)
zsh: command not found: jq
WARNING:  Python 3.5-3.7 will be deprecated on August 8th, 2023. Please use Python version 3.8 and up.

If you have a compatible Python interpreter installed, you can use it by setting
the CLOUDSDK_PYTHON environment variable to point to it.

tristandawson@Tristans-MBP-2 ~ % sudo apt install jq
Password:
The operation couldn’t be completed. Unable to locate a Java Runtime that supports apt.
Please visit http://www.java.com for information on installing Java.
```

Summary:

After receiving the first `zsh: command not found: jq` error, I tried installing jq using `apt-get`. 
I tried over and over using the commands I was finding online until I found out that Mac doesn't support apt-get.
I then tried using `brew` instead but was running into errors with that as well. After a deep dive of mistakes on the
internet, I found out that I could in fact use `brew` to install `jq`, but I was using `sudo brew install jq` instead 
of just `brew install jq` and if you could not tell from the error messages above like I couldn't, running Homebrew
as root is extremely dangerous and no longer supported. Long story short, I tried a bunch of repetitive, unnecessary
commands and got so frustrated I couldn't see what was right in front of me!


Finally, I got jq installed using:
```
brew install jq
```

Now that jq was installed, I could finally access the external IP address using the following command:
```
IPADDRESS=$(gcloud compute forwarding-rules describe www-rule --region us-west4 --format="json" | jq -r .IPAddress)
```

I tried showing the external IP address with:
```
echo $(IPADDRESS)
```

However, I used the wrong command and was not supposed to use `()` in the command.
So in order to successfully show the external IP address, I used:
```
echo $IPADDRESS
```

I then checked to make sure that I had curl:
```
which curl
```

After verifying that I did, I used the `curl` command to access the external IP address, replacing **IP_ADDRESS** with an external IP address from the previous command:
```
while true; do curl -m1 $IPADDRESS; done
```

## Create an HTTP load balancer

First, I created the load balancer template:
```
gcloud compute instance-templates create lb-backend-template \
> --region=us-west4 \
> --network=default \ 
> --subnet=default \
> --tags=allow-health-check \
> --machine-type=e2-medium \
> --image-family=debian-11 \
> --image-project=debian-cloud \
> --metadata=startup-script='#!/bin/bash
quote> apt-get update
quote> apt-get install apache2 -y
quote> a2ensite default-ssl
quote> a2enmod ssl
quote> vm_hostname="$(curl -H "Metadata-Flavor:Google" \
quote> http://169.254.169.254/computeMetadata/v1/instance/name)"
quote> echo "Page served from: $vm_hostname" | \
quote> tee /var/www/html/index.html
quote> systemctl restart apache2'
```

After runnin this command, I was prompted that there were updates available for some Google CLoud CLI components, so I ran the following command to update them:
```
gcloud components update
```

After updating, I tried to create a managed instance group based on the templates:
```
gcloud compute instance-groups managed create lb-backend-group \
--template=lb-backend-template --size=2 --zone=us-west4-b
```

This resulted in me being met with the following error:
```
ERROR: gcloud failed to load. You are running gcloud with Python 3.7, which is no longer supported by gcloud.
Install a compatible version of Python 3.8-3.12 and set the CLOUDSDK_PYTHON environment variable to point to it.
```

Having just done a components update, I thought that the update might have something to do with the error I was receiving, so I tried reverting to the previously installed version:
```
gcloud components update --version 445.0.0
```

However, I recieved the same error message. This resulted in me doing some research on the error message which led to me changing some files and directories:
```
which python3
```
I tried using what was recommended on a StackOverflow page to fix the problem:
```
export CLOUDSDK_PYTHON=/usr/local/bin/python3.10
```

After trying to run the `gcloud compute instance-groups managed create lb-backend-group \
--template=lb-backend-template --size=2 --zone=us-west4-b` again a different error, I decided to check which version of Python I had installed:
```
python3 --version
```

For some reason, I then reinstalled Python:
```
brew install python
```

After trying the previous `gcloud` command again and receiving the same error, I used the following commands to look at my bin folder:
```
cd /usr/local/bin
ls
```

This is when I realized that in my previous attempt, I used `export CLOUDSDK_PYTHON=/usr/local/bin/python3.10`, which didn't work because the correct path for me was `/ur/local/bin/python3.11`.

After noticing this, I ran the following command:
```
export CLOUDSDK_PYTHON=/usr/local/bin/python3.11
```

I then tried our gcloud command again which finally ended up successfully creating a managed instance group based on the templates:
```
gcloud compute instance-groups managed create lb-backend-group \
--template=lb-backend-template --size=2 --zone=us-west4-b
```

After finally being successful with the previous command, I created the `fw-allow-health-check` firewall rule:
```
gcloud compute firewall-rules create fw-allow-health-check \
--network=default \
--action=allow \
--direction=ingress \
--source-ranges=130.211.0.0/22,35.191.0.0/16 \
--target-tags=allow-health-check \
--rules=tcp:80
```

Now that the instances are up and running, I set up a global static external IP address that customers use to reach the load balancer:
```
gcloud compute addresses create lb-ipv4-1 \
> --ip-version=IPV4 \
> --global
```

**Note the IPv4 address that was reserved:**
```
gcloud compute addresses describe lb-ipv4-1 \
> --format="get(address)" \
> --global
```

I then created a health check for the load balancer:
```
gcloud compute health-checks create http http-basic-check \
> --port80
```

NOTE: I mistakenly entered `--port80` in the previous command instead of `--port 80`, so I reentered the command with the correct syntax:
```
gcloud compute health-checks create http http-basic-check \
> --port 80
```

I then created a backend service:
```
gcloud compute backend-services create web-backend-service \
> --protocol=HTTP \
> --port-name=http \
> --health-checks=http-basic-check \
> --global
```

After creating the backend service, I added my instance group as the beackend to the backend service:
```
gcloud compute backend-services add-backend web-backend-service \
> --instance-group=lb-backend-group \
> --instance-group-zone=us-west4-b \
> --global
```

I then created a URL map to route the incoming requests to the default backend service:
```
gcloud compute url-maps create web-map-http \
> --default-service web-backend-service
```

I created a target HTTP proxy to route requests to my URL map:
```
gcloud compute target-http-proxies create http-lb-proxy \
> --url-map web-map-http
```

Finally, I tried to create a global forwarding rule to route incoming requests to the proxy:
```
gcloud compute forwarding-rules create http-content-rule \
> --address=lb-ipv4-1\
> --global \
> --target-http-proxy=http-lb-proxy \
> --ports=80
```

NOTE: I have gotten stuck here and connot figure out what to do. I will attach what I have tried so far with the error messages I have received:
```
gcloud compute forwarding-rules create http-content-rule \
> --address=lb-ipv4-1\
> --global \
> --target-http-proxy=http-lb-proxy \
> --ports=80
ERROR: (gcloud.compute.forwarding-rules.create) Could not fetch resource:
 - Invalid value for field 'resource.target': 'https://compute.googleapis.com/compute/v1/projects/sysadmin-238/global/targetHttpProxies/http-lb-proxy'. Invalid target type TARGET_HTTP_PROXY for forwarding rule in scope REGION

tristandawson@Tristans-MBP-2 ~ % gcloud config list zone
ERROR: (gcloud.config.list) Section [core] has no property [zone].
tristandawson@Tristans-MBP-2 ~ % gcloud compute forwarding-rules create http-content-rule \
--address=lb-ipv4-1\
--global \
--target-http-proxy=http-lb-proxy \
--ports=80
^[[DERROR: (gcloud.compute.forwarding-rules.create) Could not fetch resource:
 - Invalid value for field 'resource.target': 'https://compute.googleapis.com/compute/v1/projects/sysadmin-238/global/targetHttpProxies/http-lb-proxy'. Invalid target type TARGET_HTTP_PROXY for forwarding rule in scope REGION

tristandawson@Tristans-MBP-2 ~ % gcloud compute forwarding-rules create http-content-rule \
--address=lb-ipv4-1\
--global \
--region=us-west4 \--target-http-proxy=http-lb-proxy \
--ports=80  
ERROR: (gcloud.compute.forwarding-rules.create) Could not fetch resource:
 - Invalid value for field 'resource.target': 'https://compute.googleapis.com/compute/v1/projects/sysadmin-238/global/targetHttpProxies/http-lb-proxy'. Invalid target type TARGET_HTTP_PROXY for forwarding rule in scope REGION

tristandawson@Tristans-MBP-2 ~ % gcloud confid set compute/region us-west4
ERROR: (gcloud) Invalid choice: 'confid'.
Maybe you meant:
  gcloud resource-settings set-value
  gcloud compute snapshot-settings describe
  gcloud compute snapshot-settings update
  gcloud compute forwarding-rules set-target
  gcloud compute backend-services set-iam-policy
  gcloud compute instance-templates set-iam-policy
  gcloud compute url-maps set-default-service
  gcloud compute disks set-iam-policy
  gcloud compute images set-iam-policy
  gcloud compute instance-groups set-named-ports

To search the help text of gcloud commands, run:
  gcloud help -- SEARCH_TERMS
tristandawson@Tristans-MBP-2 ~ % gcloud config set compute/region us-west4
Updated property [compute/region].
tristandawson@Tristans-MBP-2 ~ % gcloud compute forwarding-rules create http-content-rule \
--address=lb-ipv4-1\
--global \
--target-http-proxy=http-lb-proxy \                   
--ports=80
ERROR: (gcloud.compute.forwarding-rules.create) Could not fetch resource:
 - Invalid value for field 'resource.target': 'https://compute.googleapis.com/compute/v1/projects/sysadmin-238/global/targetHttpProxies/http-lb-proxy'. Invalid target type TARGET_HTTP_PROXY for forwarding rule in scope REGION
```
