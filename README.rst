===============================
kuryr-libnetwork
===============================

.. image:: https://raw.githubusercontent.com/openstack/kuryr/master/doc/images/kuryr_logo.png
    :alt: Kuryr mascot
    :align: center


Docker for OpenStack Neutron

Kuryr-libnetwork is `Kuryr's <https://github.com/openstack/kuryr>`_ Docker
libnetwork driver that uses Neutron to provide networking services. It provides
containerised images for the common Neutron plugins.

This repo provides libnetwork specific functionalities such as handler methods
for libnetwork apis. All the logic/utilities that can be shared among
different container networking frameworks such as Docker's libnetwork,
K8s's cni and so on, is maintained in seperate Kuryr repo as a common library.


* Free software: Apache license
* Documentation: http://docs.openstack.org/developer/kuryr
* Source: http://git.openstack.org/cgit/openstack/kuryr
* Bugs: http://bugs.launchpad.net/kuryr

Features
--------

* TODO


Prerequisites
-------------

::

    $ sudo pip install -r requirements.txt

Installing Kuryr's libnetwork driver
------------------------------------

Running the following will grab the requirements and install kuryr

::

    $ sudo pip install .


Configuring Kuryr
-----------------

Generate sample config, `etc/kuryr.conf.sample`, running the following

::

    $ tox -e genconfig


Rename and copy config file at required path

::

    $ cp etc/kuryr.conf.sample /etc/kuryr/kuryr.conf


Edit keystone section in `/etc/kuryr/kuryr.conf`, replace ADMIN_PASSWORD:

::

    auth_uri = http://127.0.0.1:35357/v2.0
    admin_user = admin
    admin_tenant_name = service
    admin_password = ADMIN_PASSWORD


In the same file uncomment the `bindir` parameter with the path for the Kuryr
vif binding executables:

::

    bindir = /usr/local/libexec/kuryr


Running Kuryr
-------------

Currently, Kuryr utilizes a bash script to start the service. Make sure that
you have installed `tox` before the execution of the below command.

::

    $ sudo ./scripts/run_kuryr.sh

After the booting, please restart your Docker service, e.g.,

::

    $ sudo service docker restart

The bash script creates the following file if it is missing.

* ``/usr/lib/docker/plugins/kuryr/kuryr.json``: Json spec file for libnetwork.

Note the root privilege is required for creating and deleting the veth pairs
with `pyroute2 <http://docs.pyroute2.org/>`_ to run.

Testing Kuryr
-------------

For a quick check that Kuryr is working create a network:

::

    $ docker network create --driver kuryr test_net
    785f8c1b5ae480c4ebcb54c1c48ab875754e4680d915b270279e4f6a1aa52283
    $ docker network ls
    NETWORK ID          NAME                DRIVER
    785f8c1b5ae4        test_net            kuryr

To test it with tox:

::

    $ tox

You can also run specific test cases using the ``-e`` flag, e.g., to only run
the *fullstack* test case.

::

    $ tox -e fullstack

Generating Documentation
------------------------


We use `Sphinx <https://pypi.python.org/pypi/Sphinx>`_ to maintain the
documentation. You can install Sphinx using pip.

::

    $ pip install -U Sphinx

In addition to Sphinx you will also need the following requirements
(not covered by `requirements.txt`)::

    $ pip install oslosphinx reno 'reno[sphinx]'

The source code of the documentation are under *doc*, you can generate the
html files using the following command. If the generation succeeds,a
*build/html* dir will be created under *doc*.

::

    $ cd doc
    $ make html

Now you can serve the documentation at http://localhost:8080 as a simple
website.

::

    $ cd build/html
    $ python -m SimpleHTTPServer 8080

Limitations
-----------

To create Docker networks with subnets having same/overlapping cidr, it is
expected to pass unique pool name for each such network creation Docker
command. Docker cli options -o and --ipam-opt should be used to pass pool
names as shown below:

::

    $ sudo docker network create --driver=kuryr --ipam-driver=kuryr \
      --subnet 10.0.0.0/16 --ip-range 10.0.0.0/24 \
      -o neutron.pool.name=neutron_pool1 \
      --ipam-opt=neutron.pool.name=neutron_pool1 \
      foo
      eddb51ebca09339cb17aaec05e48ffe60659ced6f3fc41b020b0eb506d364

Now Docker user creates another network with same cidr as the previous one,
i.e 10.0.0.0/16, but with different pool name, neutron_pool2:

::

    $ sudo docker network create --driver=kuryr --ipam-driver=kuryr \
      --subnet 10.0.0.0/16 --ip-range 10.0.0.0/24 \
      -o neutron.pool.name=neutron_pool2 \
      --ipam-opt=neutron.pool.name=neutron_pool2 \
      bar
      397badb51ebca09339cb17aaec05e48ffe60659ced6f3fc41b020b0eb506d786


External Resources
------------------

The latest and most in-depth documentation is available at:
    <https://github.com/openstack/kuryr/tree/master/doc/source>

