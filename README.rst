======
Xanadu
======


**a collection of Ansible playbooks to set-up basic but fully functional FreeBSD hosts and an ecosystem of service jails**

**This is a beta version at which I am throwing a couple of hours daily at best, tending to priotary needs on my side first. I have a very long road map to complete and I would advise anyone interested to wait until I have completed some sort of decent documentation and testing infrastructure.**

Free software: BSD license

    .. | PyPi: https://pypi.python.org/pypi/pybsd

    | Github: https://github.com/rebost/xanadu

    .. | Read the Docs: http://pybsd.readthedocs.org/


`USEFUL FREEBSD COMMANDS <https://github.com/rebost/xanadu/blob/master/FREEBSD_COMMANDS.rst>`_

`BIBLIOGRAPHY <https://github.com/rebost/xanadu/blob/master/BIBLIOGRAPHY.rst>`_


Installation
============

Clone the repo: ::

   git clone git@github.com:rebost/xanadu.git && cd xanadu

*or* ::

    git clone https://github.com/rebost/xanadu.git && cd xanadu

Create a virtual environment (presupposes `virtualenvwrapper <http://virtualenvwrapper.readthedocs.org/>`_): ::

    mkvirtualenv --python=/usr/local/bin/python2 xanadu

Install/update requirements: ::

    pip install --upgrade -r requirements.txt

Retrieve dependencies from Ansible Galaxy: ::

    ansible-galaxy install --server https://galaxy-qa.ansible.com -r requirements.yml

Deploy variables safely and keep sensitive values version-controlled in a separate repository:

[*see* `examples/ansible_variables <https://github.com/rebost/xanadu/tree/master/examples/ansible_variables>`_ *for an example of variables layout*] ::

    cp -Rp examples/ansible_variables ../


Edit the ansible_variables files to reflect your hosts and use case



Deploy on Digital Ocean
=======================

Store api-related info in environment variables: ::

    export DO_API_VERSION='2'
    export DO_API_TOKEN='<YOUR_API_TOKEN_HERE>'

Display Digital Ocean account details (useful in particular to retrieve ssh keys hashes): ::

    ./do_account.sh

Display Digital Ocean options: ::

    ./do_options.sh



Apply playbooks to hosts and jails
==================================

Create droplet01:

[*until this* `merge request <https://github.com/ansible/ansible-modules-core/pull/2835>`_ *is accepted,
comment the ipv6 parameter in* `playbooks/roles/do_new_droplet/tasks/main.yml, l.14 <https://github.com/rebost/xanadu/tree/master/playbooks/roles/do_new_droplet/tasks/main.yml#L14>`_] ::

    ssh-keygen -f ~/.ssh/known_hosts -R droplet01.example.net
    ansible-playbook --extra-vars "hostname=droplet01.example.net" playbooks/create_droplet.yml

Add droplet01.example.net to your inventory file: ::

   echo droplet01.example.net >> <path_to_your_hosts_file_dir>hosts

You can now access droplet01.example.net with: ::

    ssh -Ap 123 root@droplet01.example.net

**Agent forwarding should be enabled with caution** (`man ssh_config <https://www.freebsd.org/cgi/man.cgi?query=ssh_config&sektion=5&n=1>`_)

Run site-wide play: ::

    ansible-playbook playbooks/site.yml

This will take care of first-class citizen hosts then install any as-of-yet-uninstalled jail and then run the global playbook on them



Common usage
============

All tasks are tagged for ease of use: ::

    ansible-playbook --tags="pf,openntpd" playbooks/site.yml

You can dry-run a diff'ed playbook limited to a specific host: ::

    ansible-playbook --check --diff --limit droplet01.example.net playbooks/site.yml

If you add the following config to your .ssh/config... ::

    cat << EOF >> ~/.ssh/config
    Host droplet01
      HostName droplet01.example.net
      Port 123
      User root
      ForwardAgent yes

    Host mail02
      HostName mail02.example.net
      Port 4001
      User root
      ForwardAgent yes
    EOF

    chmod 600 ~/.ssh/config

... you can simplify ssh access: ::

    ssh droplet01
    ssh mail02

**Agent forwarding should be enabled with caution** (`man ssh_config <https://www.freebsd.org/cgi/man.cgi?query=ssh_config&sektion=5&n=1>`_)

Get a human-readable representation of the dynamic inventory: ::

    ./hosts/site.py -d



Create and deploy a letsencrypt cert
====================================

in the letsencrypt jail

    export DOMAINS="-d poudriere01.docbase.net"
    export DIR=/tmp/letsencrypt-auto
    mkdir -p $DIR
    letsencrypt certonly -v --server https://acme-v01.api.letsencrypt.org/directory \
    -a standalone --standalone-supported-challenges http-01 \
    --email xxx@yyyy.net --agree-tos --non-interactive \
    --expand \
    --force-renewal --rsa-key-size 4096 --webroot-path=$DIR $DOMAINS


on the host system

    export JAIL=poudriere01
    export RPROXY=rproxy01
    export LETSENCRYPT=letsencrypt01
    export LSDIR=poudriere01.docbase.net
    mkdir -p /usr/jails/${RPROXY}/etc/ssl/${JAIL}/{certs,private}
    chmod -R 750 /usr/jails/${RPROXY}/etc/ssl/${JAIL}
    cp -fpH /usr/jails/${LETSENCRYPT}/usr/local/etc/letsencrypt/live/${LSDIR}/{cert.pem,chain.pem,fullchain.pem} /usr/jails/${RPROXY}/etc/ssl/${JAIL}/certs/
    cp -fpH /usr/jails/${LETSENCRYPT}/usr/local/etc/letsencrypt/live/${LSDIR}/privkey.pem /usr/jails/${RPROXY}/etc/ssl/${JAIL}/private/
    chmod -R 640 /usr/jails/${RPROXY}/etc/ssl/${JAIL}/{certs,private}/*
    chown -R 0:80 /usr/jails/${RPROXY}/etc/ssl/${JAIL}
    chown 0:80 /usr/jails/${RPROXY}/etc/ssl/certs/dhparam.pem
    chmod 640 /usr/jails/${RPROXY}/etc/ssl/certs/dhparam.pem

