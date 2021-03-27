+++
title = "How to setup a VPS - Debian, Docker"
date = 2020-07-24
description = ""

[extra]
extract_from = "content"
+++

In this article, I will describe how I setup my VPS servers based on Debian 10.
The goal of this setup is to have a working and secure Docker setup with
some monitoring and a few nice tools.

## Password manager

I won't elaborate on the absolute necessity to use a password manager. I am
using Keepass, and that's more than useful to store all those lengthy passwords.

I have chosen Keepass because it's free and open source software, and it's very
secure. On Windows, I recommend using the [official client][keepass]. On other
platforms, I find [KeepassXC][keepass-xc] more stable. Note that both are
compatible as they use the same file format to store passwords.

[keepass]: https://keepass.info/
[keepass-xc]: https://keepassxc.org/

## Debian 10

This is on you, as it varies for each cloud provider. I chose Debian because it
doesn't evolve too fast, so I can forget it for a while without fear that I am
late on updates. To be really safe doing so, I took the following actions:
- I subscribed to the [Debian security newsletter][debian-sec-newsletter].
- I skimmed through the [Debian Security FAQ][debian-security-faq].
- I skimmed through the [Securing Debian Manual][debian-sec-manual] (I hope I
  didn't overlook something).

**Credits**: I took inspiration from DigitalOcean's [great
tutorial][digitalocean-setup-debian-10] to write down those instructions. I
recommend reading it, as I will not go into the details of the procedure.

[debian-sec-newsletter]: https://lists.debian.org/debian-security-announce/
[debian-security-faq]: https://www.debian.org/security/faq
[debian-sec-manual]: https://www.debian.org/doc/user-manuals#securing
[digitalocean-setup-debian-10]: https://www.digitalocean.com/community/tutorials/initial-server-setup-with-debian-10

### Connecting via SSH and creating a new user

From your computer's terminal:

```bash
ssh root@your_server_ip

# Change your root password if you haven't done it yet
passwd
```

As we should not use `root` when not necessary, we will create a new user right
away (I will use `tlacroix`, replace it with whatever username you want). This
user will be a super-user, as we will use it for the following steps.

```bash
adduser tlacroix
usermod -aG sudo tlacroix
```

We can take the opportunity to do an update, just in case:

```bash
apt update
apt upgrade
```

### Configuring SSH key for the new user

**Credits**: [How to Set Up SSH Keys on Debian
10](https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys-on-debian-10).
Read it for more details.

Create a key on your personal computer (use one per computer!) if you don't
already have one. From you computer, do:

```bash
ssh-keygen -b 4096
```

I recommend using a passphrase, to stay safe when you forget to lock you
computer.

Now, we must copy the public key to the server. From your computer, do:

```bash
ssh-copy-id username@your_server_ip
```

Now you don't have to use your user password anymore, but the SSH key
passphrase.

### Securing SSH

Now that we can login as someone else than `root` without having to type a
password, let's forbid root login. Edit `/etc/ssh/sshd_config` and update as
follows:

```ini
PermitRootLogin no
PermitEmptyPasswords no
PasswordAuthentication no  <-- force using SSH keys
```

Then restart the SSH daemon:

```bash
sudo systemctl restart ssh
```

### Firewall - Initial setup

Now is a good time to setup a firewall, ~~to complicate our life~~ to control
what goes in and out of our server. We will come back to it later multiple
times, each time we will need to make a port visible to the outside world.

I was previously using `iptables`, but I find it too cumbersome to use. To
persist its configuration, I created a `systemd` service running a script during
boot, located under `/etc`, that contained calls to `iptables` to configure it.
However, this file had a non-obvious syntax (`iptables` CLI, actually), so I'm
moving to `ufw`, the *Uncomplicated Firewall*.

**Tip**: if, in the future, you have connectivity issues with some service
during initial setup, chances are high that it's because of the firewall. Always
keep that in mind.

From your server:

```bash
# Install UFW
sudo apt update
sudo apt install ufw

# Allow OpenSSH (otherwise we would be locked out)
sudo ufw allow OpenSSH

# Only do *after* the previous command
sudo ufw enable

# To have the current status of the firewall
sudo ufw status verbose
```

Now, if by accident you do get locked out of your server because of the
firewall, have a look for "KVM" in your provider's management interface, and
search the net for instructions.

### Unattended Upgrades

Last but not least, let's configure Unattended Upgrades. Quoting the [official documentation][unattended-upgrades]:

> The purpose of unattended-upgrades is to keep the computer current with the
> latest security (and other) updates automatically.
>
> If you plan to use it, you should have some means to monitor your systems,
> such as installing the apt-listchanges package and configuring it to send you
> emails about updates. And there is always /var/log/dpkg.log, or the files in
> /var/log/unattended-upgrades/.

Let's set everything up. I will only write down the commands with some comments.
Refer the the [official documentation][unattended-upgrades] for more details.

```bash
sudo apt-get install unattended-upgrades apt-listchanges
dpkg-reconfigure -plow unattended-upgrades
```

[unattended-upgrades]: https://wiki.debian.org/UnattendedUpgrades

#### Email configuration, to be notified during upgrades

To be notified by emails, update `/etc/apt/apt.conf.d/50unattended-upgrades`:

```cpp
Unattended-Upgrade::Mail "root";
```

**Credits**: [Redirection to the mailbox](https://blog.bobbyallen.me/2013/02/03/how-to-redirect-local-root-mail-to-an-external-email-address-on-linux/)

TODO: [Configure encryption](https://zurgl.com/how-to-configure-tls-encryption-in-postfix/).

```bash
# Install and configure postfix + mailx
sudo apt-get install postfix mailutils
## Chose "Internet mail" if domain name, otherwise "Local only"

# Setup redirection from local emails to external mailbox (only if "Internet Mail")
sudoedit /etc/aliases
## Add: root: my-email@mail-provider.net
sudo newaliases
sudo systemctl restart postfix

# Test it (look in your spam folder)
echo "This is a test." | mail -s "Test email from server" root
```

Phew, now Debian is configured.

## Miscellaneous tools

Those tools are not mandatory, but I find them convenient.

### Necessary tools

Those are just must have:

```bash
sudo apt install curl
```

### Mosh - Mobile SH

[https://mosh.org/](https://mosh.org/)

SSH connections easily hang up and disconnect, if your laptop goes to sleep or
your phone changes antenna. To solve this problem, I use Mosh, that basically
never dies.

```bash
sudo apt-get install mosh

# Allow 20 simultaneous connections in the firewall
sudo ufw allow 60001:60020/udp
```

Now, you should be able to use Mosh instead of SSH:

```bash
mosh user@server-ip
```

### tmux - Terminal multiplexer

tmux is another way of solving the disconnection issue I just mentioned, but is
not limited to that: as the name suggests, it enable multiplexing terminal, i.e.
having multiple terminals inside one. I won't elaborate much, because that would
require yet another post, and because the net is full of good resources.

```bash
sudo apt install tmux

# Basic how-to
## To run it, do only once, or after restarts of the server
tmux
## To attach to a running session
tmux a
```

tmux special key is `Ctrl+B`. Then, if you need quick help, use `Ctrl+B, ?`.

### ranger - VIM-inspired file manager

[https://github.com/ranger/ranger][ranger-github]

```bash
sudo apt install ranger

# To use it:
ranger
```

To get started, read [ranger's GitHub page][ranger-github].

[ranger-github]: https://github.com/ranger/ranger

### htop - interactive process viewer

[https://hisham.hm/htop/](https://hisham.hm/htop/)

```bash
sudo apt install htop

# To use it:
htop
htop -u user
```

If you want to hide threads (lines in green): **F2** > **Display options** > **Hide userland
process threads**.

### EarlyOOM - kills hungry processes early

Linux features an Out-Of-Memory Killer (OOM killer) that will terminate one or
more processes to free up memory for the system when the available memory is
exhausted. It selects the process to kill based on complex heuristics. However,
it is not configurable and I often found that it triggered too late.

EarlyOOM does the same job, but at a configurable threshold.

```bash
sudo apt install earlyoom

sudo systemctl status earlyoom

# To reduce the threshold to 4%
sudoedit /etc/default/earlyoom
## EARLYOOM_ARGS="-r 60 -m 4"
sudo systemctl restart earlyoom
```

The default threshold is at 10%, where a SIGTERM is sent, and 5% with a SIGKILL.
I reduced from 10% to 4% because my server is short on resources an I have
hungry services.

## Monitoring and alerting with Netdata

I won't elaborate on the necessity to have monitoring and alerting for an
infrastructure. [Netdata][netdata-website] is a free and open source solution
just for that.

I choose to install it independently as Docker, i.e. no as a Docker container,
to isolate it from Docker failures.

I recommend heading to the official website for instructions:
[https://learn.netdata.cloud/](https://learn.netdata.cloud/).

[netdata-website]: https://www.netdata.cloud/

**TL;DR**: If you want the stable channel:

```bash
sudo apt install curl
bash <(curl -Ss https://my-netdata.io/kickstart.sh) --stable-channel
```

Now, head to: [http://localhost:19999](http://localhost:19999)... And that won't
work, because the firewall won't let us enter. You can open the port as below,
read the following section or use a reverse-proxy such as Caddy or Nginx. Your
choice :)

```bash
sudo ufw allow 19999
```

### Netdata Cloud

Instead of opening a port and configuring your firewall, you can opt to use
[Netdata Cloud][netdata-cloud] instead. That's a free service to monitor as many
servers as you want (and offer them a backdoor in your infrastructure). Their
website is clear enough, so I won't go into the procedure.

[https://www.netdata.cloud/][netdata-cloud]

[netdata-cloud]: https://www.netdata.cloud/

## Small break: Break it all

Now is time to see how our system behaves in presence of failure. We have
EarlyOOM to kill bad processes (remember, it checks every 60 seconds by
default), Mosh that will never die, Netdata to alert us (it has default
alert rules) and `postfix` to forward emails right to our mailbox.

An easy and efficient way to hang the system is the fork bomb (explanations
[here][understand-fork-bomb]). Type that and observe:

```bash
:(){ :|:& };:
```

... Ok, EarlyOOM didn't help, and emails from postfix came afterwards or not at
all because the system was busy. But I don't have an idea for a fast way to
break it other than the fork bomb. Still, you should see red on Netdata Cloud
dashboard.

[understand-fork-bomb]: https://www.cyberciti.biz/faq/understanding-bash-fork-bomb/

Depending on your luck, your server will recover or not. If it does not, just
head to your management website and reboot it.

## Docker

Now let's go with Docker. I won't introduce it, again, the net is full of good
resources. As usual, I recommend reading the official instructions on the
website:

[https://docs.docker.com/install/linux/docker-ce/debian/](https://docs.docker.com/install/linux/docker-ce/debian/)

But here is a recap:

```bash
# Install necessary prerequisites
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg2 \
    software-properties-common

# Add Docker's official PGP key
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -

# Verify the key
# Should be: 9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88
sudo apt-key fingerprint 0EBFCD88

# Add the stable Docker repository
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"

# Install Docker Engine - Community
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io

# Verify the install
sudo docker run hello-world

# Add your user to the docker group to avoid having to use sudo each time
sudo usermod -aG docker <user>
# You need to logout and login
docker run hello-world

# Install docker-compose
sudo apt install docker-compose
```

---

In future articles:
- Caddy or Traefik?
- Docker Swarm vs Kubernetes
- Docker Swarm
- Swarmpit
