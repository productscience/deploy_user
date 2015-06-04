Deploy User
========

This role creates a deploy user, to ease deployment when using with deploy keys on services like Github, or Bitbucket. I created this after initially finding the deployment process with slightly fiddly, and wanting to avoid messing around with `sudo` and ssh environment variables just to deploy an web app.

This is written with the assumption it is being used on Debian, or Debian based operating systems, like Ubuntu.

Requirements
------------

None

Role Variables
--------------

The following variables are set in defaults.yml for the role:

```yaml
deploy_user_name: deploy
deploy_user_ssh_priv_key_path: lookup('file', '~/.ssh/id_rsa')
deploy_groups: webapp, www-data
```

Explaining these roles:

- **deploy_user_name** is the name of the deploy to create. By default they have a home directory
- **deploy_user_ssh_priv_key_path** is the path to the private-key for the deploy user. The corresponding public key would be entered into the github project at https://github.com/{username}/{project}/settings/keys.
- **deploy_groups**: typically the same group as a web server, or developers, to allow them to ssh in to troubleshoot if necessary.


Dependencies
------------

None

Example Playbook
-------------------------

You would typically use this before using the Ansible `git` module, to put the deploy user's private key onto the server, so it can be used during automated deployments. Assuming we had defined a `app_name` and `git_branch` defined in our playbook vars, we might use it in the playbook might like so:

```yaml

- hosts: webapps
  roles:
     - productscience.deploy_user

- tasks:

  - name: fetch code from github repo, using the deploy key
    git: repo=git@github.com:productscience/some_project.git
         dest=/var/www/{{ app_name }}
         accept_hostkey=True
         version={{ git_branch }}
         key_file=/home/{{ deploy_user_name }}/.ssh/deploy

  - name: gracefully reload app server with new code
    service: name={{appname}} state=reloaded

```

This would ssh into github using the deploy user's private key, and checkout the desired git branch into `/var/www/{{ app_name }}`, before the reloading the server to serve the latest version of the application to users.


License
-------

MIT

Author Information
------------------

This is part of a set of open sourced roles used by Product Science, for deploying and managing web applications. For more, see productscience.co.uk, or email hello@productscience.co.uk
