

<!-----

Conversion notes:

* Docs to Markdown version 1.0β33
* Wed Jan 19 2022 08:56:50 GMT-0800 (PST)
* Source doc: Automation Support for Self Managed Engines
----->


# Automation Support for Self-Managed Engines


###### Revision history: Jan 18 2022 - initial draft


### Preface
Use this Ansible playbook to create a deployment and start one or more engines from that deployment. The deployment will be created in 
the StreamSets Control Hub, so you should have an account and familiarize yourself with the StreamSets basics for 
environments, deployments and engines first. It's assumed that you have the default self-managed environment available.
The engines started with this Ansible playbook will be "self-managed" engines.


## Ansible Playbook to **Create Deployment and Start Engine**

#### The Sample Playbook

These files: [Deployment_create_engine_start.yml](https://github.com/streamsets/sample-dataops-deployment-ansible/blob/main/deployment_create_engine_start.yml)
with [advanced_config_files_tasks.yml](https://github.com/streamsets/sample-dataops-deployment-ansible/blob/main/advanced_config_files_tasks.yml)
and [engines_start.yml](https://github.com/streamsets/sample-dataops-deployment-ansible/blob/main/engines_start.yml)
make up the sample playbook.


#### Sample hostsfile

A sample hostsfile, [hostsfile_controlhub](https://github.com/streamsets/sample-dataops-deployment-ansible/blob/main/hostsfile_controlhub),
is included. This is where you specify the hosts where the engines will be run.


#### Issues



* Tested only on BSD Bash Darwin Kernel Version 20.6.0 using a MacBook 


#### Prerequisites


* [Install Ansible](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html)
* Java installed on the host nodes. Check the [engine requirements](https://docs.streamsets.com/portal/platform-datacollector/latest/datacollector/UserGuide/Installation/Requirements.html#concept_vzg_n2p_kq) 
for specific java version requirements.
* Docker installed on the host nodes if doing DOCKER installations and docker daemon started
* The remote_user must have permissions to run java or docker
* For connecting to remote hosts, add appropriate credentials to your ansible.cfg
* Parent directories for engine installation and download directories must exist on the host where the engine(s) will run. The defaults are /tmp/streamsets.download and /tmp/streamsets.install


#### Repo structure

Files for engine configuration are in folders named` {engine-type}/{version}[/{scala-binary-version}]/advanced-config/`. This folder structure must exist to run the playbook, even if the folders are empty.

As shown in this example structure.


```
.
├── DC
│   └── 4.3.0
│       ├── advanced_config
│       │   ├── credential-stores.properties
│       │   ├── sdc-log4j.properties
│       │   ├── sdc-security-policy
│       │   └── sdc.properties
│       ├── stagelibs.yml
│       └── stagelibs_all.yml
├── TF
│   └── 4.2.0
│       ├── 2.11
│       │   ├── advanced_config
│       │   │   ├── credential-stores.properties
│       │   │   ├── transformer-log4j.properties
│       │   │   ├── transformer-security.policy
│       │   │   └── transformer.properties
│       │   ├── stagelibs.yml
│       │   └── stagelibs_all.yml
│       ├── 2.12
│       │   ├── advanced_config
│       │   │   ├── credential-stores.properties
│       │   │   ├── transformer-log4j.properties
│       │   │   ├── transformer-security.policy
│       │   │   └── transformer.properties
│       │   ├── stagelibs.yml
│       │   └── stagelibs_all.yml
│       └── dep.out
├── advanced_config_files_tasks.yml
├── deployment_create_engine_start.yml
├── engines_start.yml
├── environments_engine_versions_retrieve.yml
└── hostsfile_controlhub
```



#### Required Environment Variable Settings

Some information must be supplied first for the playbook to run. You can set these as environment variables or by editing the hostsfile.



* `CRED_TOKEN`: From the StreamSets ControlHub, under Manage => API Credentials, generate credentials and copy-paste the environment variables from “Example usage:”
* `CRED_ID`: (same)
* `STREAMSETS_SCH_URL`: path to the DataOps ControlHub (e.g. `cloud.login.streamsets.com`)
* `STREAMSETS_ENV_ID`: A self-managed environment must exist. Copy the ID into this environment variable. 
Find it by viewing the details of the environment on the web interface.
* `STREAMSETS_ENGINE_VERSION`: See below.
* `STREAMSETS_ENGINE_TYPE`: See below.
* `STREAMSETS_BUILD_NUMBER`: See below.
* `STREAMSETS_SCALA_BINARY_VERSION`: See below.

#### How to Specify the Engine Version with environment variables

The engine ID is made up of the engine type, the engine version, the scala binary version (for engine type Transformer) and the build number:
`{STREAMSETS_ENGINE_TYPE}:{STREAMSETS_ENGINE_VERSION}:{STREAMSETS_SCALA_BINARY_VERSION}:{STREAMSETS_BUILD_NUMBER}`

Example of a datacollector engine ID: `DC:4.3.0::Released`

Example of a transformer engine ID: `TF:4.2.0:2.12:Released`

These are the environment variables to set to specify your desired engine version:
* `STREAMSETS_ENGINE_TYPE`: ‘DC’ or ‘TF’. Defaults to ‘DC’
* `STREAMSETS_ENGINE_VERSION`: defaults to ‘4.1.0’
* `STREAMSETS_SCALA_BINARY_VERSION`: Only applies to Transformer engines. ‘2.11’ or ‘2.12’. Defaults to ‘2.11’
* `STREAMSETS_BUILD_NUMBER`: ‘Released’  For all released versions of engines, leave this as is.


#### Optional Environment Variable Settings

These settings have default values, so setting these is optional. The defaults are shown.

* `STREAMSETS_DOWNLOAD_DIR`: on the host where the engine will run, a path to the parent folder of a download folder for a tarball installation
* `STREAMSETS_INSTALL_DIR`: on the host where the engine will run, a path to the parent folder of an installation folder for a tarball installation
* `STREAMSETS_INSTALL_TYPE`: ‘TARBALL’ or ‘DOCKER’. Defaults to ‘TARBALL’
* `STREAMSETS_ENGINE_LABELS`:
* `STREAMSETS_DEPLOYMENT_TAGS`:


### hostsfile

The hostsfile must include host groups. An example hostsfile is included.
* engines<br>the host nodes where engines will be started
* controlhub<br>where you will run the ansible script commands. This can be localhost.



#### Run the playbook

* Install Java 8, 11 or 14 on the host nodes for TARBALL installations
* Install docker on the host nodes if doing DOCKER installations, and start docker daemon

```
ansible-playbook -i hostsfile_controlhub deployment_create_engine_start.yml
```

Tip: for more human readable output, put ANSIBLE_STDOUT_CALLBACK=yaml on the same line like this:


```
ANSIBLE_STDOUT_CALLBACK=yaml  ansible-playbook -i hostsfile_controlhub deployment_create_engine_start.yml
```


#### Explanation

This simple playbook uses `uri` tasks to communicate with the DataOps ControlHub API. It creates a Deployment, 
updates the deployment several times, retrieves the installation command for a self-managed engine from the created 
deployment and executes that command. That command is a curl command to download a shell script which in turn downloads 
other shell scripts and uses them to launch the engine.

Some very limited error checking is done. Run ansible-playbook with verbose option to diagnose problems.

