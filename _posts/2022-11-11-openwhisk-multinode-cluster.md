---
title: Deploying OpenWhisk on a Multi-Node Cluster Using Ansible
# sub-title: Model Serving
layout: post
tags: [serverless, ansible, openwhisk]
mathjax: true
cover-img: /static/img/linux-vm-cover.png
social-share: false
readtime: true
---

**Post Goal:**

This blog post summarizes how to deploy OpenWhisk on a multi-node cluster using Ansible. Surprisingly, [OpenWhisk's Github](https://github.com/apache/openwhisk/tree/master/docs) does not provide any kind of documentation on how to do this (they do have documentation on distributed deploying using Kubernetes in their defense). After deadling with all sorts of issues the last week, I finally managed to use Ansible and deploy OpenWhisk on a 4 node cluster. This post assumes you have knowledge of what OpenWhisk and serverless computing is. 

**What is Ansible?**

Ansible is an open source DevOps tool that serverless operators can use to deploy OpenWhisk on a cluster. Ansible uses "playbooks" to automate deployment jobs. Playbooks are nothing more than simply YAML files. I recommend you take a look at [this tutorial](https://www.cloudbees.com/blog/yaml-tutorial-everything-you-need-get-started) if you are unfamiliar with YAML. The advantage of using YAML is that it is easy to read, write, and debug as it's meant to be in a human readable format. 

Ansible is completely agentless. This means Ansible communicates with all nodes through `ssh` by default. This is another useful design decision, as this means if nodes can be access between one another via `ssh` without the need for a password, Ansible will be able to easily connect these nodes. 

OpenWhisk already contains the playbooks required for deployment. These playbooks contain configuration for different components of the OpenWhisk architecture. The only thing you as an OpenWhisk operator needs to do is provide what Ansible calls an "inventory file". The inventory file is simply a `txt` file where you can group the hosts and control the actions of specific group.s 

