# OCR Projet 04 Pelican

## Installation Gitlab

### Gitlab

```shell
sudo mkdir -p /data/gitlab/{config,data,logs}

sudo docker run --detach --hostname localhost --publish 4433:443 --publish 8080:80 --publish 2022:22 --name gitlab --restart always --volume /data/gitlab/config:/etc/gitlab --volume /data/gitlab/logs:/var/log/gitlab --volume /data/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce:latest

mkdir -p /data/gitlab/runner

docker run -d --name gitlab-runner --restart always -v /var/run/docker.sock:/var/run/docker.sock -v /data/gitlab-runner:/etc/gitlab-runner gitlab/gitlab-runner:latest
```

### Nginx

```shell
sudo apt install nginx

sudo rm /etc/nginx/sites-enabled/default

sudo vim /etc/nginx/sites-available/gitlab-proxy.conf
```

```nginx
server {
  listen 80;
  server_name localhost;

  location / {
    proxy_pass http://0.0.0.0:8080;
    proxy_set_header        X-Real-IP  $remote_addr;
  }
}
```

```shell
sudo ln -s /etc/nginx/sites-available/gitlab-proxy.conf /etc/nginx/sites-enabled/

sudo service nginx reload
```

## Installation Gitlab Runner

```shell
curl -LJO https://gitlab-runner-downloads.s3.amazonaws.com/latest/deb/gitlab-runner_amd64.deb

sudo dpkg -i gitlab-runner_amd64.deb
```

## Pelican

### Installation

```shell
sudo apt-get install -y python3-virtualenv python3-dev python3-pip virtualenvwrapper
echo "export PYTHONDONTWRITEBYTECODE=1" > ~/.bashrc; source ~/.bashrc

mkdir blog
cd blog
virtualenv -p python3 venv
source venv/bin/activate

pip install pelican Markdown typogrify
```

### Utilisation

Utiliser des URLS relative pour Github

```shell
pelican-quickstart
pelican

vim content/helloword.md
```

```markdown
Title: Hello Word
Date: 2020-10-13 16:00
Tags: helloword, hello, world
Category: Blog
Authors: Jnux
Summary: Mon premier article avec Pelican.

# HELLOWORD

Un HelloWord pour tester Pelican
```

```shell
pelican

pelican --listen -b 0.0.0.0
```

## Gitlab

### Configuration dépôt

`vim .gitignore `

```shell
venv
output
*.pyc
```

```shell
git init
git remote add origin http://localhost:8080/jnux/blog.git
git add .
git commit -m "Initial commit"
git push -u origin master
```

### Configuration runner

>Gitlab > blog > CI / CD Settings > Runners > Specific Runners > Set up a specific Runner manually

```shell
sudo gitlab-runner register

vim .gitlab-ci.yml
```

```yaml
job:
  stage: build
  script:
    - cd /home/vagrant/blog/
    - venv/bin/pelican content -o /home/vagrant/ocr_projet_04_blog -s pelicanconf.py
    - cd /home/vagrant/ocr_projet_04_blog/
    - git add .
    - git commit -m "Update"
    - git push origin gh-pages
```


`vim .bash_logout` (tout commenter)

`sudo gitlab-runner run`

## Github

### Sur dépôt **ocr_projet_04_blog**

```shell
git checkout -b gh-pages
echo "<h1>HelloWorld</h1>" > index.html
git add . 
git commit -m "gh-pages"  
git push -u origin gh-pages
```

> Gitlab > ocr_projet_04_blog > Settings GitHub Pages > Source > Branch:gh-pages

Test :
  `jnuxyz.github.io/ocr_projet_04_blog`

### SSH

Sur la VM :

```shell
ssh-keygen -t ed25519 -C "Vagrant"
vim .ssh/id_ed25519.pub
```

> User setting > SSH keys > New SSH Key
Test : `ssh -T git@github.com`

### Sur VM

A la racine utilisateur :

```shell
git clone git@github.com:jnuxyz/ocr_projet_04_blog.git
cd ocr_projet_04_blog/
git checkout gh-pages
~/blog/venv/bin/pelican ~/blog/content -o ~/ocr_projet_04_blog -s ~/blog/pelicanconf.py
git add . && git commit -m "Update" && git push origin gh-pages
```

