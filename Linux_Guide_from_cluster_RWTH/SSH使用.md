# SSH使用

**Basic Use**

`ssh [options] <username>@<hostname>` 

**SSH Configuration**

`~/.ssh` config file `config`

- One preset one cluster
- Use `ssh <presetname>`

**Key-based Authentication**

public (for cluster)/private (for local) key pair

Treat private key file like a physical key

**Key pair workflow**

- `ssh-keygen` generate key pair
    - Enter filename for new key. Inside `~/.ssh`
    - Enter passphrase
    - Confirm passphrase
- `ssh-copy-id` copy public key to cluster
    - On *local* PC use `ssh-copy-id -i <keyfile> <user>@<host>`
    - Alternatively, copy manually
        - *local* PC: open public key file with text editor
        - One line of text: 3 parts: algorithm, key, comment
        - On *cluster* open `~/.ssh/authorized_keys`
        - paste line adjust comment as needed

**scp**

`scp [options] sourcehost:sourcefile targethost:targetfile`

`-r` for the whole directory