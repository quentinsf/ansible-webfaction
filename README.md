# Webfaction tools for Ansible

**NOTE: These modules have now been contributed to [the ansible-modules-extras repository](https://github.com/ansible/ansible-modules-extras), so should be included with Ansible 2.0 onwards.  Please submit any pull requests for code changes there. Once Ansible 2 is released, this repository may be retained only as a documentation page.**

This is a starting point for some Ansible modules for provisioning domains, apps, and websites on Webfaction, using the [Webfaction API](http://docs.webfaction.com/xmlrpc-api/).


## Background

On Webfaction, to bring up a working web application, you need to start by doing three or four things:

1. Create an **application** based on one of their templates, and give it a name. e.g. 'my_wordpress'.  This sets up the environment needed to run it in a directory within your `webapps` directory.

2. Create a **domain** and optional subdomains. e.g. 'mywebsite.com' and 'www'.  This tells the web servers to respond to these domains, once you've set your DNS service to point the domains at the correct machine.

3. Create a **website**, which connects these together. It requires a name, a domain and optional subdomains, and a list of applications and the bits of the URL space that they should serve - e.g. ('my_wordpress', '/')

4. You may also need to create a **database** if your application depends on one.  Many of the standard templates will create one for you automatically, though.

Remember that using Webfaction is not quite like provisioning things on blank machines or VMs.  In particular, you won't be able to use sudo, and your directory layout will be somewhat constrained.  Having said that, it is an amazingly flexible service for very little financial outlay, and the guys who run it know what they're doing.  But this isn't a place for a Webfaction tutorial; I'll assume you're familiar with the concepts.


## A simple example

To use these modules, which you can find in the 'library' folder (webfaction_app, webfaction_domain etc), put them in a directory called 'library' at the same level as your top-level playbooks.  See [the Ansible docs](http://docs.ansible.com/developing_modules.html) for more information.

Once you've done that, this is what the above steps might look like in a playbook:

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
          type=mod_wsgi35-python27  # See below
          login_name={{webfaction_user}}
          login_password={{webfaction_passwd}}


      - name: Create a domain
        webfaction_domain:
          name: my_domain.org
          state: present
          subdomains:
            - testsite1
          login_name: "{{webfaction_user}}"
          login_password: "{{webfaction_passwd}}"

      # This will also create a default DB user with the same
      # name as the database, and the specified password.
      
      - name: Create a database
        webfaction_db:
          name: "{{webfaction_user}}_db1"
          password: mytestsql
          type: mysql
          login_name: "{{webfaction_user}}"
          login_password: "{{webfaction_passwd}}"

      # Yes, sadly, you do need to know your webfaction hostname
      # for the 'host' parameter in the following. 
      # But at least you don't need to know the IP address - 
      # we'll convert the hostname for you.

      - name: Create a website
        webfaction_site:
          name: testsite1
          state: present
          host: web4321.webfaction.com 
          subdomains: 
            - 'testsite1.my_domain.org'
          site_apps:
            - ['testapp1', '/']
          https: no
          login_name: "{{webfaction_user}}"
          login_password: "{{webfaction_passwd}}"
    

If this all runs smoothly, you should end up with a directory called `webapps/testapp1` in your home directory on webfaction, and Webfaction will be set up, in this case, to run a WSGI app there.  See the [Webfaction docs](http://docs.webfaction.com/xmlrpc-api/apps.html#application-types) for all the different types of apps that can be configured.

You can then use other Ansible modules to put stuff in place within that directory.

## Some Notes

* You can run playbooks that use these on a local machine, or on a Webfaction host, or elsewhere, since the scripts use the remote webfaction API - the location is not important.  However, *running them on multiple hosts simultaneously is almost certainly a Bad Idea*! So if you don't specify 'localhost' as your host, you may want to add `serial: 1` to the plays.

* If you are *deleting* domains, by using `state=absent`, then note that if you specify subdomains, those particular subdomains will be deleted.  If you don't, the domain will be deleted.

These come with no warranties, use them at your own risk, etc, but I hope someone may find them useful!

[Quentin Stafford-Fraser](http://quentinsf.com)

