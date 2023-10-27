# Installation of Jenkins and plugins via user data

In this section, we will learn how to install Jenkins and the plugins via user data.

This phase includes:

   * How to install Jenkins using user data on a Debian 
     instance.
   * How to unlock Jenkins using a script.
   * Steps to create a Jenkins admin user by using user data.
   * Steps to install recommended plugins using a script.

Before making the hands dirty, let's have 
a short introduction about Jenkins.

## What is Jenkins?
 * Jenkins is a self-contained, open-source automation server that can be used to automate all sorts of tasks related to building, testing, and delivering or deploying software.

* Jenkins can be installed through native system packages, Docker, or even run standalone by any machine with a Java Runtime Environment (JRE) installed.

### Download and Install Jenkins
Since Jenkins is written in Java, it needs the JDK to exists in the machine to be able to run, so we first need to download the JDK and then Jenkins.

If you are on a Debian distribution you can use the following bash script:
```
#! /bin/bash
sudo apt update
sudo apt install -y openjdk-11-jdk
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt -y install jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
```
### Python installation
Need to install Python to execute upcoming Python-based script

```
# Python installation steps
yes "" | sudo add-apt-repository universe
sudo apt install python2-minimal -y
sudo update-alternatives --install /usr/bin/python python /usr/bin/python2 1
sudo update-alternatives --config python
sudo apt install curl -y
curl https://bootstrap.pypa.io/pip/2.7/get-pip.py --output get-pip.py
sudo python2 get-pip.py
```
### Create Admin User
Now that we have downloaded and installed Jenkins, we can proceed to setup the bash script to create a new admin user, download recommended plugins and then confirm the Jenkins URL.

In order to create an admin user, we can use the following script:


```
#! /bin/bash
url=http://localhost:8080
password=$(sudo cat /var/lib/jenkins/secrets/initialAdminPassword)
username=$(python -c "import urllib;print urllib.quote(raw_input(), safe='')" <<< "user")
new_password=$(python -c "import urllib;print urllib.quote(raw_input(), safe='')" <<< "password")
fullname=$(python -c "import urllib;print urllib.quote(raw_input(), safe='')" <<< "full name")
email=$(python -c "import urllib;print urllib.quote(raw_input(), safe='')" <<< "hello@world.com")
cookie_jar="$(mktemp)"
full_crumb=$(curl -u "admin:$password" --cookie-jar "$cookie_jar" $url/crumbIssuer/api/xml?xpath=concat\(//crumbRequestField,%22:%22,//crumb\))
arr_crumb=(${full_crumb//:/ })
only_crumb=$(echo ${arr_crumb[1]})
curl -X POST -u "admin:$password" $url/setupWizard/createAdminUser \
        -H "Connection: keep-alive" \
        -H "Accept: application/json, text/javascript" \
        -H "X-Requested-With: XMLHttpRequest" \
        -H "$full_crumb" \
        -H "Content-Type: application/x-www-form-urlencoded" \
        --cookie $cookie_jar \
        --data-raw "username=$username&password1=$new_password&password2=$new_password&fullname=$fullname&email=$email&Jenkins-Crumb=$only_crumb&json=%7B%22username%22%3A%20%22$username%22%2C%20%22password1%22%3A%20%22$new_password%22%2C%20%22%24redact%22%3A%20%5B%22password1%22%2C%20%22password2%22%5D%2C%20%22password2%22%3A%20%22$new_password%22%2C%20%22fullname%22%3A%20%22$fullname%22%2C%20%22email%22%3A%20%22$email%22%2C%20%22Jenkins-Crumb%22%3A%20%22$only_crumb%22%7D&core%3Aapply=&Submit=Save&json=%7B%22username%22%3A%20%22$username%22%2C%20%22password1%22%3A%20%22$new_password%22%2C%20%22%24redact%22%3A%20%5B%22password1%22%2C%20%22password2%22%5D%2C%20%22password2%22%3A%20%22$new_password%22%2C%20%22fullname%22%3A%20%22$fullname%22%2C%20%22email%22%3A%20%22$email%22%2C%20%22Jenkins-Crumb%22%3A%20%22$only_crumb%22%7D"
```
At this point if we were to open the browser at localhost:8080 , we should be greeted with the Jenkins Login Page:
![Screenshot from 2023-08-14 13-40-14](https://github.com/vignesh-jumisa/readme/assets/141608315/09a20f51-d9e0-4c49-a7d8-dfa4ade26680)

And not with the ‘Unlock Jenkins’ page:

![Screenshot from 2023-08-14 13-42-50](https://github.com/vignesh-jumisa/readme/assets/141608315/af5a5f5a-d951-48b0-9f96-506c821a40f0)

### Install Recommended Plugins
In order to install recommended plugins, we can use the following script:

```
#! /bin/bash
url=http://localhost:8080

user=user
password=password

cookie_jar="$(mktemp)"
full_crumb=$(curl -u "$user:$password" --cookie-jar "$cookie_jar" $url/crumbIssuer/api/xml?xpath=concat\(//crumbRequestField,%22:%22,//crumb\))
arr_crumb=(${full_crumb//:/ })
only_crumb=$(echo ${arr_crumb[1]})

# MAKE THE REQUEST TO DOWNLOAD AND INSTALL REQUIRED MODULES
curl -X POST -u "$user:$password" $url/pluginManager/installPlugins \
  -H 'Connection: keep-alive' \
  -H 'Accept: application/json, text/javascript, */*; q=0.01' \
  -H 'X-Requested-With: XMLHttpRequest' \
  -H "$full_crumb" \
  -H 'Content-Type: application/json' \
  -H 'Accept-Language: en,en-US;q=0.9,it;q=0.8' \
  --cookie $cookie_jar \
  --data-raw "{'dynamicLoad':true,'plugins':['cloudbees-folder','antisamy-markup-formatter','build-timeout','credentials-binding','timestamper','ws-cleanup','ant','gradle','workflow-aggregator','github-branch-source','pipeline-github-lib','pipeline-stage-view','git','ssh-slaves','matrix-auth','pam-auth','ldap','email-ext','mailer'],'Jenkins-Crumb':'$only_crumb'}"
  ```

  ### Confirm Jenkins URL

If we were to log in with the credentials we have provided Jenkins, we would be prompt with the page asking us to install the plugins. But we can do this step via the following bash script.

  ```
#! /bin/bash

# The jenkins URL, if you are on an amazon instance, you can put there the
# public DNS name, in order to be able to access jenkins thorugh that URL
url=<your_jenkins_url>

user=user
password=password
url_urlEncoded=$(python -c "import urllib;print urllib.quote(raw_input(), safe='')" <<< "$url")

cookie_jar="$(mktemp)"
full_crumb=$(curl -u "$user:$password" --cookie-jar "$cookie_jar" $url/crumbIssuer/api/xml?xpath=concat\(//crumbRequestField,%22:%22,//crumb\))
arr_crumb=(${full_crumb//:/ })
only_crumb=$(echo ${arr_crumb[1]})

curl -X POST -u "$user:$password" $url/setupWizard/configureInstance \
  -H 'Connection: keep-alive' \
  -H 'Accept: application/json, text/javascript, */*; q=0.01' \
  -H 'X-Requested-With: XMLHttpRequest' \
  -H "$full_crumb" \
  -H 'Content-Type: application/x-www-form-urlencoded' \
  -H 'Accept-Language: en,en-US;q=0.9,it;q=0.8' \
  --cookie $cookie_jar \
  --data-raw "rootUrl=$url_urlEncoded%2F&Jenkins-Crumb=$only_crumb&json=%7B%22rootUrl%22%3A%20%22$url_urlEncoded%2F%22%2C%20%22Jenkins-Crumb%22%3A%20%22$only_crumb%22%7D&core%3Aapply=&Submit=Save&json=%7B%22rootUrl%22%3A%20%22$url_urlEncoded%2F%22%2C%20%22Jenkins-Crumb%22%3A%20%22$only_crumb%22%7D"
```
In the URL variable, you should put the URL through which you will access Jenkins. If you are on an amazon instance, you can put the public DNS name (which should be something like ip-10-0-0-60.ap-south-1.compute.internal . If you have installed jenkins on your machine, you can put http://localhost:8080 .

Once all these steps have been completed, you should be able to login into Jenkins and be prompted with the home page:
![Screenshot from 2023-08-14 13-44-21](https://github.com/vignesh-jumisa/readme/assets/141608315/5ec84140-7ff5-4842-aeee-75c307a81718)

After Jenkins has been installed and started, navigate to http://localhost:8080 (or the public dns if you are on the cloud) to access Jenkins in your browser.
