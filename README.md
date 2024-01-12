# How to use Jupyter Notebook in VSCode on Ares?

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

**Additional Step:** If you don't have permission to execute the tunnel.sh file, grant it.
```
chmod a+rwx tunnel.sh
```
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
6. Submit the `tunnel.sh` script to the chosen partition
```
sbatch -p [your_partition] tunnel.sh
```
Wait until the job status shows as "Running" to obtain the necessary nodelist information. You'll need this value for configuring VSCode.

## Visual Studio Code configuration


1. Launch VSCode, navigate to the Remote Explorer, expand "Remotes," and check for 'Tunnels/SSH'. If not present, install the "Remote - SSH" extension from the VSCode Extensions marketplace.
2. Click on the screwdriver icon next to SSH, opening the Command Palette and select the first suggested file, typically located at `/Users/username/.ssh/config`.
3. Paste the following configuration:
```
Host ares
    HostName ares.cyfronet.pl
    User username

Host ares_worker
    HostName nodelist  # Replace with your copied nodelist (ex. ac1111)
    ProxyJump ares
    Port 22223      #Change to the desirable port
    User username  # Replace with your Ares username
    StrictHostKeyChecking no
```
ProxyJump directive, in conjunction with other parameters, facilitates a secure connection to the computing node `ares_worker` by routing the traffic through the main Ares server `ares` via the specified port and user credentials.

4. Access the Command Palette again and open `Remote-SSH Settings`.
5. In the Remote.SSH: Server Install Path section, add two paths:
   
Key1: `ares` and Value1:
```
/net/ascratch/people/username/.vscode
```
Item2: `ares_worker`, Value2: 
```
/net/ascratch/people/username/.vscode
```
This step defines the installation paths for VSCode on Ares and Ares computing nodes.

## Setting Up Jupyter Notebook in VSCode on Ares

1. While logged in to Ares, create a folder to store your project. The location depends on the project size and the number of files. Learn more about storage [here](https://docs.cyfronet.pl/display/~plgpawlik/Ares#Ares-Storage).
2. Navigate to the "Remote Explorer" in VSCode.
3. Expand the "SSH" section, and you should see `ares` and `ares_worker`.
4. Choose "Connect in New Window." It will open the Welcome page of VSCode.
5. In the Command Palette, navigate to the created folder and open the terminal.
6. Create a new virtual environment
```
python -m venv 'venv_name'
```
Replace 'venv_name' with your desired virtual environment name.

7. Activate the Virtual Environment
```
source 'venv_name'/bin/activate
```
8. Install the `ipykernel` package
```
pip install ipykernel
```
9. Make sure you have the Python extension and Jupyter extension installed in VSCode. If prompted by VSCode to install any additional packages on ares_worker, follow the suggestions and install them.
10.   In the Command Palette, choose "Create New Jupyter Notebook."
11.   Select previously created virtual environment as Kernel.

Now, you have successfully set up and run a Jupyter Notebook in VSCode on Ares.





