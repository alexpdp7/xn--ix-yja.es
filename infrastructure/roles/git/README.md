# git

## Create a repository

On the server:

```
mkdir -p ~/git
cd ~/git
git init --bare $REPO
```

## Push an existing repository

```
git remote add $REMOTE_NAME $SERVER:git/$REPO
git push $REMOTE_NAME main
```

