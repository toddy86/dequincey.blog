---
layout: post
title: Common Git Commands
categories: tutorial
---

A cheat sheet I have created to remind myself of common Git commands. This is obviously not an extensive list of all possible git commands. Just a list of the common ones I use. 

<!-- more -->
<b>Project goal</b> 
  
Learn and summarise common git commands into a cheat sheet. 
  
---
<b>Lessons learnt</b>
<ul>
  <li>How to clone, branch, commit and merge into repositories</li>
</ul>

---
<h3>Creating & Cloning Projects</h3>

| Git Command | Description |
|-------|--------|
| git init | Create an initial git repository locally |
| git cline repository_name.git | Clone an existing remote repository | 


---

<h3>Ignoring Files & Folders</h3>
To ignore certain files and folders from being able to be committed, complete the following steps:

<ul>
	<li>Create a .gitignore file by running the following command from the terminal {% highlight ruby %}touch .gitignore{% endhighlight%}</li>
	<li>Access the .gitignore file, either through a text editor or via the terminal using VIM {% highlight ruby %}vim .gitignore{% endhighlight%}</li>
	<li>Add files and/or folders to the ignore file</li>
</ul>

{% highlight ruby %}
# Terminal command to create a .gitignore file 
touch .gitignore

# Terminal command to edit the file using vim
vim .gitignore

# vim command to insert
i  # The text INSERT will then be displayed at the bottom of the page

# Ignore folders
folder_name1/
folder_name2/

# Ignore a single file
parent_folder/sub_folder/file_name.txt

# Ignore packages
\*.7z
\*.iso
\*.zip

# Ignore a folder, except a single file
folder_name1/*  # Ignore everything in the folder 
!folder_name/file_name.txt  # But don't ignore this specific file

# Quit and don't save changes in vim (first press escape)
:q! # Then press enter

# Quit and save changes in vim (first press escape)
:wq # Then press enter


{% endhighlight%}

<a href="https://git-scm.com/docs/gitignore">gitignore documentation</a>.


---
<h3>Status, Adding & Committing Files</h3>

| Git Command | Description |
|-------|--------|
| git status | Check the status of the local repository / branch | 
| git add file_name | Add a single file to the staging area for committing | 
| git add . | Add all new and changed files to staging area for committing | 
| git commit -m "good commit message" | Commit changes | 

---


<h3>Branching & Merging</h3>

| Git Command | Description |
|-------|--------|
| git branch | List all of the local branches | 
| git branch -a | List all of the local & remote branches | 
| git branch branch_name | Creates a new branch | 
| git checkout branch_name | Switch to a branch | 
| git merge branch_name | Merge a branch into the active one | 
| git merge branch_name target_branch | Merge a branch into another one | 

---

<h3>Pushing & Pulling Updates</h3>

| Git Command | Description |
|-------|--------|
| git pull | Pull all latest committed changes  | 
| git pull branch_name | Pull latest changes froma remote branch | 
| git push origin branch_name | Push a branch into a remote repository | 

---