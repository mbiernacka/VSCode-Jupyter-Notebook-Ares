# How to use Jupyer notebooks in VSCode on Ares?

**Introduction to Using VSCode Notebook on Ares**

In this guide, we'll explore how to leverage Jupyter Notebooks within Visual Studio Code (VSCode) on the Ares supercomputer. Due to the internal nature of Ares computations, direct exposure of the Jupyter Notebook's web socket to the external world isn't straightforward. Therefore, we'll employ the concept of SSH tunneling to establish a secure connection between our local machine and the Ares computing nodes.

**SSH Tunneling Concept:**

SSH tunneling, also known as port forwarding, is a technique that allows secure communication between a local machine and a remote server by encapsulating the traffic within the secure SSH protocol. In the context of Ares, we utilize SSH tunneling to access the Jupyter Notebook running on a computing node.

In the upcoming steps, we'll detail the process of setting up the SSH tunnel, connecting to the Jupyter Notebook on Ares, and seamlessly working with the notebook content using Visual Studio Code. 
## (Optional) Generating and Adding SSH Keys for Ares

### Add public key (to avoid entering the password every time)

1. Navigate to your home directory:

 ```bash
 cd ~
 ```

2. Check if the .ssh directory exists:

 ```
 ls -a | grep .ssh
 ```
3. Generate an ed25519 key pair (public and private keys):
  ```
  ssh-keygen -t ed25519
```
4. Set the file name and optional passphrase (press Enter for default in all 3 steps):

```
Enter file in which to save the key (/home/user/.ssh/id_ed25519):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```
5. Navigate to the .ssh directory:
```
cd ~/.ssh
```
6. Copy the content of the `id_ed25519` file (private key).
7. Log in to the Ares supercomputer.
```
ssh username@ares.cyfronet.pl
```
9. Open the `ssh/authorized_keys` file in a text editor on Ares
```
nano ~/.ssh/authorized_keys
```
10. Paste the copied public key to the file and save it.
11. Logout from Ares and try loggin in to it again. After performing these steps, you should no longer be prompted for a password when connecting to Ares, as the SSH keys will be used for authentication.

---
## Setting Up SSH Tunnel and Environment on Ares

1. SSH Tunnel Script (`tunnel.sh`)

Create a script for establishing an SSH tunnel on Ares. The script, named `tunnel.sh`, is designed to be submitted as a job to the SLURM job scheduler.

```bash
#!/bin/bash

#SBATCH --job-name="tunnel"
#SBATCH --time=1:59:00

module load gcc/11.3.0
module load python/3.11.0-gcccore-11.3.0

/usr/sbin/sshd -D -p 22223 -f /dev/null -h ${HOME}/.ssh/id_ed25519
```
**Note:**
Customize the script and module loading according to your specific requirements.

The script initiates an SSH tunnel by running the SSH daemon (sshd) in the background. This tunnel listens on port 22223 and uses a configuration file located at /dev/null while utilizing the specified ed25519 private key.

2. Modify `.bashrc` configuration
```
nano .bashrc
```
3. Find the line containing "export package" and add `ml python` to it. Save the file.
```
# export SYSTEMD_PAGER=
ml python
```
This modification ensures that the Python module is loaded each time a new terminal session is initiated. 
4. Reload the updated configuration from .bashrc:
```
source .bashrc
```
5. Check the installed Python version to ensure it matches the version specified in the SSH tunnel script.
```
python --version
```


