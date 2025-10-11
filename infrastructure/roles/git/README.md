# git

## Create a repository

On the server:

```
git init --bare $REPO
```

## Push an existing repository

```
git remote add $REMOTE_NAME $SERVER:$REPO
git push $REMOTE_NAME main
```

## Exposing via gitweb

```
sudo ln -s $ABSOLUTE_PATH_TO_REPO /var/lib/git/$NAME.git
```

You can use `~/foo` as the `ABSOLUTE_PATH_TO_REPO` to expose a repository in your home directory.

## Exposing via https

```
mv $ABSOLUTE_PATH_TO_REPO/hooks/post-update.sample $ABSOLUTE_PATH_TO_REPO/hooks/post-update
ln -s $ABSOLUTE_PATH_TO_REPO ~/public_html/$REPO.git
```

Ensure that you push once to the repo, or run `git update-server-info` in the repository.
