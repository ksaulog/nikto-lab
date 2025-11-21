
## Requirements to run lab

1. Kali Linux VM (Nikto comes pre-installed with Kali)
2. Docker (Optional but recommended)
3. Some vulnerable web server
# Lab Steps
## Step 1: Install Docker

We install Docker to make it easier to acquire and run some publicly available web applications. To install Docker, we will follow their [installation guide](https://docs.docker.com/engine/install/debian/) for installing Docker on Debian, which Kali Linux is based on. 

First uninstall potentially conflicting packages:

```shell
sudo apt remove $(dpkg --get-selections docker.io docker-compose docker-doc podman-docker containerd runc | cut -f1)
```

There are multiple ways to download Docker, but we will be using the `apt` repository. Following the guide, we have to add the Docker repository to Apt sources:

```shell
sudo apt update
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/debian
Suites: bookworm    # Kali Linux's stable debian release
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```

Now that that `apt` basically knows where to get Docker from, install the latest versions of the Docker packages.

```shell
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

To verify the installation and that docker is running, run:

```shell
sudo systemctl status docker
```

If it's not running, start Docker using:

```shell
sudo systemctl start docker
```

## Step 2. Download a vulnerable web application

To demonstrate Nikto, we need a target web application. Nikto is an active scanner so you it's not advisable to use it on websites you don't have permissions to test on. Luckily, there are many vulnerable websites that are designed for teaching web security. The web app that we will be using for this lab is called [OWASP Mutillidae II](https://owasp.org/www-project-mutillidae-ii/) which it is built on a LAMP stack. 

There are multiple ways of building the Mutillidae website, but for ease we will be using a pre-built image available on DockerHub.

First grab the code from [mutillidae-dockerhub](https://github.com/webpwnized/mutillidae-dockerhub):

```shell
git clone https://github.com/webpwnized/mutillidae-dockerhub.git
cd mutillidae-dockerhub.git
```
> `git` is readily available with Kali Linux, install if you don't have it

Notice that there is a **docker-compose.yml** file in the directory we just downloaded. The file gives instructions to `docker compose` to pull the images from DockerHub, run them, and connect them through the instructed ports and networks.

To initialize the containers:
```shell
docker compose up
```

Now the website is running on [127.0.0.1](127.0.0.1). Very simple compared to building it yourself!

When you first access the website, the site will tell you that the database hasn't been initialized and will direct you to click on a link to initialize it. Once the database has been set up, now is a good time to scan the website using Nikto.

## Step 3: Running Nikto commands

To get started with Nikto, let's see what options it has. Run:

```shell
nikto
```

At the top of the output, you will see it says:

```shell
+ ERROR: No host (-host) specified
```

This means that at the minimum, you need to supply Nikto with a target host IP or URL. You will also see the other options available to tweak how the tool behaves. 

To run Nikto on our server, specify the IP address