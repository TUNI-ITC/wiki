# Version Control
It is important that all your **source code** and **documents** are in version control since one day your computer will break down and that day you will thank yourself having them under version control. There are many other reasons, but that is the day when it really pays off.

Below are instruction for various different purposes.

## Personal version control using university ssh servers
This is what you should do for your personal documents, such as CV, job applications, list of publications, love letters etc. that you feel uncomfortable to put on public servers such as [GitHub](https://github.com), but that you wish to be safe, versioned and backup.

For that purpose the university ssh servers and Subversion provide suitable tools. First, connect *id.company.org* and obtain rights to use these services (typically via managing your user rights, IT services and their Linux servers). Wait a few minutes and you should access to the servers if the process is automatic.

Create personal repository:

    $ ssh linux-ssh.company.org
    $ mkdir svn_repos; cd svn_repos
    $ mkdir <personal_dir>
    $ svnadmin create /home/<my name>/svn_repos/<personal_dir>

Now the repository is created and you can access it from your personal computer. First take out the repo:

    $ cd Work
    $ svn co svn+ssh://<my name>@linux-ssh.company.org/home/<my name>/svn_repos/<personal_dir>

Now you have personal repository that is stored and backup in the university system. SVN (Subversion) is pretty similar to Git, but simpler. For example, you don't "push" and "pull", but use "svn commit" to add you changes and "svn update" to update changes to the existing repository. 

A Fine **SVN BOOK** is available here [http://svnbook.red-bean.com/](http://svnbook.red-bean.com/)
 
## Research project code and documents in Github

Someone needs to write.

