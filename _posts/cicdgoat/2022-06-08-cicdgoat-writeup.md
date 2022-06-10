---
published: false
---
Recently I came across this tweet and figured it would be a good starting point to learn some DevOps misconfigurations.  

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">🐐 CI/CD Goat<br><br>A deliberately vulnerable CI/CD environment<br><br>Learn and practice CI/CD security through a set of 10 challenges, enacted against a real, full blown CI/CD environment <br><br>By <a href="https://twitter.com/cider_sec?ref_src=twsrc%5Etfw">@cider_sec</a> <a href="https://twitter.com/hashtag/appsec?src=hash&amp;ref_src=twsrc%5Etfw">#appsec</a> <a href="https://twitter.com/hashtag/pentesting?src=hash&amp;ref_src=twsrc%5Etfw">#pentesting</a><a href="https://t.co/7LNu6htIen">https://t.co/7LNu6htIen</a></p>&mdash; Clint Gibler (@clintgibler) <a href="https://twitter.com/clintgibler/status/1514981872899657730?ref_src=twsrc%5Etfw">April 15, 2022</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## **Setting up the CI/CD Goat**

The entire pipeline is setup over docker .The install instructions are fairly simple to follow from the GitHub; just download the docker-compose yaml file and build it ; need to have docker installed for obvious reasons. If the install was successful then we should have images up and running like so

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2F0EO5VGwL4BiwMAq68Wbm%2Fimage.png?alt=media)

We go with the assume breach scenario as mentioned in the challenges : 
* We have access to gitea and the ability to create branch  and push code to the repos . 
* We have access to the Jenkins application .  

## Challenges

Log into the CTF portal at [http://localhost:8000/challenges ](http://localhost:8000/challenges ) 

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2Fiy4mqC0tjzWIMmNGE98Q%2Fimage.png?alt=media&token=eef6eb26-80fa-45d9-8a67-b17e7967c6a5)

## Easy challenges

**White rabbit** :  
![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FnonFCIY3YjoLJrr077lE%2Fimage.png?alt=media&token=ba14c7b0-033e-47e7-9684-68234c41a448)

As the challenge suggests the we need to get  flag1 located at the Jenkins Credential store by using the repository named Wonderland/white-rabbit hosted on gitea. To solve this challenge we need to perform a Direct PPE Attack. Essentially this attack can be performed when an attacker has access to a remote repository and can modify the  CI file either on the remote repo directly or by creating a new branch and submitting a Pull Request[PR] which the pipeline processes automatically . More details on this attack from the folks who created this challenge [here](https://www.cidersecurity.io/blog/research/ppe-poisoned-pipeline-execution/).


Login to gitea and view the repo
![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2F6XAU3whyhnd6PrCHS7ge%2Fimage.png?alt=media&token=34a5cc81-2ca5-4e82-8c00-6d7ca5841d9b)


Notice we are in  the one and only main branch .Now I'll clone this repo and work on the exploit on a new branch named whiterabbit-flag1 . 

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2F0s3cK5zh7hjzV97k1ssq%2Fimage.png?alt=media&token=99c2922b-9dd3-435d-907b-c700a145a1b2)

Let's modify the Jenkisfile   a to pull flag1 from the credential store. Essentially we use the withCredentials()  binding plugin which allows credentials to to bound to variable name; Now we create a new variable named flag1 and bind the credentialId to that . 
The Jenkisfile should be modified like so :

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FfssoUgS4d0hcL7CzmzwP%2Fimage.png?alt=media&token=baf71373-a7bf-4757-b0b8-64bcd87ec5c4)

Now we simply commit all these files and push them to the remote repo.

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FN44bWyOyamtqxqWEl1tS%2Fimage.png?alt=media&token=a2bae389-863a-485a-a35d-c80a8bce9d82)

Great !! our new branch is created and with the modified Jenkisfile
![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FTWPhKYqhpIHO7vc1R92A%2Fimage.png?alt=media&token=2da2a707-693b-44a4-8edf-b54ba8eb9ad3)

Now all we have to do is create a Pull Request [PR]  to merge our new branch into the main branch and Jenkis will process this request .

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FBKJ6VXBH5XNF0ljlANab%2Fimage.png?alt=media&token=09027a9a-488a-4760-be4d-f0a660756ad4)

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FXByYy60oFvqCIRhVG8hN%2Fimage.png?alt=media&token=bfd34e33-b20d-46f4-9f16-dbfb38d85f76)

Now login to Jenkis and after a while  we can see our PR being processed.
![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FEx0ppJzop5UWVXkRG8kg%2Fimage.png?alt=media&token=363cd973-2547-492b-aec5-6833328b4bd1)

Navigate to that specific pipeline and view the console history .We have the base64 encoded flag of the first challenge !
![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FyJnSZxcftwY5fctdDwDh%2Fimage.png?alt=media&token=82412d9e-560f-460d-b92e-1427ff19b9d0)

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2Ftlde6aHuuGgUPRsprqIk%2Fimage.png?alt=media&token=d3410515-6a9b-46f8-b286-982f03626365)

Base64 decode the blob and we have our first flag
![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FKtVc6Cd6porqQ4UTOH0X%2Fimage.png?alt=media&token=e23358cc-a178-4bf9-ac2c-094562666a1a)

**Mad hatter** 

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FVTXBMk48MW1r810zRRlB%2Fimage.png?alt=media&token=2c1a5dea-59e9-4473-a952-da06dc24178a)

In this challenge  we are hinted that the Jenkinsfile is protected and cannot be modified, so a DirectPPE attack is not possible. Let's confirm this ! 

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2F5qyubPCYxAb9k4CO1wtp%2Fimage.png?alt=media&token=91005e96-2f88-49cd-8987-96702fa4a286)

We see that there are two repos for mad-hatter . One contains the Jenkinsfile and the other one has the  the code files. Let's try to clone the repo which has the Jenkinsfile and make changes to it  like we did in the previous challenge and see if it works!
The modified Jenkinsfile is as shown below . ; We push these files into the remote repo and we get access denied !

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FCeE27fLf1NDIv7X7t7OR%2Fimage.png?alt=media&token=4a1bccb2-c52a-484a-af75-bbb0e951de7c)

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FzOXiBGxXn02fffv7JicY%2Fimage.png?alt=media&token=18867576-3220-45f1-8a5b-8235934c2a00)

As suspected Direct PPE is not possible, but not all is lost ! Usually the CI files rely on other files to perform build operations . If we could hijack any of these "other" files then we can indirectly hack the pipeline without making changes to the CI file!! This ,in a nutshell, is what we call an Indirect PPE attack. Taking a look at the Jenkinsfile we can see that it relies on a make file which has the flag3 as a credential variable named passwordVariable. We can  edit this  make file as it is not write protected and in a different repository.

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FqyEXUCjTW9OcnkMzNpgO%2Fimage.png?alt=media&token=5ca4528e-4199-4f9f-88e5-aa19dd67a29b)

So we clone the other repo and modify the make file  and push  changes as shown in images below
![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FJ6GzL31WzCMjXJ2qhxpG%2Fimage.png?alt=media&token=24db303c-f64c-4754-846c-f67f5cfba5a4)

Essentially we modify the make file to make an http request to our controlled server with the flag as an url parameter.

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FkYjlJn9ouh4TxCob8czP%2Fimage.png?alt=media&token=0f9337cd-3a06-4be1-a13a-b59ef755263e)

We see that our push was successful and the newly modified make file
![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FVgPupmCCYhWEi1d0LdTv%2Fimage.png?alt=media&token=2f77b5fa-5c9e-4c23-bd3f-c47cb7651e47)

I won't be showing pull requests unless required from here on as they are similar to the previous challenge. So ,Jenkins uses our newly modified make file and sends a request to our server with the flag3 as shown in the image below. 

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2Fd2jNTY57PvKOCMZXhufm%2Fimage.png?alt=media&token=f1251b55-ad1a-421f-8b52-d5c05033e065)

**Duchess** : 

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FYnTfrSQ29creLIhq0sFQ%2Fimage.png?alt=media&token=effe8df5-4aab-49b8-a727-70304bd397e3)

The third easy and final challenge hints at the Wonderland/duchess repository and apparently we might find PyPi token if we dig around ! 

We'll clone the repository and have  a look at that . Essentially we have to have a look at the all the previous commits in this repo and check if any secrets are left bind which the developers forgot to purge. Let's do that ! 

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2F2k1R47q95GgZFBnbP3e2%2Fimage.png?alt=media&token=a3285072-f209-4bdb-adff-8e291e8ed555)

There are 696 commits in this repo!! Looking at all of these will be a painful task  . There are automated tools which can find these sort of secrets, commits based or regular exps, special strings,etc . Gitleaks is one such tool. 

```apt install gitleaks```

We simply run this tool against the repo and it will do it's magic by looking into all kinds of previous commits and history and files and display any juicy info 

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2Fn9eznVDV1JUGXYRmlLrS%2Fimage.png?alt=media&token=c330c116-2782-4e36-9d0b-967a5f0af533)

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2F2TKiD6oYa0Vz7UNTO5vt%2Fimage.png?alt=media&token=5e28d07e-8bf1-43ea-a937-44bf75c8f9c7)

We are done with the easy challenges !! 


## Medium challenges  :
**Cheshire Cat**: 
![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FDD5kbE0F7NGChppmMCwd%2Fimage.png?alt=media&token=eb49c908-6e07-4b10-a127-f995c2b6aa3d)

This challenge we need to pull of a direct PPE attack on the Jenkins controller on the built-in node . 
What does the above statement mean ?? This image taken from [Jenkins Documentation](https://www.jenkins.io/doc/book/managing/nodes/) gives some idea 

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FRKAbjejs4HX0UHaz5AvB%2Fimage.png?alt=media&token=d194fdb7-1c8c-4ba7-8cdc-9b291b816df1)

Essentially there are dedicated nodes where every build agents run. All build operations are done on a node independently without any relation to other nodes it is a concept called Controller Isolation . Jenkins Controller which is the  main component of the entire Jenkins application  is also on a  node which is the "built-in" node! It is not advised to run build ops on this node due to performance degradation. But this node has all the secrets and juicy information about the severs, etc. If we could run code on this "built-in" node then we have access to all the secrets!  Let's  see that in action ! 
I'll clone the repo  checkout to a new branch and modify the Jenkinsfile as shown in the images below 

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FxcKM4I68vAjDKIDQus4d%2Fimage.png?alt=media&token=2b0d0959-0c88-45cd-b0e1-ce5f635f8c3e)

We instruct Jenkins agent to use the "built in" node instead of "any" node and then display the flag5 

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2Fu7jqA6Nr8UZmrDiLcGHY%2Fimage.png?alt=media&token=581e3668-65b2-4ff6-b800-628c0629efa4)

I'll push the these changes in the remote repo and submit a pull request.PR not shown in the images  as it is similar to first challenge. 

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2Fqgvt1P8MAyVZLTiUgGgz%2Fimage.png?alt=media&token=61e0428a-6c0c-4342-a1eb-24ffd9cb7f55)


![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2Flvkn5qfyTxRKY1t3MyMH%2Fimage.png?alt=media&token=aa5403bb-6fc9-44f7-8507-b56b74e5115d)

Login to jenkins and see the PR being processed click on the specific PR and check the console history to get flag5

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FJumVHZURmyucxjAX42hX%2Fimage.png?alt=media&token=fdd88c15-276d-441a-9761-589056eade07)



![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FbYvRnbPtYzltOWQyv4fx%2Fimage.png?alt=media&token=4efd6fc8-9fed-447e-ac99-b47dfb07c864)



**Twiddledum** :   

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FUnSbi0fYviDUhYjCrfxS%2Fimage.png?alt=media&token=d3ad75a0-4fe3-43e4-8e51-c6bd83d5c507)

The challenge does not hint at any kind of attack vector , we have to figure this out manually by looking into the twiddledum repository . 

Let's clone it and see if we can add or modify files . As suspected we do not have write permissions on this repo .. but as we see in other challenges and it a common recurring theme that we can hijack other files which this repo uses . 

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FilD16quRDeJz5imlwsip%2Fimage.png?alt=media&token=f6918225-17d0-4b4b-aa66-751c8b85a1b8)


Looking at the package.json file we notice that it uses a package from the local git server ! 

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2Fqz3pfsxfpRg2rDmRUXS2%2Fimage.png?alt=media&token=157e43c3-c32a-439e-bc79-306273376b56)

If we have write access over this "twiddledee"  package repo then we can hijack this pipeline! Let's try that

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FUHNUgueSxJr1mg4xqZ6h%2Fimage.png?alt=media&token=797a1194-fabf-4d74-8b40-c76396cf0647)

We use the childprocess module within javascript to run code and make a curl request to our controlled server with the flag6 as the url paramater ! 

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FggiUi22piv5maZy2frGt%2Fimage.png?alt=media&token=88801eb1-edad-49e9-ae84-eb74917991db)

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FTloFLcaRaI98qffiNp63%2Fimage.png?alt=media&token=c9b10272-35e2-413e-a712-2d6d22438d22)

Initially I tried without the tags and could not get the flag .. As the the pipeline was pulling stuff from the tags and ignoring everything else. I will show why  later , make a note to add tags to the commit as shown above!

Okay!! Now we trigger the wonderland-twiddledum  pipeline, has to be done manually!. When the pipeline is built it will fetch  JavaScript the twiddledee repository and as a result execute our malicous js code in the index.js file and make a request to our server with the flag6!!
Login to jenkins and click on the wonderland-twiddledum  pipeline and click "BuildNow". After a few seconds we see Jenkins making a request to our server with the flag . 

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FCBbF4PjoVtSHJcC3sS4z%2Fimage.png?alt=media&token=5baa5adc-f3b8-4f29-8bfc-de4b4d7cd5c9)



![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2Fbk32BCCRU3225NYv6xst%2Fimage.png?alt=media&token=2cc5f7fb-6076-42c6-a527-621f145c7211)

We have solved this challenge successfully! 

The reason we needed tags was because Jenkins pull the repo files from the tags number  and nothing else as is evident in the console history 


![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2Fe6aidlefEtLS7ww1xfuD%2Fimage.png?alt=media&token=4a5a1ae3-9fdd-45c2-ad82-10af0a81e80f)







**Dodo** : 
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

I'll push the these changes in the remote repo and submit a pull request.PR not shown in the images  as it is similar to first challenge.  

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2Fqgvt1P8MAyVZLTiUgGgz%2Fimage.png?alt=media&token=61e0428a-6c0c-4342-a1eb-24ffd9cb7f55)

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2Flvkn5qfyTxRKY1t3MyMH%2Fimage.png?alt=media&token=aa5403bb-6fc9-44f7-8507-b56b74e5115d)

Login to jenkins and see the PR being processed click on the specific PR and check the console history

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FJumVHZURmyucxjAX42hX%2Fimage.png?alt=media&token=fdd88c15-276d-441a-9761-589056eade07)

![](https://files.gitbook.com/v0/b/gitbook-x-prod.appspot.com/o/spaces%2F0ozxUoEgqX8lSXt4wQpe%2Fuploads%2FbYvRnbPtYzltOWQyv4fx%2Fimage.png?alt=media&token=4efd6fc8-9fed-447e-ac99-b47dfb07c864)


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






