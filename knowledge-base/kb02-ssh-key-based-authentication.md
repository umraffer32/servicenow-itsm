# Configure SSH for key-based authentication and disable password login

> **ServiceNow article fields**
> - **Knowledge Base:** IT
> - **Category:** Security and Hardening
> - **Article type:** How To
> - **Short description:** Configure SSH for key-based authentication and disable password login
> - **Keywords:** SSH, ssh-keygen, ed25519, key-based authentication, password authentication, sshd_config, hardening, Linux

---

## Purpose

Move a Linux host from password-based Secure Shell (SSH) login to key-based authentication, then turn password login off. With password login disabled there is nothing to guess, so password brute-force is removed as a way onto the host.

## Environment

- Linux server or virtual machine (Ubuntu or Debian)
- OpenSSH server (`sshd`)
- Administrative (`sudo`) access on the host

## Read this first

Do not disable password login until you have confirmed key login works. Keep your current session open while you make these changes. If the key is misconfigured and password login is already off, you can lock yourself out of the host.

## Step 1. Generate a key pair on the client

```bash
ssh-keygen -t ed25519 -C "your-comment"
```

The ed25519 type is a good default. It gives strong security with a short, fixed-size key, it is fast, and it uses a signing method that avoids the random-number weaknesses that have exposed ECDSA private keys in the past.

If you give the key a custom name instead of the default `id_ed25519`, remember that detail for Step 3. A custom-named key is not tried automatically.

## Step 2. Copy the public key to the host

**Option A.** Use `ssh-copy-id`, which is the quickest when you have network SSH access already:

```bash
ssh-copy-id user@host
```

**Option B.** Append the public key by hand. This is handy when you are already at the console or in a SPICE window, where pasting is faster than typing the full command. Paste the contents of the `.pub` file into `~/.ssh/authorized_keys` on the host.

Either way, the permissions must be right or `sshd` ignores the file. Set the `~/.ssh` directory to `700` and `authorized_keys` to `600`.

## Step 3. Test key login before changing anything

From the client, confirm the key logs you in with no password prompt:

```bash
ssh user@host
```

If you gave the key a custom name, SSH tries the default key names first and the login fails. The easiest fix is an SSH config file. Define the host once in `~/.ssh/config` and connect by a short name from then on:

```
Host mybox
    HostName 10.200.50.20
    User youruser
    IdentityFile ~/.ssh/your-key-name
```

After that, `ssh mybox` uses the right key and user every time, with no flags to remember. If you would rather not set that up, point to the key directly on the command line instead:

```bash
ssh -i ~/.ssh/your-key-name user@host
```

## Step 4. Harden the SSH server config

Edit `/etc/ssh/sshd_config` and set these values:

```
PubkeyAuthentication yes
PasswordAuthentication no
KbdInteractiveAuthentication no
PermitRootLogin no
```

`PubkeyAuthentication` is usually on by default, but setting it makes the intent clear. `KbdInteractiveAuthentication no` closes the keyboard-interactive path so it cannot be used to fall back to a password.

## Step 5. Apply and verify

Restart the service:

```bash
sudo systemctl restart ssh
```

Rebooting the host also applies the change but is not necessary. On some distributions the service is named `sshd` instead of `ssh`.

Then verify from a new session. Key login should still work, and a forced password attempt should be refused:

```bash
ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no user@host
```

That should come back denied, which confirms password login is off.

## Automating this at scale

Doing this by hand is fine for a few machines. For more than that, manage `sshd_config` and `authorized_keys` with a configuration-management tool such as Ansible, so every new host is hardened the same way without editing each one by hand.
