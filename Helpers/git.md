`git config --global credential.helper store`
tells Git:

- **“Save my username + PAT after I enter them once, and reuse them automatically later.”**
    
- It saves them in a file on your computer (usually `~/.git-credentials`), so future `git pull/push` (and the Obsidian Git plugin) won’t keep prompting you.
  
