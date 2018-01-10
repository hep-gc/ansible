[![PyPI version](https://img.shields.io/pypi/v/ansible.svg)](https://pypi.python.org/pypi/ansible)
[![Build Status](https://api.shippable.com/projects/573f79d02a8192902e20e34b/badge?branch=devel)](https://app.shippable.com/projects/573f79d02a8192902e20e34b)


Ansible
=======

Ansible is a radically simple IT automation system.  It handles configuration-management, application deployment, cloud provisioning, ad-hoc task-execution, and multinode orchestration - including trivializing things like zero downtime rolling updates with load balancers.

Read the documentation and more at https://ansible.com/

Many users run straight from the development branch (it's generally fine to do so), but you might also wish to consume a release.

You can find instructions [here](https://docs.ansible.com/ansible/intro_installation.html) for a variety of platforms.

Changing the SSH port with Ansible?
===================================

This fork/branch of https://github.com/ansible/ansible provides a patch to test a non-standard ssh port specification and revert to the default ssh port (normally 22) if the non-standard port is not open. Essentially, Ansible uses the following logic to employ ssh:

<pre><code>
      if self.port is not None:
         ssh -p {{ self.port }} ...
      else:
         ssh ...
</code></pre>

This patch changes the logic to:

<pre><code>
      if self.port is not None and self.port is OPEN:
         ssh -p {{ self.port }} ...
      else:
         ssh ...
</code></pre>

Currently the recommended way to handle ssh port changes with an un-patched Ansible is to use wait_for/when tasks in a playbook as follows:

<pre><code>
---
# file: demo.yaml
# ansible_ssh_port is used in preference to ansible_port because it works across Ansible v1 and v2.

- hosts: all
  gather_facts: false
  vars:
    ansible_ssh_port: 2222

  pre_tasks:
    - name: Test connection to port 2222
      local_action:
        module: wait_for
        port: "{{ ansible_ssh_port }}"
        timeout: 5
      register: test_2222
      ignore_errors: true

    - name: Set ansible_ssh_port to 22 if cannot connect to 2222
      set_fact:
        ansible_ssh_port: 22
      when: test_2222.elapsed >= 5

    - setup:

  roles:
    - set_non-std_port
</code></pre>

In a multi-host environment, this method is unreliable because it tests a single host but applies the result to all hosts. In contrast, the patch tests the port on each host as the connection is required, applies the result only to the tested host and requires no specific specification in the playbook. The directory ssh_port_setting_example provides demonstration playbooks and templates for ssh port setting to exercise the patch's function:

<pre><code>
---
# file: demo.yaml
# The non-standard ssh ports are specified in the host inventory and could be different for
# each host; see "ssh_port_setting_example" directory.

- hosts: all
  roles:
    - set_non-std_port
</code></pre>

Installation of the patch
=========================

   1. Remove any current installation of ansible.
   1. Clone the Ansible fork with "git clone git@github.com:hep-gc/ansible.git".
   1. Switch to the cloned repository (ie. "cd ansible") and  choose the version to install by using the the "git checkout {{patched-\*}}" command.
   1. Install the Ansible core modules with "git submodule update --init --recursive".
   1. Install the patched ansible with "python setup.py install".

And now, back to ...

Design Principles
=================

   * Have a dead simple setup process and a minimal learning curve
   * Manage machines very quickly and in parallel
   * Avoid custom-agents and additional open ports, be agentless by leveraging the existing SSH daemon
   * Describe infrastructure in a language that is both machine and human friendly
   * Focus on security and easy auditability/review/rewriting of content
   * Manage new remote machines instantly, without bootstrapping any software
   * Allow module development in any dynamic language, not just Python
   * Be usable as non-root
   * Be the easiest IT automation system to use, ever.

Get Involved
============

   * Read [Community Information](https://docs.ansible.com/community.html) for all kinds of ways to contribute to and interact with the project, including mailing list information and how to submit bug reports and code to Ansible.
   * All code submissions are done through pull requests.  Take care to make sure no merge commits are in the submission, and use `git rebase` vs `git merge` for this reason.  If submitting a large code change (other than modules), it's probably a good idea to join ansible-devel and talk about what you would like to do or add first and to avoid duplicate efforts.  This not only helps everyone know what's going on, it also helps save time and effort if we decide some changes are needed.
   * Users list: [ansible-project](https://groups.google.com/group/ansible-project)
   * Development list: [ansible-devel](https://groups.google.com/group/ansible-devel)
   * Announcement list: [ansible-announce](https://groups.google.com/group/ansible-announce) - read only
   * irc.freenode.net: #ansible

Branch Info
===========

   * Releases are named after Led Zeppelin songs. (Releases prior to 2.0 were named after Van Halen songs.)
   * The devel branch corresponds to the release actively under development.
   * For releases 1.8 - 2.2, modules are kept in different repos, you'll want to follow [core](https://github.com/ansible/ansible-modules-core) and [extras](https://github.com/ansible/ansible-modules-extras)
   * Various release-X.Y branches exist for previous releases.
   * We'd love to have your contributions, read [Community Information](https://docs.ansible.com/community.html) for notes on how to get started.

Authors
=======

Ansible was created by [Michael DeHaan](https://github.com/mpdehaan) (michael.dehaan/gmail/com) and has contributions from over 1000 users (and growing).  Thanks everyone!

Ansible is sponsored by [Ansible, Inc](https://ansible.com)

Licence
=======
GNU
Click on the [Link](COPYING) to see the full text.


