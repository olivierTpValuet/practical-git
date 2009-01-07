[Practical Git for People and Corporations](..) - Users Guide

# Setting up Git

## on Windows

the recommended tool to use git is mSysGit (here is used release 1.6)
* download its installer from [the home page of mSysGit](http://code.google.com/p/msysgit) (see link "featured" in green box on the right-hand side of that home page)[Download msysGit]
* install it by running its installer and by answering to the following questions: 
    * welcome page: next
    * information page: next
    * destination location: use default directory (not a mandatory), next
    * menu folder: use default value, next
    * additional tasks: at least, check ~~Add "git bash here"~~, next
    * path environment: "Use Git Bash Only" (or "Run Git from Windows Command Prompt" if you have Cygwin), next
    * ssh executable: choose plink, next
    * software is then being installed
* test its installation:
    * if you have cygwin on your box: just open a cygwin/windows prompt
    * else: from windows explorer, right-click on your home folder and choose "Git Bash here" to open a window git prompt
    * `$ git --version => git version 1.6.1.9.g97c34`
* config it (you will not modify directly the configuration file `~/.gitconfig`, but through `git config` that will create the file if first time or update it):
$ git config --global user.name     "<firstName> <LASTNAME>"
$ git config --global user.email    <emailAddress>
$ git config --global core.autocrlf false
$ git config --global color.ui      auto

example with john smith having a nice email domain:

$ git config --global user.name     "John SMITH"
$ git config --global user.email    john.smith@oz.moon
$ git config --global core.autocrlf false
$ git config --global color.ui      auto

For project-based configuration options, remove the `--global` argument. This will update your project's `<path>/<project>/.git/.gitconfig` file instead, and will be shared with the team. Note that project configuration options win over global configuration options.
* test its config:
`$ git config --list
=>
core.symlinks=false
core.autocrlf=true
color.diff=auto
pack.packsizelimit=2g
user.name=<firstName> <LASTNAME>
user.email=<firstName>.<lastName>@<company>.com
core.autocrlf=false
color.ui=auto`

# Quick workflow on local work

git init ; to create a new local repo
<create files>
git add . ; to get all files
git add <files> ; to get only some files
git status ; or git diff --cached
git commit ; note: will prompt for a message
git commit -a -m "<message>" ; = git add . ; git commit -m "<message>"

git branch ; show local branches (the one with a * indicates the current branch)
git branch <name> ; create a new branch
git checkout <branch> ; switch from current branch to branch <branch>
git checkout -b <branch> ; = git branch <branch> ; git checkout <branch>

git merge <branch> ; merge branch <branch> into current branch

# Git Worst Practices

Let's try to debunk bad Git habits here.

* Watch out not to grow the habit of using the `-a` argument to `git commit` mechanically.
    * Let's say you do an interactive staging of some changes to commit, using the wonderful `git add -i`. Then, you `commit -a`. Instead of committing only what you had interactively selected to be committed, you end up committing everything!

# Accessing remote repositories

## preliminary: the ssh key pair

1. if not done yet, open a window git prompt on your home folder:
          * with windows explorer, right click on your home folder <username> => a window popup opens
          * select Git bash here => the window git prompt opens

2. check if folder .ssh exists ($ ls -l .ssh):
    * if no, it means that you never generated your ssh keys, then time to do it!
    * if yes, check if the content of the file id_rsa.pub ends with the chosen email address that you will record in your remote repo server
                o if no, make a backup copy of the already generated keys, both private and public (id_rsa and id_rsa.pub)
                o if yes, see last step

3. generate your ssh keys:
    * with openssh for gitosis or gitorious
    * with putty for github

4. provide your keys regarding the remote repo server:
4.1. in case of use of gitosis:
    * connect as original administrator on ci slave
    * drop public sshkey files into keydir of your local gitosis-admin repository:
        * $ cd <gitosis-admin-home>
        * $ cp ~<username>/.ssh/id_rsa.pub keydir/<firstName>.<lastName>@<domainName>.pub
        * $ sudo chmod 644 keydir/<firstName>.<lastName>@<domainName>.pub
    * link that key into git (by pushing file into remote repository):
        * $ git add keydir/<firstName>.<lastName>@<domainName>.pub
        * $ git commit -m "added authentication for <user> with public sshkey <firstName>.<lastName>@<domainName>.pub"
        * $ git push

4.2. in case of use of gitorious:
    * browse to home page of the web server if your gitorious
    * register yourself on this site
        * use your nickname as your username, and your email address from which you want to be warned
        * you will receive an activation email, containing a url to use to activate your account.
        * when you click on that url, you automatically log into the gitorious web site which tells that your account has been activated.
        * set up your user account:
            * click on My Account
            * click on Edit details and write your Real Name; then Save this change
            * click on Add SSH Key and paste in the textbox the whole contents of your public (*.pub) ssh key
            * when gitorious set you up, you can get an official project from a remote repository

4.3. in case of use of github:
    * check if your default ssh application is PuTTY:

      $ ssh -v
      PuTTY Link: command-line connection utility
      ...

      If it's not PuTTY Link that is called, then:
          o add the path to your PuTTY installation to Windows' PATH
          o in your bash shell:
                + if you use msysgit:
                  mv /bin/ssh.exe /bin/ssh.exe.old
                  ln -s <path-to-PuTTY-in-bash>/plink.exe /bin/ssh.exe

                + if you use cygwin:
                  ln -s <path-to-PuTTY-in-bash>/plink.exe /usr/local/bin/ssh

    * if not done yet, generate your public/private key pair with puttygen (and copy the public key shown in the puttygen window for next step)
    * provide your public key to github (copy from puttygen your public key generated by puttygen, and paste it in your github account thru your github account page)
    * provide your private key to pageant
    * configure a session called github-proxy with putty configuration:
          o set the host to ssh.github.com, port to 443.
          o under Connection -> SSH -> Auth, select your private key file for SSH authentication
          o under Connection -> Proxy, select HTTP and enter appropriate information for the host and port of the proxy, and your proxy username and password
    *
      If ever these tips get out of date, go see this guide on github, but be weary that it's more complicated to find your way through this stuff!

## the repositories themselves

1. use of gitosis:
    * add user in required group of projects:
        * $ vim gitosis.conf:
      [group <aGroupOfProjects>]
        writable = { <relativePathToOneProject> }
    -   members = { <userGitId> }
    +   members = { <userGitId> } <firstName>.<lastName>@<domainName>
        * $ git add gitosis.conf
        * $ git commit -m "granted commit rights for <firstName>.<lastName>@<domainName>.pub on groups { <GroupOfProjects> }"
        * $ git push

2. use of gitorious:
         1. if not done yet, register yourself on this site (see section above)
         2. to clone a project hosted on gitorious: see the project's web page, click on its mainline repository, click on More info... alongside the Public clone url
         3. to create a project:
            * set slug
            * you must push an existing, local repository towards a newly created gitorious project before yourself or anybody else can clone the project from gitorious (until you do this initial push, if someone tries to clone the project from gitorious, git will tell him/her that there's no matching remote HEAD)
         4. each repository provides the basic instructions to help you push to it or clone it

3. use of github:
      * w/o a firewall:
      $ git clone http://github.com/<path>/<project-name>.git
      $ git clone git@github.com:<path>/<project-name>.git

      * with a firewall, this is harder, but still easily done using the following tips:
          o To clone a github-hosted repository to which you don't need to be able to push later, use git with the http protocol:
            http_proxy=http://<proxy-usermane>:<proxy-password>@172.22.176.47:3128 git clone http://github.com/<path>/<project-name>.git

            The output produced should look like this:
            Initialized empty Git repository in /<path>/<project-name>/.git/
            ...
            Getting pack list for http://github.com/<path>/<project-name>.git/
            ...
            walk 5fcb34d0f815ed5b8cf9ee35a77692f5c1ed7f84
            Checking out files: 100% (678/678), done.

          o To clone a github-hosted repository which you will want to pull/push from/to later:
                + you need to fork it under your own github account. See github's guides for this easy step
                + create a github-proxy thru putty as indicated above
                + when your github account has its own fork of the project, use git with ssh instead of http:
                      git clone git@github-proxy:<path>/<project-name>.git

            The output produced will look different than if you cloned over http:
            Initialized empty Git repository in <path>/<project-name>/.git/
            remote: Counting objects: 999, done.
            remote: Compressing objects: 100% (706/706), done.
            Receiving objects: 100% (999/999), 2.37 MiB | 27 KiB/s, done.
            Resolving deltas: 100% (282/282), done.
            Checking out files: 100% (678/678), done.

4. general use:
      $ git clone       git://<networked-box>/<path>/<project-name>.git
      $ git clone       <git>@<networked-box>:<path>/<project-name>.git
      $ git clone     rsync://<networked-box>/<path>/<project-name>.git
      $ git clone  http://<git.networked-box>/<path>/<project-name>.git
      $ git clone                             <path>/<project-name>.git

# Full project repository workflow

* John Doe: Boss, I need to work on something!
* Boss: John, you'll work on <project>!

> Best practice:
    **An official repository for a project contains at least two branches: one stable, and one for staging purposes.**
    The staging branch is where changes are tried before being merged into the stable branch.
    In this example, it's called *edge*.

                         |                      |           OFFICIAL
                         |                      |
                         |                      |             stable
                         |                      |               edge

## Setting up a local repository

* John Doe: Boss, I'm setting up for <project>!

### Checking out from a repository

> Best practice:
    **Clone/fetch/pull from the official repository.**
    Stick to this even if you forked the official repository into another public repository.

John runs one of the following options to checkout the project along with its history:

    1$ git clone  git://<host>/<path>/<project>/official.git
    1$ git clone http://<host>/<path>/<project>/official.git
    1$ git clone (SSH: XXXX to be documented later)

1- Checks out an initial local copy of the OFFICIAL repository.

    JD-LOCAL             |                      |           OFFICIAL
                         |                      |
                 stable <------------------------------------ stable
                   edge <------------------------------------   edge


If John already had a public fork of this project, he would have done the same:

    1$ git clone git://<host>/<path>/<project>/official.git

1- Note that the Git URL for the clone references the OFFICIAL repository and **not** JD-FORK.

    JD-LOCAL            |       JD-FORK       |          OFFICIAL
                        |                     |
                stable <---------------------------------- stable
                        |             stable  |
                  edge <----------------------------------   edge
                        |               edge  |

### Setting up tracking of a remote branch

> Best practice:
    **Automate the synchronization of official remote branches towards your local repository.**
    Stick to this even if you forked the official repository into another public repository.
    This way, your working copy will always be the integration point of what's coming from the official repository, and it won't hinder your fork repository in perpetually benefiting from the official changes to the project, although it might not be clear to you now.

    1$ cd <project>
    2$ git branch --track edge origin/edge

1- Gets inside the project's working tree to interact with git (and with the project, of course).  
2- Sets up automatic tracking of the OFFICIAL edge branch, for future easy syncing.

Note that we don't need to explicitly track the OFFICIAL stable branch; this is automatically, implicitly done when you `git clone`.

## Setting up topic branches

> Best practice:
    **Make a topic branch for each future feature pushed to the official repository.**
    Try to give the topic branch a name that properly synthesizes the nature of the change.

* John Doe: Hey Boss, I'll work on *topic-1* and *topic-2*!
* Boss: Fine, go ahead!

### Creating a local topic branch

> Best practice:
    **format-branch-names-this-way** and, if they're shared, prefix their name with a parent directory whose name is the initials of the branch's author: **jd/topic-description**

    1$ git branch   <topic> edge
    2$ git checkout <topic>

or, as a shortcut:

    1+2$ git branch -b <topic> edge

1- Creates a local branch for *topic*, based off of edge.  
2- Selects the local *topic* branch, to work on it.

    JD-LOCAL             |                      |           OFFICIAL
                         |                      |
                 stable  |                      |             stable
                   edge  |                      |               edge
                         |                      |
                topic-1  |                      |
                topic-2  |                      |

#### Setting up the branch name in shell prompt

TODO:[Try it](http://github.com/guides/put-your-git-branch-name-in-your-shell-prompt) and it works, document it :)

## Setting up a sharing repository

> Best practice:
    **Work on a local repository and frequently push your changes to another repository.**
    In this example, the second repository is called **JD-FORK**.
    It will serve you as a sharing access point for you towards your team, and as a redundant backup.

### Setting up a public fork

John sends his public RSA key to his sysadmin, and asks him to create a sharing repository for him.

    1$ cd <path>/<project>
    2$ git clone --bare git://<host>/<path>/<project>/official.git jd-fork.git

2- Makes a copy of OFFICIAL, without a working tree.

    JD-LOCAL             |        JD-FORK       |           OFFICIAL
                         |                      |
                 stable  |              stable <------------- stable
                   edge  |                edge <-------------   edge
                         |                      |
                topic-1  |                      |
                topic-2  |                      |

### Setting up a remote branch in the fork repository

    1$ git checkout   <topic>
    2$ git remote add <topic> JOE@<host>:<path>/<project>/jd-fork.git
    3$ git fetch      <topic>

1- Selects the local *topic* branch, to track the future changes.  
2- Sets up a tracking reference of the local *topic* branch unto JD-FORK, for future easy syncing.  
3- Syncs up the local *topic* branch *from* JD-FORK, as a safety measure. (Probably to see if there's already a shared branch of the same name.)

    JD-LOCAL             |        JD-FORK       |           OFFICIAL
                         |                      |
                 stable  |              stable  |             stable
                   edge  |                edge  |               edge
                         |                      |
                topic-1  |        (jd/topic-1)  |
                topic-2  |        (jd/topic-2)  |

### Syncing a local topic branch to a fork repository

    1$ git push <topic> JOE:refs/heads/<topic>

1- Syncs up the local *topic* branch towards JD-FORK, effectively creating it on JD-FORK.

    JD-LOCAL             |        JD-FORK       |           OFFICIAL
                         |                      |
                 stable  |              stable  |             stable
                   edge  |                edge  |               edge
                         |                      |
              topic-1 ------------> jd/topic-1  |
              topic-2 ------------> jd/topic-2  |

### Making future branch syncs automatic

    1$ git config branch.<topic>.remote            <john>
    2$ git config branch.<topic>.merge  refs/heads/<john>

1- Makes future `git push` commands automatically sync the topic branch to JD-FORK.  
2- Makes future `git pull` commands automatically sync the topic branch from JD-FORK to JD-LOCAL.

## Working on topic branches

* John: Let's work on *topic-1*!

### Making changes on a local topic branch

    1$ git checkout <topic>

1- Selects the local branch *topic*, to track the related changes.

> Git's lightweight operation of checking out branches is something that needs to be well understood.
  If you check out a branch, your working directory will need to look 100% like that branch.
  Otherwise, you couldn't say your working tree represents that branch.
  Therefore, Git will need to remove any difference in content that is currently in your working tree but is not in the branch's tree.
  Thankfully, Git doesn't want you to loose uncommitted changes, so it will only let you checkout another branch if everything in your current working tree is committed.
  This means that if you have uncommitted changes in your working directory and want to checkout another branch, you won't be able to do so until you commit or stash them away.

    JD-LOCAL             |        JD-FORK       |           OFFICIAL
                         |                      |
                 stable  |              stable  |             stable
                   edge  |                edge  |               edge
                         |                      |
                topic-1' |          jd/topic-1  |
                topic-2  |          jd/topic-2  |

### Backing up a local topic branch to the fork repository

* John: Let's call this a day; time to go home!

Run:

    1$ git commit -m "leaving home"
    2$ git push

2- Syncs everything towards JD-FORK, thanks to John's usage of best practices when he set up his repository. Note that at the same time, the stable and edge branches are automatically synced also. This behavior is automatic and implicit for the stable branch; any other branch that's synced at the same time is done because we have previously set it up this way.

    JD-LOCAL             |        JD-FORK       |           OFFICIAL
                         |                      |
                 stable --------------> stable  |             stable
                   edge -------------->   edge  |               edge
                         |                      |
                topic-1'----------> jd/topic-1' |
                topic-2 ----------> jd/topic-2  |

JD-FORK's stable and edge branches are now in sync *with the state of OFFICIAL at the time of the last sync from OFFICIAL to JD-LOCAL*. This is what we want. It clearly highlights on which official revision of the edge branch John's *topic* branches are based. (There's probably more reasons for using this approach, too.)

#### Restoring a local topic branch from the fork repository

* *Jack*: Yes! Now that John has left for home, it's time to make him pay for the last *public* humiliation he made me suffer!

Run:

    $ #hack hack hack...
    $ git branch -d topic-1    # -d means "delete"

* *Jack*: HahahahahahahaHAAAAAAAA!

The sun goes down and up, and John comes back to work. For *some* reason, he wonders what happened to his *topic-1* branch. But anyway, he doesn't care too much:

    1$  git pull

1-  Syncs everything from JD-FORK, thanks to John's usage of best practices when he set up his repository.

    JD-LOCAL             |        JD-FORK       |           OFFICIAL
                         |                      |
                 stable <-------------- stable  |             stable
                   edge <--------------   edge  |               edge
                         |                      |
                topic-1'<---------- jd/topic-1' |
                topic-2 <---------- jd/topic-2  |

## Asking a colleague for help

* John: Ouch, I can't believe how this HTML programming language is hard to understand!

Luckily for John, Suzy Kue knows quite a deal about HTML.

* Boss: John, I thought you might need help with this new HTML thingy. If that ever happens, I think Suzy Kue might be the good candidate to ask for help. Our office in Japan recruited her recently.
* John: Oh thank you boss, but I should be fine, you know.

John then waits for the good moment to send an email to *super Suzy*, when the boss won't be near his screen. And meanwhile, he pushes his latest changes to his public repository:

    $ git commit
    $ git push

Results in:

    JD-LOCAL             |        JD-FORK       |           OFFICIAL
                         |                      |
                 stable --------------> stable  |             stable
                   edge -------------->   edge  |               edge
                         |                      |
                topic-1'----------> jd/topic-1' |
                topic-2 ----------> jd/topic-2  |

Sometime later, *the* moment comes for John to write an email to Suzy, asking her to "take a look at this code if you don't mind: `git://<...>/jd-fork.git, *branch jd/topic-1*`".

Luckily  again for John, 5 minutes later, he receives an answer from Suzy, telling him to "have a look at my proposal for a fix, at: `git://<...>SK-FORK.git, *branch jd/topic-1*`". John eagerly takes a look at Suzy's changes:

    1$ git fetch jd/topic-1 git://<...>SK-FORK.git topic-1-sk
    2$ git diff  topic-1 topic-1-sk
    3$ git merge topic-1-sk

1- John doesn't use "git pull" because he's not sure yet if he likes Suzy's patch; so he fetches it as *topic-1-sk*.  
2- A quick `diff` makes sense.  
3- John likes Suzy's patch; therefore, he merges it into the current branch.

    JD-LOCAL             |        JD-FORK       |           OFFICIAL
                         |                      |
                 stable  |              stable  |             stable
                   edge  |                edge  |               edge
                         |                      |
                   /------------ jd/topic-1-sk  |
                   |     |                      |
                   V     |                      |
                topic-1' |          jd/topic-1' |
                topic-2  |          jd/topic-2  |

Now, what John doesn't know yet is that anyone may see that Suzy helped him with this patch.

## Testing development changes with Hudson

* John: Boss will be impressed seeing *my* mastery of HTML! Let's make sure, first, that our continuous integration tests won't fail.

John is confident his changes should work fine.

    $ git commit
    $ git push jd/topic-1 git://<host>/<path>/<project>-ci-dev.git

* John: Cool, tests pass!

## Requesting a pull from the official repository

Seeing *topic-1* is now feature-complete, John puts the finishing touches to the branch's code and prepares it to be integrated into OFFICIAL.

    $ git commit
    $ git push

    JD-LOCAL             |        JD-FORK       |           OFFICIAL
                         |                      |
                 stable --------------> stable  |             stable
                   edge -------------->   edge  |               edge
                         |                      |
                topic-1'----------> jd/topic-1' |
                topic-2 ----------> jd/topic-2  |

* John: Boss, *topic-1* is done!
* Boss: Great, John! Let's have a look at it! By the way, did you need Suzy's help, finally?
* John: What for!?

## Having a look at someone else's branch

Boss pulls *topic-1* on his BOSS-LOCAL from JD-FORK, and takes a look at it...

    1$  git stash save my-actual-work
    2$  git pull
    3$  git remote add jd git://<...>/jd-fork.git
    4$  git checkout -b edge<topic>1 edge
    5$  git pull jd jd/topic-1
    6$  git diff edge edge<topic>1
    7$  git branch
    8$  git checkout my-previous-branch
    9$  git stash list
    10$ git apply my-actual-work

1-  Puts away uncommitted changes temporarily  
.  
.  
.  
10- Restores some uncommitted changes that had been put away previously

* Boss: Good work, John!

## Pulling a topic branch on the official repository

* Boss: Let's edge John's *topic-1* branch on OFFICIAL!

Boss logs on OFFICIAL, pulls *topic-1* from JD-FORK, and waits for Hudson:<project>:edge to tell him all tests have passed. 

...All tests pass. Boss commits *topic-1* to edge.

    JD-LOCAL             |        JD-FORK       |           OFFICIAL
                         |                      |
                 stable  |              stable  |             stable
                   edge  |                edge  |              edge'
                         |                      |                ^
                         |                      |                |
                topic-1' |          jd/topic-1'------------------/
                topic-2  |          jd/topic-2  |

## Rinsing and repeating

Meanwhile, John finishes his work on *topic-2*...


    JD-LOCAL             |        JD-FORK       |           OFFICIAL
                         |                      |
                 stable <------------------------------------ stable
                         |              stable  |
                         |                      |
                  edge' <------------------------------------  edge'
                         |                edge  |
                         |                      |
                         |          jd/topic-1' |
               topic-2'  |          jd/topic-2  |

And then:

    JD-LOCAL             |        JD-FORK       |           OFFICIAL
                         |                      |
                 stable ------------->  stable  |             stable
                  edge' ------------->   edge'  |              edge'
                         |                      |
                         |                      |
                         |                      |
                       --------->               |
               topic-2'--------->   jd/topic-2' |

TODO:If the remote branch doesn't get removed like I showed in this graph (I'm yet to test this), try [this](http://github.com/guides/remove-a-remote-branch) and please update this section according to the truth.

Then Boss takes a look at *topic-2*; since it looks good to him and Hudson:<project>:edge is happy with it, Boss commits *topic-2* on OFFICIAL's edge.

    JD-LOCAL             |        JD-FORK       |           OFFICIAL
                         |                      |
                 stable  |              stable  |             stable
                  edge'  |               edge'  |             edge''
                         |                      |               ^
                         |                      |               |
               topic-2'  |         jd/topic-2'------------------/

## Making a new release

 * Boss: Congrats Team, OFFICIAL's edge is stable, let's call this a release!

Boss tries a merge of edge with stable on OFFICIAL, and waits for Hudson:<project>:*stable* to tell him all tests pass.

Boss commits and tags this new release.

    JD-LOCAL             |        JD-FORK       |           OFFICIAL
                         |                      |
                 stable  |              stable  |            stable'
                         |                      |               ^
                         |                      |               |  
                  edge'  |               edge'  |             edge''
                         |                      |
               topic-2'  |         jd/topic-2'  |

## Rinsing and repeating

Some time later, John starts work on *topic-3*...

    JD-LOCAL             |        JD-FORK       |           OFFICIAL
                         |                      |
                stable' <----------------------------------- stable'
                         |              stable  |
                 edge'' <-----------------------------------  edge''
                         |               edge'  |
                         |                      |
                         |         jd/topic-2'  |
                topic-3  |                      |

## Grab the following help scripts!

## Advantages of using this workflow

The advantages of this set of practices are many:

* Everyone works within their own repository
* Everyone works on their own schedule
    * There's no process waiting to be completed that blocks *me* from moving on to whatever *I* need/want to do next
    * *I'm* not forcing anyone to drop what they're doing right now to handle *my* pull requests

* Moreover:
    * Each repository is on an equal footing
        * In particular, we would like every fork to have the same stable branch, so that if the official repository should ever be lost, there would be plenty of redundant backups
    * We also want it to be easy for each developer to pull in changes from the official repository
    * Finally, it's a bad idea in general to work on the stable branch; experienced git users typically work on separate development branches and then merge those branches into stable when they're done

* The extra work is worth the effort, because with this configuration:
    * *My* changes will be easily identifiable in *My* named branch
    * *I* can easily get updates from the official repository
    * Any updates *I*'ve pulled into stable and edge are automatically pushed up to *my* fork on the server
    * The simple 'git push' command will push up changes for all local branches that have a matching branch on the remote
    * If *I* make it a point to pull in updates to my local stable and edge but not work directly on them, my fork will match up with the official repository

* So what is the benefit of all this to our team's projects?
    * The easier it is for *me* to pull in updates, the more likely it will be that the pull request will be for code that merges easily with the latest releases
    * *I* can tell if someone is pulling updates by looking at their stable and edge branches and seeing if they match up with the latest branches on the official repository
    * By getting *myself* in the habit of working on branches, the team is going to get better, more organized code contributions

# References

* **General**
    * [Git-SCM](http://www.git-scm.com) - Download, Community Book, Tutorials, Reference
    * [GitHub Guides](http://github.com/guides/home)

* **Cheat Sheets**
    * [Git Cheat Sheet](http://zrusin.blogspot.com/2007/09/git-cheat-sheet.html) - see this page's attachments for a cleanly resized version

* **Tutorials**
    * [Official - Tutorial](http://www.kernel.org/pub/software/scm/git/docs/gittutorial.html)
    * [Official - Git for SVN Users](http://git.or.cz/course/svn.html)
    * [Official - Everyday Git with 20 commands](http://www.kernel.org/pub/software/scm/git/docs/everyday.html)
    * [Git for The Lazy](http://www.spheredev.org/wiki/Git_for_the_lazy)
    * [A Tour of Git - The Basics](http://cworth.org/hgbook-git/tour)
    * [An introduction to git-svn for SVN deserters](http://utsl.gen.nz/talks/git-svn/intro.html)

* **Links**
    * [Git Resources](https://37s.backpackit.com/pub/1465067)
    * [Git-SCM's Documentation Links](http://www.git-scm.com)

* **Tutorials - More advanced stuff**
    * [Git Awesomeness - Git Rebase --interactive](http://blog.madism.org/index.php/2007/09/09/138-git-awsome-ness-git-rebase-interactive)
    * [The Thing About Git](http://tomayko.com/writings/the-thing-about-git)
    * [Official - Git How To](http://www.kernel.org/pub/software/scm/git/docs/howto-index.html) - Advanced stuff

* **Reference**
    * [Official - Man Pages](http://www.kernel.org/pub/software/scm/git/docs/)
    * [Official - FAQ](http://git.or.cz/gitwiki/GitFaq)
    * [Git for The Confused](http://www.gelato.unsw.edu.au/archives/git/0512/13748.html)
    * [Git-SCM's Documentation Links](http://www.git-scm.com)

* **Screencasts** - A great, quick way to understand Git
    * [Git Casts](http://gitcasts.com/)
    * Community Book (through Git-SCM site link upwards) - in HTML version only
    * [Peepcode's Git Screencast](http://peepcode.com/products/git) (Costs 9$)

* **Deep Understanding / Books**
    * [Git for Computer Scientists](http://eagain.net/articles/git-for-computer-scientists/) - small
    * [Git From Bottom Up](http://www.newartisans.com/blog_assets/git.from.bottom.up.pdf) (PDF) - Useful, deep understanding coverage
    * Git From Bottom Up (PDF with my yellow highlightings) - see this page's attachments to download it
    * [Git Magic](http://www-cs-students.stanford.edu/~blynn/gitmagic/)
    * [Git-SCM's Community Book](http://www.git-scm.com) - Always up-to-date; Wide coverage
    * [Peepcode's Git Internals Book](http://peepcode.com/products/git-internals-pdf) (Costs 9$) - Complementary; Wide coverage

* **Workflow**
    * [Our Git Deployment Workflow](http://www.brynary.com/2008/8/3/our-git-deployment-workflow)

* **Hosting**
    * [GitHub](http://www.github.com) - Commercial Web Front; Free only for Public-Small; Highly social site; Very useful; Pleasing to use
    * [Gitorious](http://www.gitorious.org) - Open Source Web Front; Free hosting
    * [Repo.or.cz](http://repo.or.cz) - Git Web Front - Free hosting

* **Java**
    * JGit - The most complete implementation of Git in any other language (Java).
    * [Maven-enabled project hosting with GitHub](http://www.jroller.com/mrdon/entry/maven_enabled_project_hosting_with)

* **Best Practices**
    * [Setting up your repositorys for shared projects](http://blog.insoshi.com/2008/10/14/setting-up-your-git-repositories-for-open-source-projects-at-github/)
