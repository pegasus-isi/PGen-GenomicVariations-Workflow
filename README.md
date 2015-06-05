NGS-GenomicVariations-Workflow
==============================

The workflow is controlled by several configuration files:

*inputs-ref.txt* - should contain only one URL, and that would be
for the reference genome to use.

*chromosomes.txt* - a list of chromsomes to process. This should
be the same format as the reference genome. One way to generate
this list is:

```
cat reference.fa | grep '^>' | grep -v -i scaffold
```

*inputs-fastq.txt* - a list of URLs for the fastq files to process.
Depending on the settings in conf/main.conf, the list can be either
single-end or pair-end. In either case, put one URL on each line.

*conf/main.conf* - this one contains general properties such as input
formats and filters.

*~/.ngs-workflow.conf* - Create the file with this content:

```
# local refers to the submit host. Specify paths to a directory
# which can be used by the workflow as work space, and locations
# for local software installs.
[local]

work_dir = /local-scratch/%(username)s/workflows

irods_bin = /ccg/software/irods/3.2/bin

# tacc refers to configuration for the TACC Stampede 
# supercomputer. To use this machine, you need an allocation
# (start with TG-) and you also need to know your username
# and storage group name for the system. The easiest way to 
# obtain those is to log into the system, and run:
# cds; pwd
# This should return a path like: /scratch/00384/rynge. The
# storage group is the second level, and your username is 
# last level.
[tacc]

allocation = TG-ABC1234

username = rynge

storage_group = 00384
```

## Credentials

You might need a combination of credentials based on where you are
running the workflow and where input/output data is staged from/to.

Basic files are pulled from the submit host with scp. This is to keep
the requirements on the submit host light, and make it easy to run the
submit host as a cloud instance in for example Atmosphere. A workflow
needs an ssh key:

```
mkdir -p ~/.ssh
ssh-keygen -t rsa -b 2048 -f ~/.ssh/workflow
     (just hit enter when asked for a passphrase)
cat ~/workflow.pub >>~/.ssh/authorized_keys 
```

To access data from the iPlant iRods repository, you need a file in your
home directory named ~/irods.iplant.env, with 0600 permission and
content like:

```
irodsHost data.iplantcollaborative.org
irodsPort 1247
irodsUserName YOUR_IRODS_USERNAME
irodsZone iplant
# Pegasus requirement
irodspassword 'YOUR_IRODS_PASSWORD'
```

If you want to run the workflow at TACC, you need a X509 proxy. Create
one with (this needs to be done before each workflow run):

    myproxy-logon -s myproxy.xsede.org -t 144:00 -l YOUR_XSEDE_USERNAME


## Starting the workflow

To generate the workflow for execution in HTCondor pool:

    ./workflow-generator --exec-env distributed

to run on TACC (and don't forget to renew your X509 proxy):

    ./workflow-generator --exec-env tacc-stampede


