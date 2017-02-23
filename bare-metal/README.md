# Bare metal install guide
PNDA can be installed using the AWS CLI onto pre-existing machines or 'bare-metal'.

1. Obtain a set of machines on which to install PNDA. The requirements are:
   -  5 machines for pico. See [the pnda guide](https://github.com/pndaproject/pnda-guide/blob/develop/provisioning/aws/PREPARE.md#required-resources) for suggested CPU and memory requirements for each machine.
   - 17 machines for standard.
   - On every machine, a log volume of 10GB or more at /dev/xvdc.
   - On the data node, one to three volumes of 35GB or more at /dev/xvdd /dev/xvde /dev/xvdf for data.
   - If your devices are not named xvd{c|d|e|f} then edit [this file](https://github.com/pndaproject/pnda-aws-templates/blob/bare-metal/bootstrap-scripts/base.sh) so the disk volumes you have are mounted at the right places.
2. Install either Ubuntu 14.04 or Redhat Enterprise Linux 7 on all machines
3. If not already in place, setup ssh key based access to the machines.
   - create an RSA keypair to use to access the machines:

        ```sh
        ssh-keygen -b 2048 -t rsa -f mykey.pem -q -N ""
        ```
   - create a sudo user `<username>` on each machine:

        ```sh
        sudo su
        adduser <username> --disabled-password --gecos ""
        echo <username> ALL=NOPASSWD: ALL >> /etc/sudoers
        ```
   - allow login for that user with the created key on each machine:

        ```sh
        mkdir /home/<username>/.ssh
        cat mykey.pem.pub >> /home/<username>/.ssh/authorized_keys
        ```
4. Copy `pnda_env_example.yaml` to create `pnda_env.yaml`
5. Edit `pnda_env.yaml` with `ec2_access: OS_USER: <username>`
6. Edit `pnda_env.yaml` with `pnda_application_repo: PR_FS_TYPE: fs`
7. (Optional) Edit `pnda_env.yaml` with `ntp: NTP_SERVERS: my.ntp.server` if internet based NTP servers are not reachable from your environment.
8. (Optional) Fork the `platform-salt` repository or clone it locally to where the CLI is run from and do a search/replace for `eth0` if your network interfaces are not named `eth0`. Then Edit `pnda_env.yaml` to point `platform_salt: PLATFORM_GIT_REPO_URI` at the fork, or `platform_salt: PLATFORM_SALT_LOCAL` if using a clone of `platform-salt` local to where the CLI is run.
9. Create a [pnda mirror](https://github.com/pndaproject/pnda/tree/develop/mirror) and [build the PNDA binaries](https://github.com/pndaproject/pnda/tree/develop/build) and edit `pnda_env.yaml` with the address of it: `mirrors: PNDA_MIRROR: http://mirror_address`. Please use a clean internet connected machine to build the mirror and the PNDA binaries otherwise the dependency calculations can be incorrect and the mirror will be missing packages, or will fail entirely.
10. Edit bare-metal/pico.json or bare-metal/standard.json (or copies of these files) with the IP addresses of these machines. `ip_address` must be reachable from each machine but does not have to be reachable from where the cli is run, `public_ip_address` (set for the bastion node only) must be reachable from where the cli is run. This allows the bastion to act as a gateway into a private network. Add or remove datanodes, kafka nodes, zookeeper nodes (standard only), opentsdb nodes (standard only) to match the number of machines that you have.
11. Ensure all ports are open between the machines and port 22 is open externally on the bastion machine
12. Install pip packages required by the CLI

    ```sh
    # make sure to use pip2 or the install will fail later on
    cd cli
    sudo pip2 install -r requirements.txt
    ```
13. Run pnda-cli to install PNDA with the -m flag for bare-metal installation.

    ```sh
    cli/pnda-cli.py -e pnda -s mykey -f pico -m 'bare-metal/my-pico.json' create
    cli/pnda-cli.py -e pnda -s mykey -f standard -m 'bare-metal/my-standard.json' create
    ```
14. Run pnda-cli to add new nodes to PNDA:

    ```sh
    cli/pnda-cli.py -e pnda -s mykey -f pico -m 'bare-metal/my-pico.json' expand
    ```