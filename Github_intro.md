# How to collaborate on the DSM metadata git

## First, only once
- Create (or use your current) account on github, and let me know your username, so I can add you as a collaborator.
- Make sure to install [git](https://git-scm.com/install/windows) on you laptop. As easy as opening your shell (on windows) and running
```
winget install --id Git.Git -e --source winget 
```


- Go to the DSM meta stack [github](https://github.com/collinvrooij/AU_DSM_metadata) page, click on the green Code button and copy the https
- Go to the desired folder location on your local pc in the file explorer, where you want your local version of the AU_DSM_metadata folder to live
- When you are there, right click somewhere in the folder and click Open in Terminal. So you will see something like this
![[Pasted image 20260420084427.png]]

Then in the terminal type in
```
  git clone https://github.com/collinvrooij/AU_DSM_metadata.git    
```

This will copy the repository and all its files to your local PC, where you can adjust them.
## Every time you want to change something
So now you have a local clone of the git repository. This means you just have a local copy snapshot of the github repo. Every time you, or someone else wants to change something in the files, you have to make sure you have the latest version of the github repo. You do this by opening the terminal, and moving towards your local git folder. Then in here you type in
```
git fetch
git pull
```
This will copy all the changes others made to the github repo to your local version.
If you dont do this, you might encounter file conflicts, when you will try to push your changes to the github repo, we dont want that :) 

After fetching, you can just alter the files locally, as you see fit.

**Important**
Always add your changes to the changelog.md file. Here you should be elaborate and as complete as possible. 

After making your changes, open the terminal again and move to your local git repo. Then type in 
```
git add . 
git commit -m "Your message"
git push
```
where the dot behind add indicates you want to add all the files you changed. You can also specify which files you want to add to the git repo. Make sure your message in between brackets is short, so just. 

"Included recent addition of S2 composites" or "Included short section of data tiling"