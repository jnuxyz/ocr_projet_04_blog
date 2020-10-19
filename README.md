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
after_script:
  - echo "job ${CI_JOB_NAME} end"

job-with-before-after-script:
  stage: build
  before_script:
    - echo "exécuté avant"
  script:
    - echo "script"
      "sur plusieurs"
    - echo "lignes"
    - cat <<< $(echo 'salut !')
  after_script:
    - echo "exécuté après"
```


`vim .bash_logout` (tout commenter)

`sudo gitlab-runner run`

## Github

> ocr_projet_04_blog > Settings GitHub Pages > 

pip install pelican markdown ghp-import