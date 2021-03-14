# Continuous Integration with Jenkins

Continuous integration (CI) is the practice of automating the
integration of code changes from multiple contributors into a single
software project. It's a primary DevOps best practice, a software
development strategy that increases the speed of development while
ensuring the quality of the code that teams deploy. Developers
continually commit code in small increments (at least daily, or even
several times a day), which is then automatically built and tested
before it is merged with the shared repository.

A source code version control system is the crux of the CI process. The
version control system is also supplemented with other checks like
automated code quality tests, syntax style review tools, and more.
Examples of source code version control tools are Git, Subversion (SVN),
Mercurial, Team Foundation Server (TFS), Monotone and Bazaar.

### The importance of continuous integration

Without CI, developers must manually coordinate and communicate when
they are contributing code to the end product. This coordination extends
beyond the development teams to operations and the rest of the
organization. Product teams must coordinate when to sequentially launch
features and fixes and which team members will be responsible.

The communication overhead of a non-CI environment can become a complex
and entangled synchronization chore, which adds unnecessary bureaucratic
cost to projects. This causes slower code releases with higher rates of
failure, as it requires developers to be sensitive and thoughtful
towards the integrations. These risks grow exponentially as the
engineering team and codebase sizes increase.

Without a robust CI pipeline, a disconnect between the engineering team
and the rest of the organization can form. Communication between product
and engineering can be cumbersome. Engineering becomes a black box which
the rest of the team inputs requirements and features and maybe gets
expected results back. It will make it harder for engineering to
estimate time of delivery on requests because the time to integrate new
changes becomes an unknown risk.

### What CI does

CI helps to scale up headcount and delivery output of engineering teams.
Introducing CI to the aforementioned scenario allows software developers
to work independently on features in parallel. When they are ready to
merge these features into the end product, they can do so independently
and rapidly. CI is a valuable and well-established practice in modern,
high performance software engineering organizations.

Examples of CI tools are Jenkins, CircleCI, AWS CodeBuild, Azure DevOps,
Atlassian Bamboo, and Travis CI.

In this project, we will implement CI for the Tooling Website using
Jenkins.

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image1.png)

## Prerequisite
Completion of Network-attached-Storage-Demo project (Project 1)

## Step 1: Prepare the Jenkins server

Create a new Ubuntu 20.04 virtual machine and install Jenkins on it.

Install NFS client

*\$ sudo apt update*

*\$ sudo apt -y install nfs-common*

Mount /var/lib/ and target the NFS server's export for Jenkins i.e.
/mount/opt in this case.

*\$ sudo mount -t nfs -o rw,nosuid 10.0.0.4:/mount/opt /var/lib*

Verify that NFS was mounted successfully by running *df -h*.

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image2.png)

Make sure that the changes will persist on Jenkins-Server after reboot.

Update /etc/fstab file so that the mount configurations will persist
after restart of the server.

Add the below line at the end of the contents of the /etc/fstab with any
editor of choice:

10.0.0.4:/mount/opt /var/lib nfs defaults 0 0

*\$ sudo nano /etc/fstab*

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image3.png)

Jenkins is a Java application and requires Java 8 or later to be
installed on the server.

Run the following commands as root or user with sudo privileges or root
to install OpenJDK 11:

*\$ sudo apt install openjdk-11-jdk*

Once the installation is complete, verify it by checking the Java
version:

*\$ java -version*

Import the GPG keys of the Jenkins repository using the following wget
command:

*\$ wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key \| sudo
apt-key add --*

Add the Jenkins repository to the system with:

*\$ sudo sh -c \'echo deb http://pkg.jenkins.io/debian-stable binary/ \>
/etc/apt/sources.list.d/jenkins.list\'*

Install the latest version of Jenkins:

*\$ sudo apt update*

*\$ sudo apt install Jenkins*

Verify the status of the Jenkins service:

*\$ sudo systemctl status Jenkins*

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image4.png)

Enable Jenkins server to always start at boot:

*\$ sudo systemctl enable Jenkins*

Disable UFW

*\$ sudo UFW disable*

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image5.png)

Open port 8080 on the Jenkins server's Network Security Group:

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image6.png)

To set up the new Jenkins installation, open a browser, type the IP
address or DNS name of the server, followed by port 8080 i.e.
http://your_ip_or_domain:8080.

A page similar to the following will be displayed, prompting you to
enter the Administrator password that is created during the
installation:

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image7.png)

Use cat to display the password on the terminal:

\$ sudo cat /var/lib/jenkins/secrets/initialAdminPassword

Copy the 32-character password from the terminal, paste it into the
"Administrator password" field and click "Continue".

The next screen presents the option of installing suggested plugins or
selecting specific plugins:

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image8.png)

Click the Install suggested plugins option, which will immediately begin
the installation process.

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image9.png)

When the installation is complete, the next screen will prompt to set up
the first administrative user. Enter the name and password for the user.
It's possible to skip this step and continue as admin using the
32-character password.

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image10.png)

The next page will ask to set the URL for the Jenkins server. The field
will be already populated with the IP address of the server.

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image11.png)

Confirm the URL by clicking on the Save and Finish button, and the setup
process will be completed. A confirmation page confirming that "Jenkins
is Ready!" will show up.

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image12.png)

Click "Start using Jenkins" to visit the main Jenkins dashboard:

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image13.png)

The Jenkins server has been successfully installed at this point.

Note: If an error message is displayed in the \"Manage Jenkins\" page
stating 'It appears that your reverse proxy setup is broken', there is
an option in the \"Global Security Settings\". Look for CSRF Protection
setting, tick \"Enable proxy compatibility\" and Save this setting.

## Step 2: Prepare the Jenkins Slave server

The Jenkins server can run build tasks, but in a scenario where there
are multiple builds at the same time, this can affect the performance of
the server. A Jenkins Slave is a Java executable that runs on a remote
machine. The Jenkins master server acts to schedule the jobs and assign
slaves and send builds to slaves to execute the jobs. Slaves can run on
a variety of operating systems. A project can be configured to always
run on a particular Slave machine or a particular type of Slave machine,
or simply let Jenkins pick the next available Slave. This will reduce
build overhead to the Jenkins master and builds can be run in parallel.

Create a new Ubuntu 20.04 virtual machine and install the same Java
version that was installed on Jenkins master server.

*\$ sudo apt update*

*\$ sudo apt install openjdk-11-jdk*

Once the installation is complete, verify it by checking the Java
version:

*\$ java --version*

The Jenkins Server (Master) needs to be able to connect to the Slave via
*ssh*, with the Private key known by the Master, and the corresponding
public key is put in the agent's *\~/.ssh/authorized_keys* file.

Create private and public SSH keys. The following command creates the
private key *id_rsa* and the public key *id_rsa.pub*.

*\$ ssh-keygen --t rsa*

The setup will request for a file in which to save the key. Press enter
key to use the default file and location (/home/user_name/.ssh/id_rsa).
To increase security, a passphrase is advisable to associate it to your
Private Key.

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image14.png)

Add the public SSH key to the list of authorized keys on the Jenkins
Slave server

*\$ cat \~/.ssh/id_rsa.pub \>\> \~/.ssh/authorized_keys*

Ensure that the permissions of the \~/.ssh directory is secure, as most
SSH daemons will refuse to use keys that have file permissions that are
considered insecure:

*\$ sudo chmod 700 \~/.ssh*

*S sudo chmod 600 \~/.ssh/authorized_keys \~/.ssh/id_rsa*

Copy all the contents of the private SSH key (\~/.ssh/id_rsa) from the
Slave server to a notepad:

*\$ cat \~/.ssh/id_rsa*

Log in to the Jenkins console via the browser and click on \"Manage
Jenkins\" and scroll down to the bottom. From the list click on \"Manage
Nodes and Clouds\". In the new window click on \"New Node\".

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image15.png)

Give a name to the node, select \"Permanent Agent\" and click on OK

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image16.png)

In the remote root directory field, enter a path in the slave node. Note
that SSH user must have read/write access to this directory path. The
slave server user\'s home directory can be used .i.e.
/home/SLAVE_USERNAME.

For the Launch method, select 'Launch agents via SSH', then insert the
Slave VM's Public IP address (or Private IP if the Jenkins Server and
the Slave Server are in the same virtual network) in the Host field.

To add the SSH credentials, click the Add button and click on the
Jenkins drop-down button.

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image17.png)

Jenkins will pop-up a new window to add credentials. Select the kind as
\"SSH Username with private key\" from the drop-down. Enter the user
name of the slave node. In the private key field, click on the 'Enter
directly' button and add the private key copied from the Jenkins Slave
server.

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image18.jpeg)

Click on add and select the credentials we created from the drop-down.

In the Host key Verification Strategy, select Manually trusted key
Verification Strategy. Do not tick the 'Require manual verification of
initial connection'.

Save the configuration and the agent connection log should be similar
to:

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image19.png)

The Slave node is now ready for use.

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image20.png)

## Step 3 - Configure Jenkins to retrieve source codes from GitHub using
Webhooks

Enable Webhooks in the GitHub repository settings of the tooling
website.

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image21.gif)

Go to Jenkins web console, click "New Item" and create a "Freestyle
project".

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image22.png)

To connect the GitHub repository, its URL is required, which can be
copied from the repository.

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image23.png)

In configuration of the Jenkins freestyle project choose Git repository
in 'Source Code Management', provide there the link to the Tooling
GitHub repository and credentials (user/password) so Jenkins could
access files in the repository.

In 'Build Triggers' section, tick GitHub hook trigger for GITScm
polling. This will enable that a new build is launched automatically (by
webhook) after any change is pushed to the master branch. Note that the
project job has to run (build) at least once manually before the hook
will work.

Save the configuration and run the build manually. Click "Build Now"
button, if the configuration was done correctly, the build will be
successful and will be seen under \#1.

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image24.png)

Click on \#1 for more information on the build. The page will also show
the node that ran the build task: Slave-Node.

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image25.png)

This build does not produce anything. The following steps will lead to
the creation of Artifacts. Artifacts are files which are associated with
a single build. Artifact is the software component that is produced
during the build, stored, and ultimately deployed into different
environments. A build can have any number of artifacts associated with
it.

Click "Configure" the job/project and select 'Archive the artifacts' in
the 'Post-build Actions'. In the 'Files to archive' box, insert two
stars (i.e. \*\*). This copies the whole repository.

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image26.gif)

Make some changes in any file in the GitHub repository (e.g. README.md
file) and push the changes to the master branch.

A new build will be launched automatically (by Webhook) which would
result in artifacts, saved on the Jenkins server.

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image27.png)

By default, the artifacts are stored on Jenkins server:

*\$ ls
/var/lib/jenkins/jobs/tooling_github/builds/\<build_number\>/archive/*

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image28.png)

The artifacts will also be copied to the NFS server as a result of the
mount:

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image29.png)

## Step 4: Configure Jenkins to copy files to NFS server via SSH

Jenkins is a highly extendable application and there are 1400+ plugins
available. A plugin named "Publish Over SSH" is needed to copy the
artifacts saved locally on Jenkins server.

On the main dashboard select "Manage Jenkins" and choose "Manage
Plugins" menu item.

On "Available" tab search for "Publish Over SSH" plugin and install it.

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image30.png)

Configure the job/project to copy artifacts over to NFS server.

On the main dashboard select "Manage Jenkins" and choose "Configure
System" menu item.

Scroll down to Publish over SSH plugin configuration section and
configure it to be able to connect to the NFS server:

Key - Provide the contents of the private key (.pem format) file used to
connect to the NFS server via SSH. To convert Putty format of private
key (.ppk) to OpenSSH format, follow the instructions in this
[documentation](https://www.simplified.guide/putty/convert-ppk-to-ssh-key).
Putty format of private key will not work.

Name - Any name of choice

Hostname -- Public IP address of NFS server (or Private IP address of
NFS server if the Jenkins Server and the NFS server are in the same
virtual network)

Username -- Username of the NFS Server

Remote directory - /mount/apps since the Web Servers use it as a
mounting point to retrieve files from the NFS server

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image31.png)

Save the configuration, open the Jenkins job/project configuration page
and add another one "Post-build Action"

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image32.png)

Configure it to send all artifact files produced by the build into the
previously-defined remote directory.

In the 'Name' section, select the NFS server.

In the 'Source files' section, insert \*\* to copy all the directories
and files.

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image33.png)

Save this configuration and change something in the README.MD file in
the GitHub Tooling repository.

The webhook will trigger a new job and something similar to this will be
seen in the Console Output of the job:

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image34.png)

To make sure that the files in /mount/apps have been updated, connect
via SSH/Putty to the NFS server and check README.MD file.

*\$ cat /mount/apps/README.md*

If the changes previously made in the GitHub repository is seen, the
project was successfully deployed.

In a scenario where there is an error in the build:

ERROR: Exception when publishing, exception message \[Permission
denied\]

Build step \'Send files or execute commands over SSH\' changed build
result to UNSTABLE

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image35.png)

Give ownership permission to the NFS server's user account on the
/mount/apps directory:

*\$ sudo chown -R azureuser: /mount/apps*

Also, in the advanced tab of the 'Send build artifacts over SSH'
post-build action, select \"verbose output\". This will let the console
output of the running job be verbose. This helps in troubleshooting
build errors.

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image36.png)

Once a build is deleted, the artifacts for that build are deleted.

If the Slave nodes are not running, the Jenkins master node would run
the build tasks.

![](https://github.com/osygroup/Images/blob/main/CI-with-Jenkins/image37.png)

## Conclusion

CI is generally used alongside an agile software development workflow.
An organization will compile a list of tasks that comprise a product
roadmap. These tasks are then distributed amongst software engineering
team members for delivery. Using CI enables these software development
tasks to be developed independently and in parallel amongst the assigned
developers. Once one of these tasks is complete, a developer will
introduce that new work to the CI system to be integrated with the rest
of the project.

Continuous integration is an essential aspect of DevOps and
high-performing software teams. Yet CI benefits are not limited to the
engineering team but greatly benefit the overall organization. CI
enables better transparency and insight into the process of software
development and delivery. These benefits enable the rest of the
organization to better plan and execute go to market strategies.

## Credits

<https://www.atlassian.com/continuous-delivery/continuous-integration>

<https://linuxize.com/post/how-to-install-jenkins-on-ubuntu-20-04/>

<https://www.edureka.co/blog/jenkins-master-and-slave-architecture-a-complete-guide/#:~:text=Jenkins%20Slave,a%20variety%20of%20operating%20systems>.

<https://github.com/spinnaker/spinnaker/issues/2067>

<https://support.cloudbees.com/hc/en-us/articles/222978868-How-to-Connect-to-Remote-SSH-Agents->

<https://stackoverflow.com/questions/32823684/jenkins-multiple-artifacts-for-the-same-build>

<http://ant.apache.org/manual/dirtasks.html#patterns>

<https://www.simplified.guide/putty/convert-ppk-to-ssh-key>

<https://superuser.com/questions/966358/jenkins-publish-over-ssh-unstable-permission-denied>
