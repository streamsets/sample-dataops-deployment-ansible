

<!-----

Conversion notes:

* Docs to Markdown version 1.0β33
* Wed Jan 19 2022 08:56:50 GMT-0800 (PST)
* Source doc: Automation Support for Self Managed Engines
----->


# Automation Support for Self-Managed Engines


###### Revision history: Jan 18 2022 - initial draft



## Ansible Playbook to **Create Deployment and Start Engine**

#### The Sample Playbook

Is these two files: [Deployment_create_engine_start.yml](https://github.com/streamsets/sample-dataops-deployment-ansible/blob/main/deployment_create_engine_start.yml) with [advanced_config_files_tasks.yml](https://github.com/streamsets/sample-dataops-deployment-ansible/blob/main/advanced_config_files_tasks.yml)


#### Sample hostsfile

[hostsfile_controlhub](https://github.com/streamsets/sample-dataops-deployment-ansible/blob/main/hostsfile_controlhub)


#### Issues



* At the time of this writing, this playbook uses the same address as the control hub host and the running engine.
* Tested only on BSD Bash Darwin Kernel Version 20.6.0 using a MacBook 


#### Prerequisites



* Java 8, 11 or 14 installed
* Parent directories for engine installation and download directories must exist on the host where the engine(s) will run. The defaults are /tmp/streamsets.download and /tmp/streamsets.install


#### Repo structure

Files for engine configuration are in folders named` {engine-type}/{version}[/{scala-binary-version}]/advanced-config/`. This folder structure must exist to run the playbook, even if the folders are empty.

As shown in this example structure.


```
├── DC
│   └── 4.1.0
│       ├── advanced_config
│       │   ├── archive
│       │   │   ├── credential-stores.properties
│       │   │   ├── sdc-log4j.properties
│       │   │   ├── sdc-security-policy
│       │   │   └── sdc.properties
│       │   └── sdc.properties
│       └── stagelibs.yml
├── TF
│   └── 4.1.0
│       ├── 2.11
│       |   └── …
│       └── 2.12
│           ├── advanced_config
│           │   ├── credential-stores.properties
│           │   ├── transformer-log4j.properties
│           │   ├── transformer-security.policy
│           │   └── transformer.properties
│           └── stagelibs.yml
├── advanced_config_files_tasks.yml
├── deployment_create_engine_start.yml
└── hostsfile_controlhub
```



#### Required Environment Variable Settings

Some information must be supplied first for the playbook to run. Set these as environment variables.



* `CRED_TOKEN`: From the StreamSets ControlHub, under Manage => API Credentials, generate credentials and copy-paste the environment variables from “Example usage:”
* `CRED_ID`: (same)
* `STREAMSETS_SCH_URL`: path to the DataOps ControlHub (e.g. `hub.streamsets.com`)
* `ENVIRONMENT_ID`: A self-managed environment must exist. Copy the ID into this environment variables


#### Optional Environment Variable Settings

These settings have default values, so setting these is optional. The defaults are shown.



* `STREAMSETS_DOWNLOAD_DIR`: on the host where the engine will run, a path to the parent folder of a download folder for a tarball installation
* `STREAMSETS_INSTALL_DIR`: on the host where the engine will run, a path to the parent folder of an installation folder for a tarball installation
* `ENGINE_TYPE`: ‘DC’ or ‘TF’. Defaults to ‘DC’
* `ENGINE_VERSION`: defaults to ‘4.1.0’
* `INSTALL_TYPE`: ‘TARBALL’ or ‘DOCKER’. Defaults to ‘TARBALL’
* `BUILD_NUMBER`: ‘Released’  For all released versions of engines, leave this as is.
* `SCALA_BINARY_VERSION`: ‘2.11’ or ‘2.12’. Defaults to ‘2.11’


#### Run the playbook


```
ansible-playbook -v -i hostsfile_controlhub deployment_create_engine_start.yml
```



#### Explanation

This simple playbook uses `uri` tasks to communicate with the DataOps ControlHub API. It creates a Deployment, updates the deployment several times, retrieves the installation command for a self-managed engine from the created deployment and executes that command. That command is a curl command to download a shell script which in turn downloads other shell scripts and uses them to launch the engine.

Some very limited error checking is done. Run ansible-playbook with verbose option to diagnose problems.


## Project Engineering Documentation {#project-engineering-documentation}


### 
    [Requirements, Design Discussions](https://docs.google.com/document/d/1eyNNIOloKM3KLx75hqrDIvHGrmJEcdCiXTdcNE62hRI/edit#heading=h.q1h504b0bzt3) - Santhosh and Anthony {#requirements-design-discussions-santhosh-and-anthony}


### 
    [Sequence Diagram of Front end to Back end API calls](https://docs.google.com/document/d/1BKIwyNBdcZ6Dh-uCR2cVHtVt39VCciQ81xGg1tfhbTA/edit) {#sequence-diagram-of-front-end-to-back-end-api-calls}
