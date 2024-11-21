+++
title = "How To Use Fancy Ansible Groups"
date = "2024-11-20"
thumbnail = "posts/fancy-ansible-groups/thumbnail.png"
tags = ["Ansible"]
type = "post"
toc = true
keywords = ["Ansible"]
description = "My new plugin for fancy Ansible groups lets you generate combined hierarchical groups and invert the relationship syntax to add parent groups to hosts easier."
showFullContent = true
+++

# Intro

I made a (https://galaxy.ansible.com/ui/repo/published/makinj/fancy_groups/)[plugin] a while back to solve the two biggest problems I had when managing complex Ansible inventories and want to share them in case others have the same gripes as me. 

# Installing
1. Run `ansible-galaxy collection install makinj.fancy_groups` or add `makinj.fancy_groups` to your `requirements.yml` file.
1. Add The following line to your `ansible.cfg` or merge it with an existing `enable_plugins` line:
  ```
  enable_plugins = auto, yaml, makinj.fancy_groups.combined_groups, makinj.fancy_groups.inverted_group
  ```
# Inverted Groups

Normally, if you want to make a new host or group and add it to a few groups, you need to use the following syntax:

```yaml
group1:
  hosts:
    host1:
group2:
  hosts:
    host1:
group3:
  hosts:
    host1
```

It felt really repetitive to me to be re-typing my host in each group and it didn't scale well when I started to have a ton of different groups that I want to apply to a new host.

With inverted groups, the same hierarchy can be expressed this way:

```yaml
---
igroup.yaml
---
plugin: makinj.fancy_groups.inverted_group
hosts:
  host1:
    parents:
    - group1
    - group2
    - group3
``` 

The `parents` option acts as a complement to the built-in `children` and `host` options and allows you to specify a group or host's parent groups directly.


# Combined Groups

I often want to use the same ansible inventory structures or variables across environments but feel encumbered by the management of those configurations in making them identical. The following syntax will generate copies of a group structure with variants that are prefeixed by either "dev" or "prod":

```yaml
---
cgroups.yaml
---
plugin: makinj.fancy_groups.combined_groups
dimensions:
- dev:
  prod:
- hypervisor:
  guest:
    children:
      headless_guest:
      gui_guest:
```

The graph for this is:

```
@all:
  |--@dev:
  |  |--@dev_guest:
  |  |  |--@dev_gui_guest:
  |  |  |--@dev_headless_guest:
  |  |--@dev_gui_guest:
  |  |--@dev_headless_guest:
  |  |--@dev_hypervisor:
  |--@guest:
  |  |--@dev_guest:
  |  |  |--@dev_gui_guest:
  |  |  |--@dev_headless_guest:
  |  |--@gui_guest:
  |  |  |--@dev_gui_guest:
  |  |  |--@prod_gui_guest:
  |  |--@headless_guest:
  |  |  |--@dev_headless_guest:
  |  |  |--@prod_headless_guest:
  |  |--@prod_guest:
  |  |  |--@prod_gui_guest:
  |  |  |--@prod_headless_guest:
  |--@hypervisor:
  |  |--@dev_hypervisor:
  |  |--@prod_hypervisor:
  |--@prod:
  |  |--@prod_guest:
  |  |  |--@prod_gui_guest:
  |  |  |--@prod_headless_guest:
  |  |--@prod_gui_guest:
  |  |--@prod_headless_guest:
  |  |--@prod_hypervisor:
  |--@ungrouped:
```

When you create a new gui guest in the prod environment, it should be as simple as adding them to the `prod_gui_guest` group, which will include them in the following groups which can all apply variables to the host's final state:

1. all
1. prod
1. guest
1. gui_guest
1. prod_guest
1. prod_gui_guest
