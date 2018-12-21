# openshift-gitea
Deploy Gitea on Openshift

## About
Gitea is a Git server build with Go: https://gitea.io

This project provide a template gitea-persistant.yaml to deploy the official docker image of gitea on a openshift V3.
You can find instruction of the gitea image here: https://docs.gitea.io/en-us/install-with-docker/ and their github repos there: https://github.com/go-gitea/gitea

This template has been tested on a opneshift V3.9

## Notes
The template install gitea with a claim to a persistent volume (hence the name gitea-persistant). It is base on a local-storage provider, if you have something else, you need to modify this config.
As the main goal here is to provide a 'light' gitea setup ideal for internal dev ( own infra setup, test, scripts, projetcs) it make use of the internal Sqlite3 database, no requirements for MySql, PostgresSql, ...

## Setup
 
As the gitea image make use of a supervisor process (s6) to configure and launch the several services ( ss, gitea, ...), and this supervisor need to be run under root, wa cannot rely on the default mecchanics of openshift for attributing a user by we must fix it before hand.
The template configure a service account for the pod whose name is 'gitea'.  This service account must be add to the anyuid scc (Service Security Contraint) of open shit to let s6 be run as root. This can be done by the command:
```console
$ oc adm policy add-scc-to-user anyuid system:serviceaccount:$YOURPROJECT:gitea
````
replace $YOURPROJECT with the name of the project you want to deploy the template

If you haven't yet run the template, you can pre-create the service account:
````console
$ oc create serviceaccount gitea
````
In addition, create a user and group for gitea whose ids you will give when you launch the template (fields USER_UID and USER_GID):

```console
$ groupadd gitea
$ useradd -g gitea gitea
````

You'll need to pick the gid and uid in /etc/passwd

Finaly checkout the template from this repos and create it:
````console
$ oc create -f gitea-persistent.yaml 
````

, then create the application via the console:
```console
$ oc new-app gitea-persistent -p ROOT_URL=<your app/project url> -p SSH_DOMAIN=<your node domain> - p USER_UID=<userid as created before> - p USER_GID=<user groupid as created before>
````
If you need more interaction youcan create the application by selecting the template in openshift console.
 