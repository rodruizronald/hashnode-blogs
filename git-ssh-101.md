---
title: SSH 101 for Git users
domain: rodruizronald.hashnode.dev
tags: ssh, git, github, bitbucket
slug: git-ssh-101
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1658846971216/--ElfsxEi.jpg?auto=compress
---

Are you tired of typing your credentials when using HTTPS? The SSH protocol allows you to connect and authenticate to remote Git repositories without providing your credentials each time.

To follow this post, you will need:
- OpenSSH and Git installed in your OS of choice. 
- An account in GitHub or Bitbucket.

Even though I chose GitHub to illustrate some steps, the fundamentals should be the same for Bitbucket.

### Connecting via SSH

1. ##### Check for existing SSH keys

	The `~/.ssh` directory is the default location where keys are stored. Open a terminal and enter the following command to check for existing keys.

	```bash
	ls -la ~/.ssh
	```

	> **Note**: Windows users must use PowerShell to access the home directory as `~`.

	By default, the filenames of supported public keys for GitHub or Bitbucket are one of the following:

	- *id_rsa.pub*
	- *id_ecdsa.pub*
	- *id_ed25519.pub*

	Go to step two [Generate new SSH key](#generate-new-ssh-key) if you don't have a key; otherwise, go to step three [Add new SSH key](#add-new-ssh-key).

	> **Note**: If the `~/.ssh` directory doesn't exist, then you don't have any existing SSH key pair.

2. ##### Generate new SSH key

	Here we use the `ssh-keygen` command to create a new authentication key pair. Both GitHub and Bitbucket recommend using the key algorithm `ed25519`. Open a terminal and paste the following command, substituting `gitpersonal@gmail.com` with your email address.

	```bash
	ssh-keygen -t ed25519 -C "gitpersonal@gmail.com" -f ~/.ssh/id_ed25519_gitpersonal
	```

	*Flags*
	- `-t` Key algorithm type.
	- `-C`  Optional comment. The email doesn't have to be a "real email"; it's mainly added as a source of information in case you have multiple keys for different GitHub or Bitbucket accounts.
	- `-f` Specifies the key filename.

	You will be asked to enter a passphrase when generating the SSH key pair. The passphrase is an authentication layer that protects access to the private part of your key pair in case someone steals them from you. It's possible to leave the passphrase empty, but no authentication is required when using your SSH key.

	> **Note**:  The ssh-keygen command creates the `~/.ssh` directory if it doesn't exist yet, but only if you use the default location.

3. ##### Add new SSH key

	Adding the key is a process that depends on the platform of your choice. I use GitHub, so the following explanation is mainly for GitHub users. However, it should be relatively similar in Bitbucket.

	First, copy the content of the SSH public key file without any newlines or whitespace. For Linux users, open a terminal and enter the following command.

	```bash
	cat ~/.ssh/id_ed25519_gitpersonal.pub
	```

	In GitHub, go to **Setting > SSH and GPG keys**, and follow the below steps.
	
	![alt](https://cdn.hashnode.com/res/hashnode/image/upload/v1658840213081/1a7XekAIr.png?auto=compress)
	
	![alt](https://cdn.hashnode.com/res/hashnode/image/upload/v1658840172075/RA1Eaa8Da.png?auto=compress)
	
	![alt](https://cdn.hashnode.com/res/hashnode/image/upload/v1658840230175/CxTK33kOJ.png?auto=compress)

4. ##### SSH authentication agent

	SSH key pairs are protected with a passphrase if you didn't leave it empty. If you left the SSH key passphrase empty, authentication is not required so you can skip this part and go to [Test SSH connection](#test-ssh-connection). To avoid typing your passphrase, you could configure an authentication agent so that you won't have to reenter your passphrase every time you use your SSH keys.
	
	> **Note**: An SSH agent is integrated into the GNOME keyring known as "Password and keys" for Linux users using GNOME-based distributions. It works out of the box; on the first connection attempt, an authentication window pops up asking for the SSH key passphrase. The passphrase is kept in the keyring until the user logs out.

	The OpenSSH suite provides two useful command-line utilities, `ssh-agent` and `ssh-add`, to ease authentication. To use the `ssh-agent` command for non-interactive authentication, open a terminal and start the agent.

	``` bash
	eval `ssh-agent`
	```

	You will then see the PID of the ssh-agent as follows:

	```bash
	Agent pid 27341
	```

	Now that the `ssh-agent` is running in the background, you must provide the passphrase of your SSH private keys. To do this, use the `ssh-add` command followed by the filename of the private key, for example:

	```bash
	ssh-add ~/.ssh/id_ed25519_gitpersonal # never expires
	ssh-add -t 1h50 ~/.ssh/id_ed25519_gitpersonal # expires in 1 hour and 50 minutes
	```

	*Flags*
	- `-t` Sets a maximum lifetime when adding identities to an agent.

	The passphrase will be removed from system memory when you log out of your shell or close the terminal session that started the `ssh-agent`.

5. ##### Test SSH connection

	To connect, open a terminal and enter the following command depending on your platform.

	*GitHub* 
	
	```bash
	ssh -T git@github.com
	```
	
	*Bitbucket*
	
	```bash
	ssh -T git@bitbucket.org 
	```
	
	When connecting for the first time to a remote server, SSH asks for confirmation to force validation of the remote party identity. Here you must check that the fingerprint printed out in the terminal matches the fingerprint of your platform of choice. Links to get public key fingerprints:
	- [GitHub](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints)
	- [Bitbucket](https://support.atlassian.com/bitbucket-cloud/docs/configure-ssh-and-two-step-verification/)

	While trying to connect, I got an `ECDSA` key fingerprint with the value `p2QAMXNIC1TJYWeIOttrVc98/R1BUFWu3/LiyKgUfQM`. I checked this value against GitHub's SSH key fingerprints and confirmed that I was indeed connecting to GitHub. This manual validation it's crucial as it helps to prevent DNS spoofing.

	![alt](https://cdn.hashnode.com/res/hashnode/image/upload/v1658840250391/h68rG4HFf.png?auto=compress)

	> **Note**: GitHub has a broad range of public IP addresses, so it's okay if you got an IP different than `140.82.121.4`.

### Known hosts file

As the previous section shows, connecting to GitHub for the first time requires user confirmation. Then after confirming, the user is warned with the following message, `Permanently added 'github.com,140.82.121.4' (ECDSA) to the list of known hosts`. What does this mean? SSH uses a file named `known_hosts` to keep track of servers a client has confirmed as trustable by allowing a connection.

> **Note**: OpenSSH creates the `known_hosts` file. You don't have to create it yourself.

The `known_hosts` file is in the hidden directory `~/.ssh`. Its content is hashed by default as specified in the system-wide configuration file. Hashed hostnames start with a '|' character, and only one hashed hostname can appear on a single line.

Most public key files are structured as follows: *hostname* *key-type* *public-key*.

Below you can see the two entries added to my `known_hosts` file after connecting to GitHub for the first time. One hostname per line. The hostnames are provided in the warning messages `github.com` and `140.82.121.4`.The *key-type* and *public-key* are the same since it's the same host.

``` bash
|1|AU2jWbC9KZDnLeNLHDyCmRlQhLA=|FM3qYxWKnafJHQ4prGIQYDaMjRo= ecdsa-sha2-nistp256 AAAAE2Vj...Z2YB/++Tpockg=
|1|/z70og5Vj7QqXgqs+SGwh5p8xdQ=|NdkTR8nmgE5CO9IunnkzZJdzWk0= ecdsa-sha2-nistp256 AAAAE2Vj...Z2YB/++Tpockg=
```

After playing with GitHub via SSH for a while, I got another warning message adding a new hostname, `140.82.121.3`, but this time I didn't have to confirm to continue. Why is that? Well, it all comes down to the host SSH key. If the host public SSH key matches one of the keys in the `known_hosts` file, then no confirmation is needed to continue. Below is the new entry added.

``` bash
|1|AU2jWbC9KZDnLeNLHDyCmRlQhLA=|FM3qYxWKnafJHQ4prGIQYDaMjRo= ecdsa-sha2-nistp256 AAAAE2Vj...Z2YB/++Tpockg=
|1|/z70og5Vj7QqXgqs+SGwh5p8xdQ=|NdkTR8nmgE5CO9IunnkzZJdzWk0= ecdsa-sha2-nistp256 AAAAE2Vj...Z2YB/++Tpockg=
|1|D7D+lg7nTVUmBZrOUc7Vhju1vNc=|ohU4287qDBACVQgLQSOzrdr4iPk= ecdsa-sha2-nistp256 AAAAE2Vj...Z2YB/++Tpockg=
```

Platforms such as GitHub and Bitbucket use [DNS load balancing](https://en.wikipedia.org/wiki/Round-robin_DNS) to deal with high loads of internet traffic so each one has a set of public IPs for which users can access their services. That is why at first I got `140.82.121.4`, but later it was `140.82.121.3`.

List of list of public IPs:
- [GitHub](https://api.github.com/meta)
- [Bitbucket](https://support.atlassian.com/bitbucket-cloud/docs/what-are-the-bitbucket-cloud-ip-addresses-i-should-use-to-configure-my-corporate-firewall/)

I find it helpful to disable hashing to have the hostnames in a human-readable format. Having them in this format allows me to clean up the `known_hosts` file when needed quickly. Please be aware the hostnames are hashed by default to hide sensible information, mainly employed for internal enterprise networks.

With OpenSSH, you can create a user configuration file to customize SSH based on your needs, such as disabling hashing. The file format and configuration options are described in `ssh_config(5)`. Because of the potential for abuse, this file must have strict permissions, read/write for the user, and not writable by others. Create a file named `config` in the hidden directory `~/.ssh` with the option `HashKnownHosts` set to `no`.

``` bash
cd ~/.ssh
echo 'HashKnownHosts no' > config
chmod 644 config
```

Hashing is only disabled for new entries, all previous entries are written into the file stay hashed. When using human-readable format hostnames are a comma-separated list of patterns, so you can have more than one in a single line.

``` bash
github.com,140.82.121.4 ecdsa-sha2-nistp256 AAAAE2Vj...Z2YB/++Tpockg=
140.82.121.3 ecdsa-sha2-nistp256 AAAAE2Vj...Z2YB/++Tpockg=
```

### Manage multiple accounts

Let's imagine you have two GitHub accounts, your personal account named `gitpersonal` and your company account named `gitcompany`, and you want to use both from your laptop with the same SSH key; you cannot, and if you try, you will get the following error in GitHub: `Key is already in use`. To solve this problem, follow the steps below.

1. Generate a key pair for each account, lets name them `id_ed25519_gitpersonal` and `id_ed25519_gitcompany`. So the `~/.ssh` directory should look something like this:
	
	``` bash
	├── config
	├── id_ed25519_gitcompany
	├── id_ed25519_gitcompany.pub
	├── id_ed25519_gitpersonal
	├── id_ed25519_gitpersonal.pub
	└── known_hosts
	```

2. Define the hosts in the user configuration file. To do this, open the `config` file previously created, and then add the following lines at the end:

	``` bash
	# Personal account
	Host gitpersonal
		User git
		HostName github.com
		PreferredAuthentications publickey
		IdentityFile ~/.ssh/id_ed25519_gitpersonal
	
	# Company account
	Host gitcompany
		User git
		HostName github.com
		PreferredAuthentications publickey
		IdentityFile ~/.ssh/id_ed25519_gitcompany
	```

	Only two command-line options change between accounts, the `Host` and `IdentityFile`. The `Host` is the hostname argument given on the command-line, see it as an alias name that points to a set of connection parameters. Then the `IdentityFile` is the file containing the private key for accessing the GitHub or Bitbucket account.

3. Assuming that the SSH agent is already running, add the SSH private keys to the agent:

	``` bash
	ssh-add ~/.ssh/id_ed25519_gitpersonal
	ssh-add ~/.ssh/id_ed25519_gitcompany
	```

4. Test the connection on both accounts. The host alias name is used to identify the configuration parameters in the `config` file.

	``` bash
	ssh -T git@gitpersonal
	ssh -T git@gitcompany
	```

	If everything worked, you should get the `You've successfully authenticated` message for both.
	
5. Clone repositories. Notice that you have to modify the default `git@github.com` or `git@bitbucket.org` with your host alias name, for instance `git@gitpersonal`.

	``` bash
	git clone git@gitpersonal:leachim6/hello-world.git
	```
	
	It's pretty convenient to use the host alias name `github.com` or `bitbucket.org` for the account you use the most so there is no need to manually change the URI. For instance, if you use your personal account the most then you could leave it as:
	
	``` bash
	# Personal account
	Host github.com
		User git
		HostName github.com
		PreferredAuthentications publickey
		IdentityFile ~/.ssh/id_ed25519_gitpersonal
	```