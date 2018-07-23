# SRE Documentation

The documentation in `/sre` is intended to be a loose collection of notes that I take down as we progress on the mythical
journey from sysadmin circa 1990 to sysadmin cira 2018. You may as well call it "devops" or "automation", but the spirit of
the venture is to integrate SRE principles into ops, because who doesn't love a good acronym loaded with the promise of magic?

## 1. Infrastructure as Code

Pretty much everyone is using Ansible / Chef / Puppet / Salt to automate ops these days. A component of this that I am
specifically interested in is unit testing infrastructure to ensure that what goes into automation is valid, and that "no 
shit" moments are caught before they're replicated at scale.

To achieve this, we're leveraging GitLab pipelines coupled with an OpenStack platform and [Molecule][1] to create testable 
roles and plays that are deployed automatically to the point of being "stage" ready, with a still manual intervention step to 
promote them to production.

For each role/play, there should be a series of tests that vet what the resource is supposed to do, and to test it to ensure 
that it works as advertised. Just trapping Ansible resources in your nearest git/svn/mercurial is not enough, and I believe 
that testable infrastructure is what differentiates true IaC from "hey, we can haz plays".

Writings on IaC are contained in my `molecule.md` file in this subfolder, as Molecule is the provisioner we're using to
create test instances, push roles/plays and invoke Testinfra to run unit tests against fresh machines on each commit.

[1]: https://molecule.readthedocs.io/en/latest/
