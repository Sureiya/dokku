# Deploying to Dokku

## Deploy tutorial

Once Dokku has been configured with at least one user, applications can be deployed via a `git push` command. To quickly see Dokku deployment in action, you can use the Heroku Ruby on Rails example app.

```shell
# from your local machine
git clone git@github.com:heroku/ruby-rails-sample.git
```

### Create the app

Create the application on the Dokku host. You will need to ssh onto the host to run this command.

```shell
# on your dokku host
dokku apps:create ruby-rails-sample
```

### Create the backing services

When you create a new app, Dokku by default *does not* provide any datastores such as MySQL or PostgreSQL. You will need to install plugins to handle that, but fortunately [Dokku has official plugins](/dokku/plugins/#official-plugins-beta) for common datastores. Our sample app requires a PostgreSQL service:

```shell
# on your dokku host
# install the postgres plugin
# plugin installation requires root, hence the user change
sudo dokku plugin:install https://github.com/dokku/dokku-postgres.git

# create a postgres service with the name rails-database
dokku postgres:create rails-database
```

> Each services may take a few moments to create.

### Linking backing services to applications

Once the service creation is complete, set the `POSTGRES_URL` environment variable by linking the service.

```shell
# on your dokku host
# each official datastore offers a `link` method to link a service to any application
dokku postgres:link rails-database ruby-rails-sample
```

> You can link a single service to multiple applications or use one service per application.

### Deploy the app

Now you can deploy the `ruby-rails-sample` app to your Dokku server. All you have to do is add a remote to name the app. Applications are created on-the-fly on the Dokku server.

```shell
# from your local machine
git remote add dokku dokku@dokku.me:ruby-rails-sample
git push dokku master
```

You should see output similar to the following:

```
Counting objects: 231, done.
Delta compression using up to 8 threads.
Compressing objects: 100% (162/162), done.
Writing objects: 100% (231/231), 36.96 KiB | 0 bytes/s, done.
Total 231 (delta 93), reused 147 (delta 53)
-----> Cleaning up...
-----> Building ruby-rails-sample from herokuish...
-----> Adding BUILD_ENV to build environment...
-----> Ruby app detected
-----> Compiling Ruby/Rails
-----> Using Ruby version: ruby-2.2.1
-----> Installing dependencies using 1.9.7
       Running: bundle install --without development:test --path vendor/bundle --binstubs vendor/bundle/bin -j4 --deployment
       Fetching gem metadata from https://rubygems.org/...........
       Fetching version metadata from https://rubygems.org/...
       Fetching dependency metadata from https://rubygems.org/..
       Using rake 10.4.2

...
```

When the deploy finishes, the application's URL will be shown.

```shell
=====> Application deployed:
       http://ruby-rails-sample.dokku.me
```

Dokku supports deploying applications via [Heroku buildpacks](https://devcenter.heroku.com/articles/buildpacks) with [Herokuish](https://github.com/gliderlabs/herokuish#buildpacks) or using a project's [dockerfile](https://docs.docker.com/reference/builder/).

### Removing a deployed app

You can also remove an application from your Dokku installation. This will unlink all linked services and destroy any config related to the application. Note that linked services will retain their data for later use (or removal).

```shell
# on your dokku host
# replace APP with the name of your application
dokku apps:destroy APP
```

This will prompt you to verify the application's name before destroying it. You may also use the `--force` flag to circumvent this verification process:

```shell
# on your dokku host
# replace APP with the name of your application
dokku --force apps:destroy APP
```

### Renaming a deployed app

> New as of 0.4.7

You can rename a deployed app using the `apps:rename` CLI tool:

```shell
# on your dokku host
dokku apps:rename OLD_NAME NEW_NAME
```

This will copy all of your app's contents into a new app directory with the name of your choice, delete your old app, then rebuild the new version of the app and deploy it. All of your config variables, including database urls, will be preserved.

### Deploying non-master branch

Dokku only supports deploying from its master branch, so if you'd like to deploy a different local branch use: ```git push dokku <local branch>:master```

You can also support pushing multiple branches using the [receive-branch](/dokku/development/plugin-triggers/#receive-branch) plugin trigger in a custom plugin.

### Skipping deployment

If you only want to rebuild and tag a container, you can skip the deployment phase by setting `$DOKKU_SKIP_DEPLOY` to `true` by running:

``` shell
# on your dokku host
dokku config:set ruby-rails-sample DOKKU_SKIP_DEPLOY=true
```

### Deploying with private git submodules

Dokku uses git locally (i.e. not a docker image) to build its own copy of your app repo, including submodules. This is done as the `dokku` user. Therefore, in order to deploy private git submodules, you'll need to drop your deploy key in `/home/dokku/.ssh/` and potentially add github.com (or your VCS host key) into `/home/dokku/.ssh/known_hosts`. The following test should help confirm you've done it correctly.

```shell
# on your dokku host
su - dokku
ssh-keyscan -t rsa github.com >> ~/.ssh/known_hosts
ssh -T git@github.com
```

Note that if the buildpack or dockerfile build process require ssh key access for other reasons, the above may not always apply.

## Deploying to subdomains

The name of remote repository is used as the name of application to be deployed, as for example above:

    $ git remote add dokku dokku@dokku.me:ruby-rails-sample
    $ git push dokku master

Is deployed to,

    remote: -----> Application deployed:
    remote:        http://ruby-rails-sample.dokku.me

You can also specify fully qualified names, say `app.dokku.me`, as

    $ git remote add dokku dokku@dokku.me:app.dokku.me
    $ git push dokku master

So, after deployment the application will be available at,

    remote: -----> Application deployed:
    remote:        http://app.dokku.me

This is in particular useful, then you want to deploy to root domain, as

    $ git remote add dokku dokku@dokku.me:dokku.me
    $ git push dokku master

    ... deployment ...

    remote: -----> Application deployed:
    remote:        http://dokku.me

## Dokku/Docker Container Management Compatibility

Dokku is, at its core, a docker container manager. Thus, it does not necessarily play well with other out-of-band processes interacting with the docker daemon. One thing to note as in [issue #1220](https://github.com/dokku/dokku/issues/1220), dokku executes a cleanup function prior to every deployment. This function removes all exited containers and all 'unattached' images.

### Adding deploy users

See the [user management documentation](/dokku/deployment/user-management).

## Default vhost

See the [nginx documentation](/dokku/nginx/#default-site).

## Dockerfile deployment

See the [dockerfile documentation](/dokku/deployment/dockerfiles/).

### Specifying a custom buildpack

See the [buildpack documentation](/dokku/deployment/buildpacks/).

## Image tagging

See the [image tagging documentation](/dokku/deployment/images).

## Zero downtime deploy

See the [zero-downtime deploy documentation](/dokku/checks-examples/).
