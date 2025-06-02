
# Galaxy + AWS ParallelCluster Integration

This repository provides configuration files, setup instructions, and scripts to deploy a scalable Galaxy bioinformatics platform on AWS using ParallelCluster with Slurm as the job scheduler.

This repo includes:
* Example ParallelCluster configuration YAML
* Galaxy job_conf.yml examples for Slurm integration
* Networking and security setup notes
* DRMAA library installation steps


## Setup Instructions for ParallelCluster Key and Networking

### 1. SSH Key Setup (`KeyName`)
Before launching the cluster, make sure you:
- Go to the **AWS EC2 console** → **Key Pairs** → create a key pair (recommended: use **ED25519** type for better security).
- Save the `.pem` or `.pem.pub` file securely — you’ll need it for `ssh` access:

```bash
ssh -i /path/to/your-key.pem ubuntu@<head-node-public-ip>
```

### 2. Security Group Setup (AdditionalSecurityGroups)

The security group (sg-**) attached to the head node must have the following inbound rules:

| Port      | Purpose                                   |
|-----------|------------------------------------------|
| 22        | SSH access to the head node              |
| 8080      | Galaxy web interface                     |
| 4079      | EFS                                      |



### 3. EFS Mounting

The EFS filesystem (fs-0428856f4d9a997bb) must:
* Have mount targets in the same VPC as your subnet.
* Allow NFS (port 2049) access between the head node and compute nodes to the EFS mount targets.

### 4. Installation of DRAMM Library

For every node, you need to install the DRMAA library to allow Galaxy to submit jobs to Slurm. This can be done by running the following commands on each node:

```bash
sudo apt-get install build-essential libslurm-dev automake bison
wget https://github.com/natefoo/slurm-drmaa/releases/download/1.1.5/slurm-drmaa-1.1.5.tar.gz
tar zxvf slurm-drmaa-1.1.5.tar.gz
cd slurm-drmaa-1.1.5
./configure --prefix="/usr/lib/slurm-drmaa"
make
sudo make install

# Then on Galaxy termination session, run:
export DRMAA_LIBRARY_PATH=/usr/lib/slurm-drmaa/lib/libdrmaa.so.1
```

### 5. Galaxy Installation

```bash
# Clone the Galaxy repository (using the latest stable release branch)
git clone -b release_24.0 https://github.com/galaxyproject/galaxy.git

cd galaxy

# Copy and edit the main Galaxy configuration file
cp config/galaxy.yml.sample config/galaxy.yml
# → Edit config/galaxy.yml as needed (refer to the galaxy.yml example in this repo)

# Copy and edit the Galaxy job configuration for Slurm
cp config/job_conf.xml.sample config/job_conf.xml
# → Edit config/job_conf.xml to configure the Slurm job runner (see job_conf.yml in this repo)

# Start Galaxy
./run.sh
```

Access Galaxy
* By default, Galaxy runs on port 8080.
* Open your browser and navigate to:


```bash
http://<head-node-public-ip>:8080

# If direct access fails (due to firewall or networking), use SSH port forwarding:

ssh -i /path/to/your-key.pem -L 8080:localhost:8080 ubuntu@<head-node-public-ip>
```



