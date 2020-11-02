# OCR Projet 04 Pelican

## Installation Gitlab

```shell
sudo apt-get install -y curl openssh-server ca-certificates tzdata
sudo apt-get install -y postfix  # Select 'Internet Site'
sudo EXTERNAL_URL="http://<IP_GITLAB>" apt-get install gitlab-ce
# Use the default account's username root to login
```

## Installation Gitlab Runner

```shell
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | sudo bash
export GITLAB_RUNNER_DISABLE_SKEL=true; sudo -E apt-get install gitlab-runner
```

## Pelican

### Installation

```shell
sudo apt-get install -y python3-virtualenv python3-dev python3-pip virtualenvwrapper
echo "export PYTHONDONTWRITEBYTECODE=1" >> ~/.bashrc; source ~/.bashrc

mkdir pelican
cd pelican
virtualenv -p python3 venv
source venv/bin/activate

pip install pelican Markdown typogrify
```

### Utilisation

Dans le dossier **pelican** :
```shell
pelican-quickstart
vim pelicanconf.py  # Décommenter "RELATIVE_URLS = True" pour Github Pages
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

Dans le dossier **pelican** :
`vim .gitignore `
```shell
venv
output
*.pyc
```

```shell
git init
git remote add origin http://<IP_GITLAB>/jnuxyz/pelican.git
git add .
git commit -m "Initial commit"
git push -u origin master
```

> Gitlab > pelican > CI / CD Settings > Variables > Add variable

Ajouter 3 variables :
- NAME_GITHUB  : Nom du profil Github
- EMAIL_GITHUB : Email du profil Github
- TOKEN_GITHUB : Token du profil Github
  > Github > Personal settings > Developer settings > Personal access tokens > Genrate nex token > Select scopes **repo**


### Configuration runner

> Gitlab > Admin Area > Runners > Set up a shared Runner manually
ou
> Gitlab > pelican > CI / CD Settings > Runners > Specific Runners > Set up a specific Runner manually

```shell
sudo gitlab-runner register  # Docker
```

Dans le dossier **pelican** :
```shell
vim .gitlab-ci.yml
```
```yaml
image: python:rc-slim

before_script:
  - apt-get update
  - apt-get -y install git
  - pip3 install pelican Markdown typogrify

gh_pages:
  script:
    - git clone https://jnuxyz:$TOKEN_GITHUB@github.com/jnuxyz/ocr_projet_04_blog.git --branch=gh-pages gh-pages
    - git config --global user.email "$EMAIL_GITHUB"
    - git config --global user.name "$NAME_GITHUB"
    - rm -rf gh-pages/*
    - pelican -s pelicanconf.py -o gh-pages
    - cd gh-pages
    - git add .
    - git commit -m "Update via CI"
    - git push origin gh-pages
```

## Github

### Sur dépôt **ocr_projet_04_blog**

```shell
git clone https://github.com/jnuxyz/ocr_projet_04_blog.git
cd ocr_projet_04_blog
git checkout gh-pages
```

Si branch **gh-pages** inexistante :

  ```shell
  git checkout -b gh-pages
  echo "<h1>HelloWorld</h1>" > index.html
  git add . 
  git commit -m "gh-pages"  
  git push -u origin gh-pages
  ```

### Configuration 
  > Github > ocr_projet_04_blog > Settings > GitHub Pages > Source > Branch:gh-pages


# Utilisation

Dans le dossier **pelican** :
  - Modifier ou ajouter des articles et des pages dans le sous-dossier *content*.
  - Envoyer les modifications sur Gitlab :
    ```shell
    git add .
    git commit -m "Update"
    git push origin master
    ```

Voir le Job de la pipeline sur Gitlab :
  > Gitlab > pelican > CI / CD > Jobs    
    
Se rendre sur la page Github [jnuxyz.github.io/ocr_projet_04_blog](https://jnuxyz.github.io/ocr_projet_04_blog/)
