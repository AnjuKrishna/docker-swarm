Instructions:
Ec2 instance where the work is done
ec2-13-58-10-163.us-east-2.compute.amazonaws.com
This machine is Ansible managment host as well

ssh connection: ssh -i "thisKey.pem" ubuntu@ec2-13-58-10-163.us-east-2.compute.amazonaws.com
Please note: private key used through out the exercise is thisKey.pem

Work done :
prepared the EC2 instance launched from aws console
- chose a free tier ubuntu machine
- created a work folder called crossover, cd to same
- created the following shell scripts
  install_ansible.sh --> installs ansible
  install_docker.sh --> installs docker
- created ansible cofiguration file and inventory.ini file for ansible configuration
- other contents
   thisKey.pem --> the private key for accessing all EC2 instances
   pre_req.yml --> installs pip,aws-cli,botocore,boto3,docker-compose
   roles --> directory for ansible roles
   configure-aws-cli.yml --> playbook for configuring awscli, creating docker-swarm
   deploy.yml --> playbook to manage  manager node in docker swarm.
   roles --> roles used in this assigment
	- aws-cli-configure --> aws-cli configuration
	- docker-aws --> installs stack for docker-for-aws using cloud formation
	- deploy_swarm --> configures a swarm manager node for running the app as a service

Assumptions: 
  used docker-for-aws as security,vpc,ELB comes out of the box
  used compose v3
  used EFS and S3 for shared mountable storage
  uploaded the application image to docker hub as anjudocker/journals
  The docker-compose file downloads and uses app image from repo

Requirements not covered
  could not execute the role deploy-swarm as python is not available on EC2 
instance created with cloud formation
Need more time to analyse and fix the same
Attaching error reported by ansible
---START ----
TASK [deploy_swarm : install python 2] *****************************************
task path: /home/ubuntu/crossover/roles/deploy_swarm/tasks/main.yml:2
<ec2-52-15-233-174.us-east-2.compute.amazonaws.com> ESTABLISH SSH CONNECTION FOR USER: docker
<ec2-52-15-233-174.us-east-2.compute.amazonaws.com> SSH: EXEC ssh -C -o ControlMaster=auto -o ControlPersist=60s -o 'IdentityFile="/home/ubuntu/crossover/thisKey.pem"' -o KbdInteractiveAuthentication=no -o PreferredAuthentications=gssapi-with-mic,gssapi-keyex,hostbased,publickey -o PasswordAuthentication=no -o User=docker -o ConnectTimeout=10 -o ControlPath=/home/ubuntu/.ansible/cp/a4efcd2ca6 -tt ec2-52-15-233-174.us-east-2.compute.amazonaws.com 'sudo -H -S -n -u root /bin/sh -c '"'"'echo BECOME-SUCCESS-igjoifkndqaqbjfrxuaerqmhhhjcqqyh; test -e /usr/bin/python || (apt -y update && apt install -y python-minimal)'"'"''
<ec2-52-15-233-174.us-east-2.compute.amazonaws.com> (127, '/bin/sh: apt: not found\r\n', 'Shared connection to ec2-52-15-233-174.us-east-2.compute.amazonaws.com closed.\r\n')
fatal: [ec2-52-15-233-174.us-east-2.compute.amazonaws.com]: FAILED! => {
    "changed": true,
    "failed": true,
    "rc": 127,
    "stderr": "Shared connection to ec2-52-15-233-174.us-east-2.compute.amazonaws.com closed.\r\n",
    "stdout": "/bin/sh: apt: not found\r\n",
    "stdout_lines": [
        "/bin/sh: apt: not found"
    ]
}
        to retry, use: --limit @/home/ubuntu/crossover/deploy.retry

PLAY RECAP *********************************************************************
ec2-52-15-233-174.us-east-2.compute.amazonaws.com : ok=0    changed=0    unreachable=0    failed=1
---END-----
Issues faced: 
1)application unit test failures
Resolution:required to have sql running when building spring boot image
In docker-compose file build the image than using prebuilt one
2)created keypair with AWS CLI, it will not connect to docker manager
Finally I had to create an instance from console and use the downloaded keypair
3)missed providing parameter EnableCloudStorEfs in cloud formation template
this caused errors while bring up service on swarm. The application will not come up
as volumes cannot be created
resolution: updated the cloud formation template to use the needed parameter.
4)NON availability of docker plugin cloudstor:aws in east region.
The issue is discussed in https://github.com/docker/for-aws/issues/20
Hence I could not test application with mounted volume in test setup which
was on EC2 instance in N.Virgina
5)used a .env file to configure runtime constraints on CPU and memory per service.
.env not read by compose file 
https://github.com/docker/compose/issues/4223
Resolution : exported the configuration variables in shell scripts
Need to source the file before running deploy  
6)python not available in docker swarm manager nodes created using cloud formation
Could not run the ansible role which starts docker service
Need more time to investigate
Reslution: This step is done manually

