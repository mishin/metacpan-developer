# MetaCPAN Developer

This is a virtual machine for the use of MetaCPAN contributors.  We do not recommend installing manually, but if you want to try it there are some instructions in the [puppet repository](https://github.com/CPAN-API/metacpan-puppet).

For information on using MetaCPAN, see [the api docs](https://github.com/CPAN-API/cpan-api/blob/master/docs/API-docs.md).

## Requirements

    - [Vagrant](http://www.vagrantup.com/downloads.html) (1.2.2 or later)
    - [VirtualBox](https://www.virtualbox.org/), we recommend [4.3.10](https://www.virtualbox.org/wiki/Download_Old_Builds), see [guest additions](http://stackoverflow.com/questions/22717428/vagrant-error-failed-to-mount-folders-in-linux-guest) if you get mounting issues
    - [A git client](http://git-scm.com/downloads)
    - An ssh client if not built in, [Windows users see
      this](http://docs-v1.vagrantup.com/v1/docs/getting-started/ssh.html).
    - To be able to download about 900MB of data on the first run

## Initial Setup

    ```bash
    git clone git://github.com/CPAN-API/metacpan-developer.git
    cd metacpan-developer
    sh bin/init.sh # clone all of the metacpan repos
    vagrant up # start the VM - will download the base box (900M) on the first run
    vagrant provision # necessary installation and configuration
    ```

If you get this error when provisioning "err: Could not request certificate: Connection refused - connect(2)"
then add the following to your /etc/resolv.conf as the first nameserver:

    ```bash
    nameserver 8.8.8.8
    ```

So, your /etc/resolv.conf should look something like

    ```bash
    domain home
    search home
    nameserver 8.8.8.8
    nameserver 10.0.2.3
    ```

- Connect to the vm

    ```bash
    vagrant ssh
    ```

- To edit and test

    Your workflow from this point will be to edit the MetaCPAN repositories
    which you have just checkout out to your local machine.  After you have
    made your changes, you can ssh to your box and restart the relevant
    services.  You will not develop on the VM directly.

    To install any missing (newly required) perl modules, as root run

    ```bash
     cd /home/vagrant
     ./bin/<path shown below>
    ```
    It's good practice to add the new modules to the cpanfile in the respective repository. `vagrant provision` will install modules from cpanfile for you.

    - metacpan-web is the web front end
        - /home/vagrant/bin/metacpan-web-carton
        - mounted as /home/vagrant/metacpan-web
        - sudo service starman_metacpan-web restart
    - cpan-api is the backend that talks to the elasticsearch
        - /home/vagrant/metacpan-api-carton
        - mounted as /home/vagrant/metacpan-api
        - sudo service starman_metacpan-api restart
    - metacpan-puppet is the sysadmin/server setup
        - mounted as /etc/puppet
        - /etc/puppet/run.sh

    Make changes in your checked out 'metacpan' repos and restart the service

- To debug the changes in the code.

    - For metacpan-web
        - sudo service starman_metacpan-web restart
    - For cpan-api
        - sudo service starman_metacpan-api restart

- To connect the web front-end to your local cpan-api backend.

    At times you may have to make a few changes to the backend and reflect the same on the front end.
    For Example: Setting up a new API endpoint.
    metacpan-web by default uses the hosted api i.e https://api.metacpan.org as its backend.
    To configure your local cpan-api, do the following.

    - In the metacpan-web repository,
    - Copy and Paste the `metacpan_web.conf` file and rename it as `metacpan_web_local.conf` that will contain:

    ```
    api                 http://127.0.0.1:5000
    api_external        http://127.0.0.1:5000
    api_secure          http://127.0.0.1:5000
    api_external_secure http://127.0.0.1:5000
    ```

    - This local configuration file will be loaded on top of the existing config file.
    - Do a vagrant reload after this or simply follow the debug steps that will reload this file.

- To connect to services on the VM

    WEB: [http://localhost:5001/](http://localhost:5001/)

    API: [http://localhost:5000/](http://localhost:5000/)

    SSH: `vagrant ssh`

- Running the metacpan-web test suite

    You'll want to run the suite at least once before getting started to make sure the VM has a clean bill of health.

    ```bash
     vagrant ssh
     cd /home/vagrant/metacpan-web
     ./bin/prove t
    ```

This recursively runs all the tests in the `t` directory in metacpan-web.  To do a partial run during development, specify the path to the file or directory containing the tests you want to run, for example:

    ```bash
    ./bin/prove t/model/release.t
    ```

This will save time during development, but of course you should always run the full test suite before submitting a pull request.

### More documentation

 * [SETTING UP / TESTING THE API](README_API.md) page
 * [HELP](HELP.md) page (including VM tuning notes)

### Problems?
 * Check the [FAQs](FAQs.md) page for common issues faced during the development process.
 * If you have trouble mounting the folders, check this fix for [guest additions](http://stackoverflow.com/questions/22717428/vagrant-error-failed-to-mount-folders-in-linux-guest)
 * Ask on #metacpan (irc.perl.org), or open an [issue](https://github.com/CPAN-API/metacpan-developer/issues)
