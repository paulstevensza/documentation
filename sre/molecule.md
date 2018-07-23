# Molecule

[Click here for Molecule's docs.][1].

## Blurb (from their docs)

Molecule is designed to aid in the development and testing of Ansible roles. Molecule provides support for testing with 
multiple instances, operating systems and distributions, virtualization providers, test frameworks and testing scenarios. 
Molecule is opinionated in order to encourage an approach that results in consistently developed roles that are well-written, 
easily understood and maintained.

## Using pipelines to execute tests

To make tests run off of git commits, we're using GitLab's pipelines and environments to force the creation of fresh instances
on OpenStack for each commit. This means having a bunch of `ansible-*.domain.tld` boxes divided into environments, each of
which has a dedicated GitLab multirunner installed to carry out the actual work of:

* Creating a new instance.
* Deploying the role.
* Testing the role.
* Terminating the instance.
* Profiting.

In order to make this scenario work, we need to ensure that Shade, which executes via a runner, can see the environment
variables that it needs in order to interact.

## Using the OpenStack driver

We use OpenStack to power our Molecule tests, because PID0 inside of a Docker container is a pretty useless way of testing 
roles that are intended to build virtual machines, especially when multiple roles go into deploying a complete VM.

Working with the `openstack` driver isn't the most intutive thing on the planet however, especially when you're running within 
the context of a GitLab pipeline, so here are some pointers.

### Shade

By default, the Molecule OpenStack driver uses [Shade][2] to interact with the OpenStack API's that permit the driver to do 
its thing. If you've even had abstract thoughts about OpenStack, you'll be aware that anything that touches the API's relies
on an `openrc` file. That `openrc` file is sourced, the service catalogue is loaded and the command line util / SDK can call
the necessary endpoints to interact with OpenStack.

If Shade can't find the environment variables that it expects from `openrc`, it'll fail with an error on `auth_url` that
kills the pipeline and fails the test, pretty much at the create step of the process.

### Fixing env vars to make Shade work.

There are two likely ways to convince your runner to import variables that the `gitlab_runner` service can expose to Shade to 
allow it to pick up the needed env vars. Note that when I talk about env vars here, I'm talking about the things that you
`export` from your shell, *not* the `$CI_*` variables that GitLab uses to pass information onto a runner.

The first way is to drop a services file at `/etc/systemd/system/gitlab_runner.d/openrc.conf` which contains the following:

```bash
[Service]
Environment="NOVA_ENDPOINT_TYPE=internalURL"
Environment="OS_ENDPOINT_TYPE=internalURL"
Environment="OS_INTERFACE=internalURL"
Environment="OS_USERNAME=admin"
Environment="OS_PASSWORD='supaseekrit123'"
Environment="OS_PROJECT_NAME=admin"
Environment="OS_TENANT_NAME=admin"
Environment="OS_AUTH_URL=http://1.2.3.4:5000/v3"
Environment="OS_NO_CACHE=1"
Environment="OS_USER_DOMAIN_NAME=Default"
Environment="OS_PROJECT_DOMAIN_NAME=Default"
Environment="OS_REGION_NAME=RegionOne"
Environment="OS_IDENTITY_API_VERSION=3"
Environment="OS_AUTH_VERSION=3"
```

That's pretty much a default `openrc` file transposed into a `systemd` services file.

Once you have that in place, you need to do the following:

```bash
$ systemctl daemon-reload
$ systemctl restart gitlab-runner
```

To confirm that you have the variables that you need, run the following command and observe the output (indentation my own):

```bash
$ systemctl show --property=Environment gitlab-runner

Environment=NOVA_ENDPOINT_TYPE=internalURL \
OS_ENDPOINT_TYPE=internalURL \
OS_INTERFACE=internalURL \
OS_USERNAME=admin \
OS_PASSWORD='supaseekrit123' \
OS_PROJECT_NAME=admin \
OS_TENANT_NAME=admin \
OS_AUTH_URL=http://1.2.3.4:5000/v3 \
OS_NO_CACHE=1 \
OS_USER_DOMAIN_NAME=Default \
OS_PROJECT_DOMAIN_NAME=Default \
OS_REGION_NAME=RegionOne \
OS_IDENTITY_API_VERSION=3 \
OS_AUTH_VERSION=3
```

Your pipeline should now be able to invoke `molecule` which can `import shade` to create a VM to pass the role/play being
tested onto.

The **second** method might be to update an `environment` key under the `[[runners]]` section in your
`/etc/gitlab-runner/config.toml` file. I haven't tested this yet, as I prefer the systemd services file, but I'll update this 
section as soon as I've tried this.

[1]: https://molecule.readthedocs.io/en/latest/
[2]: https://docs.openstack.org/shade/latest/
