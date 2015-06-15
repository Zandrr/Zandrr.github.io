---
layout: post
title:  "Common GitHub Questions"
date:   2015-06-10 09:25:21
categories: jekyll update
---


###Overview
 Git and Github are not as mind numbing as staring at your cat while she stares at the wall, or spending an entire day counting freckles, it's a bit more complicated than that. In this post I will go over a few common questions that users run into in their quests to become non-cat-staring Git and GitHub gurus.


### Forking vs Branching

> What are some good use cases for repository forking versus branching? Both of these can have code merged back by pull requests. When should we pick one model over another? 



Forking and Branching are both used for merging new code into an existing repository.  The main difference is that you **must** be a collaborator in order to branch from the original repository. Whereas a fork is a clone of the original repository and you do not need to be a collaborator to fork.


######When You Should Fork
1. If you are not a collaborator for a repository and you want to suggest changes to improve the project.  


######When You Should Branch
1. If you are a collaborator and you want to start work on a new feature for the project.

> Note: GitHub best practices say that you should always work on new features within a branch, even after forking.

### Individual files

> How could I retrieve just one file in a repo with Git? 


######If you mean that you want to pull one file from a repo to be used elsewhere, try the following:


  1. Clone the repository that contains the one file you want to your local machines.
  2. Navigate to the file you want.
  3. Copy the file you want into another folder. 
  4. Delete the entire cloned repository on your local machine, and you are left with the one file you wanted.


######If you mean that your local branch is behind by at least one commit and you want to retreive an update to only one file on your local copy, try the following:



1. ``` git fetch ```
   

   to pull down all the differences between your local repository and the remote repository
2. ``` git checkout origin/master -- < filename.js > ```
   

   to pull only filename.js into staging.

We then have a branch that is behind master for all files except filename.js

### Pull request model

> The pull request model suggests that one person or group is responsible for approving updates to a repository. How can this work, in practice, when there are many contributors as well as many decision-makers? 



Git best practices say that when a pull request is opened on a project, a collaborator (other than whomever submitted the PR) must review the changes and approve the request before any code is merged into master. A project can have any number of contributors but the collaborators (or gatekeepers) should be limited to a trusted few. 


### Pull request vs pull command

> Is a Pull Request on GitHub different from the git pull command? How are they different? 


######Pull Request
A Pull Request is a developer asking a collaborator of a given repository to initiate a ```git pull``` using the developers new code. It's used when someone that does not have collaborator rights has made changes to a repository on her local repository and wants those changes she made to be reflected on the repository that lives on GitHub.com.


######Pull Command
The git pull command is used when a someone wants to make her local repository look just like the one on the server side. Collaborator access is not needed to do a ```git pull```.


### Reducing repo size

> How can I reduce the size of a repo after updating several versions of large files, such as graphic files? Can I eliminate the old version from history? What impacts could this have on my colleagues? 



Yes, we would use an operation called 'git-filter-branch'.  This is used to rewrite branches from the past.  Although, git-filter-branch is a bit cumbersome, so I would recommend using  [BFG Repo Cleaner](http://rtyley.github.io/bfg-repo-cleaner/), a faster, simpler alternative to the git-filter-branch command.  We can use BFG to remove huge files in our repository's history.  Let's say we had a huge file called massive_image.psd and we want to delete all past versions of it, but keep the current version.


We would run: ```bfg --delete-files massive_image  my-repo.git```.  This is saying delete all files named massive_image from the past in the repository named my-repo.git.  By default BFG will protect the HEAD branch, the most recent commit will not be touched.  Your colleagues would no longer have access to past versions of this .psd file.  If they wanted to go back to an old version, it would not be possible. 


If you are working with a bunch of huge graphics files, another service might fit your needs a bit better.  GitHub has a disk quota of 1 GB per repository to make sure things are speedy.  I would reccomened using Dropbox or Google drive to handle a many large files.


###  Merging complications

> If there are 3 people who merged in Pull Requests to master, but one is discovered to be defective and needs to be undone/removed from master, is that possible? Are there any risks to this?




Yes that's totally possible.  We could simply revert the merged pull request from history.

1. Here we have three pull requests, and we want to delete *SECOND PULL REQUEST*
![image of first revert](http://alexcampbell.co/images/revert1.png)


2. First we would navigate to our closed pull requests.  As you can see, there are three closed pull requests.  Let's click on the pull request we want to remove, *SECOND PULL REQUEST*
![image of second revert image](http://alexcampbell.co/images/revert2.png)

3. We would then click the *Revert* button that has a red box around it.
![image of third revert image](http://alexcampbell.co/images/revert3.png)

4. We would create a new pull request to remove our *SECOND PULL REQUEST*.
![image of fourth revert image](http://alexcampbell.co/images/revert4.png)


Once we merge this pull request in, it's as if the *SECOND PULL REQUEST* never existed.  Problem solved! :+1:
