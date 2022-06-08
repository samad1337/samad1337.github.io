---
published: false
---
## Dodo : 
![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FqlPNd2hb3v90GissZrTO%2Fimage.png?alt=media)

In this challenge we have to make a S3 bucket public without getting detected, meaning the  application has some sort of security measures to prevent the cloud environment from being tampered with. On a side note : we are emulating an aws environment using localstack, which is a program designed to help beginners to understand aws .

Viewing the hints of this challenge we see that a program named **checkhov** is mentioned . Upon googling this program we see it detects and prevents tampering within cloud deployments. Let's confirm this by actually tampering with the **main.tf** file . This file has configuration of the bucket and it's associated ACL's.  
I'll clone the repo ,modify the  main.tf file and push the changes. 

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2Fx7q6k9uH3aqpJ6aP3y1W%2Fimage.png?alt=media)

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FRzWfE3mu1MZwgAjcc82b%2Fimage.png?alt=media&token=8b49686d-3f7a-42c2-b8fa-ce370ed289c3)

I'll trigger the pipeline manually and the build fails because checkov detected that the acl was tampered

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2Fbr9H7uf0HccLpJ75YSue%2Fimage.png?alt=media)

I need to bypass this security check happening during the build time. There are multiple ways to do this . This [blogpost](https://bridgecrew.io/blog/checkov-iac-policy-severities-prioritize-skip-fail/) from the folks who made this program goes in detail about skipping checks . 

Essentially we need to create  a **```.checkov.yaml```**  which tells  checkov to ignore all checks during the build
I'll make that file and push it to the repo 

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FRSpLLK8owhI6jFhRuZCd%2Fimage.png?alt=media&token=6c522435-8699-4503-acd0-d512acb797bd)

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FVLCHk6dPWzE01awRiwQJ%2Fimage.png?alt=media)

Trigger the pipeline again and this time checkov ignores all checks and the build succeds and we get the flag7!

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FtHzV3ZDpzxaSYqNoKanZ%2Fimage.png?alt=media&token=704825cc-0a29-4d50-a0c8-de13a480f273)



### Hard Challenges 

## Hearts
![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FnxpFDKfVCeBOwHOxHCnk%2Fimage.png?alt=media&token=c2ca7fb1-7577-4c7d-8bdc-36c7ebf536f5)

We are told that system credentials are located in Jenkins and we need to get those via an admin agent since they have the permissions! 

Our user Alice is not an agents admin as observed .

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FOEZ11sWeW4tTh4ELa86Y%2Fimage.png?alt=media)

But there exists another user named **knave** who is an agents admin.  

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FRM57Umv2WLaEGUdX2PiP%2Fimage.png?alt=media&token=835cfa81-6f6b-4a73-8e84-a7eae15db73c)

We could login to his account and configure an agent to give us the Jenkins credentials by forcing an SSH connection to our  attacker controlled SSH server via MITM . But we don't know his password! 
We could bruteforce the password but one of the hints in this challenge already discloses the password as _rockme_ .  

Two important things to notice :
* We configure an agent to perfrom SSH auth on our attacker controlled rouge SSH server 
* Port 10022 is being used becasue the rouge SSH server we are gonna setup utlises 10022 as its working port!

Downlaod this app image from [here](https://github.com/ssh-mitm/ssh-mitm/releases/download/0.6.3/ssh-mitm-x86_64.AppImage) and run the following command to setup the rouge server :
```ssh-mitm-x86_64.AppImage.1  --remote-host <attackerip>```

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FjpZMiGLSnjH8sQfKB07X%2Fimage.png?alt=media&token=b0703ae3-69e1-48df-8c98-2a586fd07431)


Now login to the jenkins and configure an agent to perform an SSH operation using the Jenkins credentials as shown in the images below . 

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FnLp4uLXxWlLxrJMZsi2q%2Fimage.png?alt=media&token=fc98112b-1348-4d00-b170-18843ba3e6bc)

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FvIpZYaJzzeYKvRlcKz5y%2Fimage.png?alt=media&token=8e7caac8-e73a-4262-b7f9-5960fb710ceb)

Note: We use the Non Verifying Verfication strategy to prevent any kind of key based or host file based auth mechanisms and it will be much easier to pull of an MITM attack if this kind of  Host Key Verification Strategy is being used . 

Now save this agent and let it run . Immediately you should see our rouge SSH sever doing an MiTM attack and sniffing the credentials ! here the password is  flag8 ! 

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2F6MwE9Xixk5iijNJL2wcd%2Fimage.png?alt=media&token=ba926940-c6e3-4a11-8211-2061dc148294)


