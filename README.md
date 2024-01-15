# How to use Jupyter Notebook in VSCode on Ares?

**Introduction to Using VSCode Notebook on Ares**

**Note:**
The general approach for using Jupyter Notebook in VSCode on the supercomputer presented in this tutorial is primarily designed for Unix/Linux-based operating systems. For users on Windows, some adjustments are necessary due to differences in command interpretation, package management, and file path handling.

In this guide, we'll show how to use Jupyter Notebooks within [Visual Studio Code on the Ares supercomputer (SLURM)](https://e-dorigatti.github.io/development/2023/12/05/vscode-slurm.html). Worker nodes are located in the internal network of Ares supercomputer and cannot easily expose services on some port at given public IPs. We cannot simply start Jupyter Notebook there and access it directly. So, we'll use SSH tunneling to create a secure link between our computer and the Ares computing nodes.

In the following steps, we'll show the process of setting up the SLURM job scheduler, creating an SSH tunnel, connecting to the Jupyter Notebook on Ares, and seamlessly working with the notebook content using Visual Studio Code.

## Generating and Adding SSH Keys for Ares

SSH, or Secure Shell, is a cryptographic network protocol providing secure remote access to computers. While password authentication is an option, using SSH keys is highly recommended for enhanced security. SSH key pairs consist of a public key (placed on the server) and a private key (retained locally). Access is granted when these keys match, eliminating the need for passwords. 

**Note:** Generating a new SSH key using ssh-keygen with the same name as an existing key will overwrite the existing key.
### Add public key

1. Navigate to your home directory:

 ```bash
 cd ~
 ```

2. Check if the .ssh directory exists:

 ```
 ls -a | grep .ssh
 ```
3. Generate an [ed25519 key pair](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-2) (public and private keys):
  ```
  ssh-keygen -t ed25519
```
4. Set the file name and optional passphrase (press Enter for default in all 3 steps):

**Note:** You have the option to use a passphrase, and it is highly recommended. The security of a key pair, regardless of the encryption method, relies on ensuring that it remains inaccessible to anyone else.
 ```
Enter file in which to save the key (/home/user/.ssh/id_ed25519):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```
The public key is stored at `/home/your_local_username/.ssh/id_ed25519.pub`, while the private key is stored at `/home/your_local_username/.ssh/id_ed25519`.

5. Navigate to the .ssh directory:
```
cd ~/.ssh
```
6. Copy the content of the `id_ed25519.pub` file (public key).
7. Log in to the Ares supercomputer to place the generated public key.
```
ssh your_ares_login@ares.cyfronet.pl
```
9. Open the `ssh/authorized_keys` file in a text editor on Ares
```
nano ~/.ssh/authorized_keys
```
10. Paste the copied public key to the file and save it.
11. Logout from Ares and try loggin in to it again. After performing these steps, you should no longer be prompted for a password when connecting to Ares, as the SSH keys will be used for authentication. But if you set a passphrase during the SSH key creation, you'll be prompted to input it while logging in.

## Setting Up SSH Tunnel and Environment on Ares

1. Start SSH Server

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
The tunnel.sh script is designed for setting up an SSH tunnel on Ares and can work without loading the `gcc` and `python` modules. However, these modules are loaded in the script to provide a suitable environment for the SSH tunnel and for further work in Jupyter Notebooks. Have in mind that Ares may have older versions of Python. Customize the script and module loading according to your specific requirements.

The script initiates an SSH tunnel by running the SSH daemon (sshd) in the background. This tunnel listens on port 22223 and uses a configuration file located at /dev/null while utilizing the specified ed25519 public key.

**Additional Step:** If you don't have permission to execute the tunnel.sh file, grant it.
```
chmod u+x tunnel.sh
```
2. Modify `.bashrc` configuration

**Note:** Modifying the .bashrc file impacts the terminal environment. Changes affect environment variables, aliases, prompt appearance, and script execution. Incorrect modifications may lead to unintended issues with the terminal's functionality and programs.
```
nano .bashrc
```
3. At the end of a file add `ml python` and save it.
```
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

## Configuring Visual Studio Code


1. Launch VSCode, navigate to the Remote Explorer, expand "Remotes," and check for 'Tunnels/SSH'. 

<img width="440" alt="Zrzut ekranu 2024-01-14 o 16 24 49" src="https://github.com/mbiernacka/VSCode-Jupyter-Notebook-Ares/assets/75391342/57253f8e-79bb-4790-b402-52c29f339a26">


If not present, install the "Remote Explorer" and "Remote - SSH" extensions from the VSCode Extensions marketplace.

<img width="499" alt="Zrzut ekranu 2024-01-14 o 16 23 01" src="https://github.com/mbiernacka/VSCode-Jupyter-Notebook-Ares/assets/75391342/57e995d3-8247-409a-a073-81942d926dde">

<img width="674" alt="Zrzut ekranu 2024-01-14 o 16 23 11" src="https://github.com/mbiernacka/VSCode-Jupyter-Notebook-Ares/assets/75391342/d20d8678-9329-441b-a476-a3bfdde74273">



2. Click on the settings icon next to SSH, opening the Command Palette and select the first suggested file, typically located at `/Users/your_local_username/.ssh/config`.
<img width="951" alt="Zrzut ekranu 2024-01-14 o 16 27 22" src="https://github.com/mbiernacka/VSCode-Jupyter-Notebook-Ares/assets/75391342/789571a9-f64d-4e3a-bf3f-83a0d6c49998">


3. Paste the following configuration:
```
Host ares
    HostName ares.cyfronet.pl
    User your_ares_login

Host ares_worker
    HostName nodelist  # Replace with your copied nodelist (ex. ac1111)
    ProxyJump ares
    Port 22223      #Change to the desirable port
    User your_ares_login  # Replace with your Ares username
    StrictHostKeyChecking no
```
The ProxyJump directive establishes a secure connection to the computing node `ares_worker`. It achieves this by directing the data through the primary Ares server, `ares`, using the designated port and user credentials.

4. Optimizing Storage Usage on Ares

When managing storage on Ares, it's essential to navigate the distinctions between [storage spaces $HOME and $SCRATCH](https://docs.cyfronet.pl/display/~plgpawlik/Ares#Ares-Storage):

$HOME space is located on /net/people/plgrid/<login> and is best for storing personal applications and configuration files but is limited to 10GB quota. It is recommended to store small files, like virtual environments or repository content there. 

$SCRATCH space is located on /net/ascratch/people/<login> and its purpose is high-speed storage for short-lived data used in computations while having 100TB quota. It is optimal for large files or temporary data associated with computational tasks. Have in mind that data older than 30 days can be deleted without notice.

When you use VSCode or VSCode Insiders on Ares, these tools create a quite big folder (even up to 500 MB). Because there are lots of small files, it's recommended to keep this folder in $HOME. To do this, access the Command Palette again and open `Remote-SSH Settings`. In the `Remote.SSH: Server Install Path section`, add two paths:
```
/net/people/plgrid/your_ares_login/.vscode
```
<img width="817" alt="Zrzut ekranu 2024-01-14 o 17 14 12" src="https://github.com/mbiernacka/VSCode-Jupyter-Notebook-Ares/assets/75391342/fb0120b5-4fe5-4ff7-abac-52aac77f3597">

## Setting Up Jupyter Notebook in VSCode on Ares

1. While logged in to Ares, clone the repository you want to work on or create a folder to store your project. The location, again, depends on the project size and the number of files.
2. Navigate to the "Remote Explorer" in VSCode.
3. Expand the "SSH" section, and you should see `ares` and `ares_worker`.
4. Choose "Connect in New Window" on ares_worker. It will open the Welcome page of VSCode.
5. In newly opened page open the Command Palette, navigate to the cloned/created folder and open the terminal.
6. Create a new [virtual environment](https://devinschumacher.com/how-to-setup-jupyter-notebook-virtual-environment-vs-code-kernels/)
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
9. Make sure you have the Python extension and Jupyter extension installed in VSCode. If prompted by VSCode to install any additional packages on `ares_worker`, follow the suggestions and install them.
10.   In the Command Palette, choose "Create New Jupyter Notebook" or start working on existing files from cloned repository.
11.   You can select previously created virtual environment as Kernel.

Now, you have successfully set up and run a Jupyter Notebook in VSCode on Ares.





