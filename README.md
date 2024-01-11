# How to use Jupyer notebooks in VSCode on Ares?

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
