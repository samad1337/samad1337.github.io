---
published: true
---
# CICD-Goat Writeup

Recently I came across this tweet and figured it would be a good starting point to learn some DevOps misconfigurations.&#x20;



> 🐐 CI/CD Goat\
> \
> A deliberately vulnerable CI/CD environment\
> \
> Learn and practice CI/CD security through a set of 10 challenges, enacted against a real, full blown CI/CD environment\
> \
> By [@cider\_sec](https://twitter.com/cider\_sec?ref\_src=twsrc%5Etfw) [#appsec](https://twitter.com/hashtag/appsec?src=hash\&ref\_src=twsrc%5Etfw) [#pentesting](https://twitter.com/hashtag/pentesting?src=hash\&ref\_src=twsrc%5Etfw)[https://t.co/7LNu6htIen](https://t.co/7LNu6htIen)
>
> — Clint Gibler (@clintgibler) [April 15, 2022](https://twitter.com/clintgibler/status/1514981872899657730?ref\_src=twsrc%5Etfw)

### Setting up the CI/CD Goat

The entire pipeline is setup over docker .The install instructions are fairly simple to follow from the GitHub; just download the docker-compose yaml file and build it ; need to have <mark style="color:orange;">docker</mark> installed for obvious reasons. If the install was successful then we should have images up and running like so

![](<.gitbook/assets/image (30).png>)

### Challenges&#x20;

Log into the CTF portal at [http://localhost:8000/challenges](http://localhost:8000/challenges)  &#x20;

![](<.gitbook/assets/image (15).png>)

#### Easy Challenges :    &#x20;

<mark style="color:red;">**White rabbit**</mark>&#x20;

![](<.gitbook/assets/image (52).png>)

As the challenge suggests the we need to get  **flag1** located at the <mark style="color:orange;">Jenkins Credential store</mark> by using the repository named <mark style="color:orange;">`Wonderland/white-rabbit`</mark> <mark style="color:orange;"></mark><mark style="color:orange;"></mark> hosted on gitea. To solve this challenge we need to perform a <mark style="color:green;">**`Direct PPE Attack`**</mark>. Essentially this attack can be performed when an attacker has access to a remote repository and can modify the  CI file either on the remote repo directly or by creating a new branch and submitting a Pull Request\[PR] which the pipeline processes automatically . More details on this attack from the folks who created this challenge [here](https://www.cidersecurity.io/blog/research/ppe-poisoned-pipeline-execution/).

We go with the assume breach scenario as mentioned in the challenges :&#x20;

* We have access to gitea and the ability to create branch  and push code to the repos .&#x20;
* We have access to the Jenkins application . &#x20;

Theoretically we should be able to pull of this attack . Let's put to that to practice .

Login to gitea and view the repo&#x20;

![](<.gitbook/assets/image (47).png>)

Notice we are in  the one and only **main** branch .Now I'll clone this repo and work on the exploit on a new branch named <mark style="color:orange;">whiterabbit-flag1</mark> .&#x20;

![](<.gitbook/assets/image (41).png>)

&#x20;Let's modify the <mark style="color:orange;">**Jenkisfile**</mark>   a to pull flag1 from the credential store. Essentially we use the [<mark style="color:green;">**`withCredentials()`**</mark>](https://www.jenkins.io/doc/pipeline/steps/credentials-binding/)  binding plugin which allows credentials to to bound to variable name; Now we create a new variable named <mark style="color:orange;">flag1</mark> and bind the <mark style="color:orange;">credentialId</mark> to that .&#x20;

The Jenkisfile should be modified like so :&#x20;

![](<.gitbook/assets/image (59).png>)

Now we simply commit all these files and push them to the remote repo.

![](<.gitbook/assets/image (46).png>)

Great !! our new branch is created and with the modified **Jenkisfile**

![](<.gitbook/assets/image (5).png>)

Now all we have to do is create a **Pull Request** \[PR]  to merge our new branch into the **main** branch and Jenkis will process this request .

![](<.gitbook/assets/image (66).png>)

![](<.gitbook/assets/image (31).png>)

Now login to Jenkis and after a while  we can see our PR being processed.

![](<.gitbook/assets/image (48).png>)

Navigate to that specific pipeline and view the console history .We have the base64 encoded flag of the first challenge !

![](<.gitbook/assets/image (11).png>)

![](<.gitbook/assets/image (24).png>)

Base64 decode the blob and we have our first flag

![](<.gitbook/assets/image (64).png>)

<mark style="color:red;">**Mad hatter**</mark>&#x20;

![](<.gitbook/assets/image (58).png>)

In this challenge  we are hinted that the Jenkinsfile is protected and cannot be modified, so a DirectPPE attack is not possible. Let's confirm this !&#x20;

![](<.gitbook/assets/image (7).png>)

We see that there are two repos for mad-hatter . One contains the **Jenkinsfile** and the other one has the  the code files. Let's try to clone the repo which has the **Jenkinsfile** and make changes to it  like we did in the previous challenge and see if it works!

The modified Jenkinsfile is as shown below . ; We push these files into the remote repo and we get access denied !&#x20;

![](<.gitbook/assets/image (74).png>)

![](.gitbook/assets/image.png)

As suspected Direct PPE is not possible, but not all is lost ! Usually the CI files rely on other files to perform build operations . If we could hijack any of these "other" files then we can indirectly hack the pipeline without making changes to the CI file!! This ,in a nutshell, is what we call an <mark style="color:green;">``</mark><mark style="color:green;">` `</mark><mark style="color:green;">**`Indirect PPE attack.`**</mark> Taking a look at the **Jenkinsfile** we can see that it relies on a make file which has the flag3 as a credential variable named passwordVariable. We can  edit this  make file as it is not write protected and in a different repository. .

![](<.gitbook/assets/image (44).png>)

So we clone the other repo and modify the make file  and push  changes as shown in images below

![](<.gitbook/assets/image (38).png>)

Essentially we modify the make file to make an http request to our controlled server with the flag as an url parameter.

![](<.gitbook/assets/image (39).png>)

We see that our push was successful and the newly modified make file

![](<.gitbook/assets/image (55).png>)

I won't be showing pull requests unless required from here on as they are similar to the previous challenge. So ,Jenkins uses our newly modified make file and sends a request to our server with the flag3 as shown in the image below.&#x20;

![](<.gitbook/assets/image (22).png>)

<mark style="color:red;">**Duchess**</mark>

![](<.gitbook/assets/image (75).png>)

The third easy and final challenge hints at the Wonderland/duchess repository and apparently we might find PyPi token if we dig around !&#x20;

We'll clone the repository and have  a look at that . Essentially we have to have a look at the all the previous commits in this repo and check if any secrets are left bind which the developers forgot to purge. Let's do that !&#x20;

![](<.gitbook/assets/image (57).png>)

There are 696 commits in this repo!! Looking at all of these will be a painful task  . There are automated _tools_ which can find these sort of secrets, commits based or regular exps, special strings,etc . <mark style="color:orange;">**Gitleaks**</mark> is one such tool.&#x20;

`apt install gitleaks`

We simply run this tool against the repo and it will do it's magic by looking into all kinds of commits and files and display any juicy info&#x20;

![Snipped output ](<.gitbook/assets/image (67).png>)

![The pypi token is the flag for this challenge ](<.gitbook/assets/image (56).png>)

We are done with the easy challenges !!&#x20;

![](<.gitbook/assets/image (73).png>)

**Medium challenges**  :    &#x20;

<mark style="color:red;">Caterpillar</mark> :                 &#x20;

![](<.gitbook/assets/image (54).png>)

We have read permissions only on the _wonderland/caterpillar_ repository we can confirm that as shown below&#x20;

![Access denied when modifying the origin repo\[shown in the image\]. The same goes for creating a branch \[Not shown in the image\]](<.gitbook/assets/image (63).png>)

If we cannot branch , how about we fork the repo ?? Let us try that&#x20;

![](<.gitbook/assets/image (71).png>)

<mark style="color:red;">Cheshire cat</mark>&#x20;



![](<.gitbook/assets/image (19).png>)



This challenge we need to pull of a direct PPE attack on the Jenkins controller on the built-in node .&#x20;

What does the above statement mean ?? This image taken from [Jenkins Documentation](https://www.jenkins.io/doc/book/managing/nodes/) gives some idea&#x20;

&#x20;

![](<.gitbook/assets/image (25).png>)

Essentially there are dedicated nodes where every build agents run. All build operations are done on a node independently without any relation to other nodes it is a concept called [Controller Isolation](https://www.jenkins.io/doc/book/security/controller-isolation/) . Jenkins Controller which is the  main component of the entire Jenkins application  is also on a  node which is the "built-in" node! It is not advised to run build ops on this node due to performance degradation. But this node has all the secrets and juicy information about the severs, etc. If we could run code on this "built-in" node then we have access to all the secrets!  Let's  see that in action !&#x20;

I'll clone the repo  checkout to a new branch and modify the Jenkinsfile as shown in the images below&#x20;

![](<.gitbook/assets/image (61).png>)

We instruct Jenkins agent to use the "built in" node instead of "any" node and then display the flag5&#x20;

![](<.gitbook/assets/image (9).png>)





I'll push the these changes in the remote repo and submit a pull request.PR not shown in the images  as it is similar to first challenge.&#x20;

![](<.gitbook/assets/image (1).png>)

![](<.gitbook/assets/image (23).png>)

Login to jenkins and see the PR being processed click on the specific PR and check the console history

![](<.gitbook/assets/image (28).png>)

![](<.gitbook/assets/image (36).png>)

<mark style="color:red;">**Twiddledum :**</mark>  &#x20;

![](<.gitbook/assets/image (32).png>)

The challenge does not hint at any kind of attack vector , we have to figure this out manually by looking into the twiddledum repository .&#x20;

Let's clone it and see if we can add or modify files . As suspected we do not have write permissions on this repo .. but as we see in other challenges and it a common recurring theme that we can hijack other files which this repo uses .&#x20;

![](<.gitbook/assets/image (43).png>)

Looking at the package.json file we notice that it uses a package from the local git server !&#x20;

![](<.gitbook/assets/image (26).png>)

If we have write access over this "twiddledee"  package repo then we can hijack this pipeline! Let's try that

![](<.gitbook/assets/image (65).png>)

We use the childprocess module within javascript to run code and make a curl request to our controlled server with the flag6 as the url paramater !&#x20;

![](<.gitbook/assets/image (10).png>)

![](<.gitbook/assets/image (8).png>)

Initially I tried without the tags and could not get the flag .. As the the pipeline was pulling stuff from the tags and ignoring everything else. I will show why  later , make a note to add tags to the commit as shown above!

Okay!! Now we trigger the **wonderland-twiddledum**  pipeline, has to be done manually!. When the pipeline is built it will fetch  JavaScript the twiddledee repository and as a result execute our malicous js code in the index.js file and make a request to our server with the flag6!!

Login to jenkins and click on the **wonderland-twiddledum**  pipeline and click "**BuildNow**". After a few seconds we see Jenkins making a request to our server with the flag

![](<.gitbook/assets/image (12).png>)

![](<.gitbook/assets/image (27).png>)

We have solved this challenge successfully!&#x20;

The reason we needed tags was because Jenkins pull the repo files from the tags number  and nothing else as is evident in the console history&#x20;

![](<.gitbook/assets/image (14).png>)

<mark style="color:red;">**Dodo :**</mark>&#x20;

![](<.gitbook/assets/image (2).png>)

In this challenge we have to make a S3 bucket public without getting detected, meaning the  application has some sort of security measures to prevent the cloud environment from being tampered with. On a side note : we are emulating an aws environment using localstack, which is a program designed to help beginners to understand aws .&#x20;

Viewing the hints of this challenge we see that a program named checkhov is mentioned . Upon googling this program we see it detects and prevents tampering within cloud deployments. Let's confirm this by actually tampering with the **main.tf** file . This file has configuration of the bucket and it's associated ACL's. &#x20;

I'll clone the repo ,modify the  main.tf file and push the changes&#x20;

![](<.gitbook/assets/image (49).png>)

![](<.gitbook/assets/image (35).png>)



I'll trigger the pipeline manually and the build fails because checkov detected that the acl was tampered

![](<.gitbook/assets/image (16).png>)

The way we bypass this is quite s
