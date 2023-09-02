## CI-CD-on-DevOps-Website-Solution
### INSTALL AND CONFIGURE JENKINS SERVER
I spunned up an Ubuntu web server on AWS cloud and SSH into it.

I installed JDK which is an important Java based package required for Jenkins to run.

sudo apt update

sudo apt install default-jdk-headless

![javainstallation](https://github.com/Oolabanji/test_/assets/136812420/1c0c13e8-8d17-4593-bf49-5a1407beb86a)


wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -

sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \

    /etc/apt/sources.list.d/jenkins.list'

sudo apt update

sudo apt-get install jenkins


sudo systemctl enable jenkins

sudo systemctl start jenkins

sudo systemctl status jenkins


![jenkinsserviceup](https://github.com/Oolabanji/test_/assets/136812420/475381ea-2508-44aa-9b5f-7db85af744cf)


Since Jenkins runs on default port 8080, I open this port on the Security Group inbound rule of the jenkins server on AWS 


![port8080](https://github.com/Oolabanji/test_/assets/136812420/5fe4333a-450e-4be9-a328-876301f5f6e5)


Jenkins is up and running, I copied and paste jenkins server public ip address appended with port 8080 on a web server to gain access to the interactive console. http://18.214.12.248:8080

![jenkins1](https://github.com/Oolabanji/test_/assets/136812420/13673002-1a5c-4b70-b836-5b0de1af524f)


The admin password was found in the '/var/lib/jenkins/secrets/initialAdminPassword' path on the server.


![jenkins2](https://github.com/Oolabanji/test_/assets/136812420/8a6a60b8-066e-4614-821e-2a32904d003a)


![jenkins3](https://github.com/Oolabanji/test_/assets/136812420/bcbfb5b8-0be6-4675-a6f0-0a1abaae2dee)

![jenkins4](https://github.com/Oolabanji/test_/assets/136812420/9029d275-3c8d-4696-ba64-88abe5f7a877)



## Attaching WebHook to Jenkins Server

On the github repository that contains application code, I created a webhook to connect to the jenkins job. In order to create webhook, go to the settings tab on the github repo and click on webhooks. Webhook should look like this 18.214.12.248:8080/github-webhook/

![webhook](https://github.com/Oolabanji/test_/assets/136812420/46602afd-9fda-492b-8029-1357b7fd7e8c)


## Creating Job and Configuring GIT Based Push Trigger

On the jenkins server, I created a new freestyle job

![freestyle1](https://github.com/Oolabanji/test_/assets/136812420/a511f7ee-485e-44da-a2b9-8a12e8b2334c)


In configuration of the Jenkins freestyle job, I choose Git repository, provide there the link to the GitHub repository and credentials (user/password) so Jenkins could access files in the repository. Also I specified the branch containing code

![jenkinssetup](https://github.com/Oolabanji/test_/assets/136812420/3d48b0f6-ad34-41ad-91ca-148a62f99dca)


![jenkins5](https://github.com/Oolabanji/test_/assets/136812420/7f9a5a98-fa17-45c2-b3a5-414814c08110)


![jenkinssetup](https://github.com/Oolabanji/test_/assets/136812420/8b7999ff-44ef-4562-80f7-39b411f29234)

## Configuring Build Triggers

I specified the particular trigger to use for triggering the job. Then Click "Configure" on the jenkins job and add these two configurations below 


##  Configure triggering the job from GitHub webhook:


![jenkins6](https://github.com/Oolabanji/test_/assets/136812420/e1e7ab45-5548-48bd-a3c4-782e4dc4438d)

## Configure "Post-build Actions" to archive all the files – files resulted from a build are called "artifacts".

![jenkins7](https://github.com/Oolabanji/test_/assets/136812420/3000128f-e17b-4c6a-aeea-26eb49294f02)

At this point, the  architecture has pretty much been built, letI tested it by making a change on any file(README  file) on the Github repository and then push it to see the triggered job 

![success](https://github.com/Oolabanji/test_/assets/136812420/54a90931-0430-4308-98d4-208223c964af)




![jenkins8](https://github.com/Oolabanji/test_/assets/136812420/45f610d6-e78f-4f39-a2c7-a50250fbaf4b)


The console output shows the created job and the successful build. In this case the code on Github was built into an artifact on our Jenkins server workspace. The artificat is by checking the status tab of the completed job .


![jenkinschange](https://github.com/Oolabanji/test_/assets/136812420/ce84bbe0-77f1-4a62-8b74-1f45c3ba2894)




![jenkinschange1](https://github.com/Oolabanji/test_/assets/136812420/92d14941-ac87-4275-b445-f5c3bec73f89)

The created artifact can be found on the local terminal too at this path /var/lib/jenkins/jobs/tooling_github/builds/5/archive/

![artifactspath](https://github.com/Oolabanji/test_/assets/136812420/23a6d4fd-2c35-4f50-96ba-d52783063f17)

## Configuring Jenkins To Copy Files(Artifact) to NFS Server

To achieve this, i installed the Publish Via SSH pluging on Jenkins. The plugin allows one to send newly created packages to a remote server and install them, start and stop services that the build may depend on and many other use cases.

I configured the job to copy artifacts over to NFS server.

I test the configuration and made sure the connection returns Success. 


![publishoverssh](https://github.com/Oolabanji/test_/assets/136812420/a564bd57-1879-4481-9182-ffe2ed454966)


![sshserver](https://github.com/Oolabanji/test_/assets/136812420/5da914c7-e849-43cd-a8ae-e4799cf089b5)

![success1](https://github.com/Oolabanji/test_/assets/136812420/445c4945-2692-4692-933f-d02d8420ac30)



I specified ** on the send build artifacts tab meaning it sends all artifact to specified destination path(NFS Server).




![artifactsoverSSH](https://github.com/Oolabanji/test_/assets/136812420/2c693c66-1b7a-44f2-8eae-8806a88e4b01)

Now I made a new change on the README file on the source code(GitHub Tooling repository) and push to github, Jenkins builds an artifact by downloading the code into its workspace based on the latest commit and via SSH it publishes the artifact into the NFS Server to update the source code.

Webhook triggered a new job and in the “Console Output” of the job like below

SSH: Transferred 25 file(s)
Finished: SUCCESS


![success3](https://github.com/Oolabanji/test_/assets/136812420/70f16971-e49e-49ea-b103-32a5f270f4bd)


To confirm that the files in /mnt/apps have been updated – I connected via SSH/Putty to the NFS server and check README.MD file

cat /mnt/apps/README.md
And I saw the changes you had previously made in your GitHub – the job works as expected.


![success4](https://github.com/Oolabanji/test_/assets/136812420/d22254bf-82d0-4451-b186-c4f70baa7b7b)









