# Webfaction tools for Ansible

This is a starting point for some Ansible modules for provisioning domains, apps, and websites on Webfaction, using the [Webfaction API](http://docs.webfaction.com/xmlrpc-api/).

NOTE: THE MODULES HAVE NOT BEEN EXTENSIVELY TESTED. USE THEM WITH CAUTION! And please help improve them!


## Background

On Webfaction, to bring up a working web application, you need to start with three things:

* Create an application based on one of their templates, and give it a name. e.g. 'my_wordpress'

* Create a domain and optional subdomains. e.g. 'mywebsite.com' and 'www'

* Create a website, which connects these together. It requires a name, a domain and optional subdomains, and a list of applications and the bits of the URL space that they should serve - e.g. ('my_wordpress', '/')

Remember that Webfaction, while it is a great service, is not quite like provisioning things on blank machines or VMs.  In particular, you won't be able to use sudo, and your directory layout will be somewhat constrained.


## A simple example

To use these modules, you should put them (webfaction_app, webfaction_domain etc) in a directory called 'library' at the same level as your top-level playbooks.  See [the Ansible docs](http://docs.ansible.com/developing_modules.html) for more information.

This is what the above steps might look like in a playbook:

    ---
    - hosts: localhost

      vars:
        webfaction_user: my_webfaction_username
        webfaction_passwd: my_webfaction_password

      tasks:
      - name: Create application on Webfaction account
        webfaction_app: 
          name=testapp1
          state=present
          type=mod_wsgi35-python27
          login_name={{webfaction_user}}
          login_passwd={{webfaction_passwd}}


      - name: Create a domain
        webfaction_domain:
          name: my_domain.org
          state: present
          subdomains:
            - testsite1
          login_name: "{{webfaction_user}}"
          login_passwd: "{{webfaction_passwd}}"


      # Yes, sadly, you do need to know your webfaction hostname
      # for the following. But at least you don't need to know the
      # IP address - we'll convert that for you.

      - name: Create a website
        webfaction_site:
          name: testsite1
          state: present
          host: web431.webfaction.com 
          subdomains: 
            - 'testsite1.my_domain.org'
          site_apps:
            - ['testapp1', '/']
          https: no
          login_name: "{{webfaction_user}}"
          login_passwd: "{{webfaction_passwd}}"
    

If this all runs smoothly, you should end up with a directory called `webapps/testapp1` in your home directory on webfaction, and Webfaction will be set up, in this case, to run a WSGI app there.  See the [Webfaction docs](http://docs.webfaction.com/xmlrpc-api/apps.html#application-types) for all the different types of apps that can be configured.

Note - you can run these on a local machine, or on a webfaction host, or elsewhere, since the scripts use the remote webfaction API - the location is not important.  Running them on multiple hosts simultaneously is almost certainly a Bad Idea, though!

These come with no warranties, use at your own risk etc, but I hope someone may find them useful!

[Quentin Stafford-Fraser](http://qandr.org/quentin)

