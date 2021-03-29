---
title: ELK Stack Project
layout: page
description: Contains scripts and configuration files to automatically deplay an ELK Stack onto a vritual network
repository: https://github.com/lithrion/ELK-Stack-Project
---


In this project we created a YAML playbook that runs on an ansible control node to setup and configure an ELK Stack server.

The project contains:
- The `project1-playbook.yml` which can be run to set up an ELK stack server.
- A `roles` directory which contains addition YAML files for configuring an ELK server, a webserver, and beats to monitor the webserver.
- A `files` directory which includes additional configuration files and sets the ip addresses of the machines being configured.
- Instructions for how to run the playbook.
- A network topology diagram showing the test network that this playbook was used to configure.

The respository with the project files can be found [here]({{ page.repository }}).
