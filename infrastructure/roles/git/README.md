# git

## Create a repository

On the server:

```
git init --bare $REPOSITORY_PATH
```

## Push an existing repository

```
git remote add $REMOTE $SERVER_HOSTNAME:$REPOSITORY_PATH
git push $REMOTE main
```

## Exposing via gitweb

```
sudo ln -s $REPOSITORY_PATH /var/lib/git/$NAME.git
```

## Exposing via https

```
mv $REPOSITORY_PATH/hooks/post-update.sample $REPOSITORY_PATH/hooks/post-update
ln -s $REPOSITORY_PATH ~/public_html/$REPO.git
```

Ensure that you push once to the repo, or run `git update-server-info` in the repository.
