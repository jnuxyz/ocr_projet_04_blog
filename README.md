# OCR Projet 04 Pelican

## Gitlab

### Installation Gitlab

```shell
sudo apt-get install -y curl openssh-server ca-certificates tzdata
sudo apt-get install -y postfix  # Select 'Internet Site'
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
sudo EXTERNAL_URL="http://<IP_GITLAB>" apt-get install gitlab-ce
# Use the default account's username root to login
```

### Installation Gitlab Runner

```shell
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | sudo bash
export GITLAB_RUNNER_DISABLE_SKEL=true; sudo -E apt-get install gitlab-runner
```

### Configuration dépôt  

Sur Gitlab créer un dépôt nommé **blog**.

```shell
git clone http://<IP_GITLAB>/jnuxyz/blog.git
```

Dans le dossier **blog** :

  `vim .gitignore`

  ```shell
  venv
  output
  *.pyc
  ```

  ```shell
  git add .
  git commit -m "Initial commit"
  git push -u origin master
  ```

Ajouter 3 variables d'environnements au dépôt:
  > Gitlab > blog > Settings > CI / CD Settings > Variables > Add variable

- NAME_GITHUB  : Nom du profil Github
- EMAIL_GITHUB : Email du profil Github
- TOKEN_GITHUB : Token du profil Github
  > Github > Personal settings > Developer settings > Personal access tokens > Genrate new token > Select scopes > repo

### Configuration Gitlab Runner
  
> Gitlab > blog > Settings > CI / CD Settings > Runners > Specific Runners > Set up a specific Runner manually  

```shell
sudo gitlab-runner register  # Shell
```

### Configuration pipeline

Dans le dossier **blog** :

```shell
vim .gitlab-ci.yml
```

```yaml
stages:
  - build
  - deploy

build-articles:
  stage: build
  script:
    - virtualenv -p python3 venv
    - source venv/bin/activate
    - pip install pelican Markdown
    - git clone https://jnuxyz:$TOKEN_GITHUB@github.com/jnuxyz/ocr_projet_04_blog.git --branch=gh-pages gh-pages
    - git config --global user.email "$EMAIL_GITHUB"
    - git config --global user.name "$NAME_GITHUB"
    - rm -rf gh-pages/*
    - pelican -s pelicanconf.py -o gh-pages
  artifacts:
    paths:
      - gh-pages/

deploy-github-pages:
  stage: deploy
  script:
    - cd gh-pages
    - git add .
    - git commit -m "Update via CI"
    - git push origin gh-pages
  dependencies:
    - build-articles
  when: manual
```

## Pelican

### Installation Pelican

```shell
sudo apt-get install -y python3-virtualenv python3-dev python3-pip virtualenvwrapper
# echo "export PYTHONDONTWRITEBYTECODE=1" >> ~/.bashrc; source ~/.bashrc
```

Dans le dossier **blog** :

```shell
virtualenv -p python3 venv
source venv/bin/activate
pip install pelican Markdown
```

### Utilisation Pelican

Dans le dossier **blog** :

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

## Github

### Dépôt **ocr_projet_04_blog**

Récuperer le dépôt :

```shell
git clone https://github.com/jnuxyz/ocr_projet_04_blog.git --branch=gh-pages
```

Si branch **gh-pages** inexistante :

  ```shell
  git clone https://github.com/jnuxyz/ocr_projet_04_blog.git
  git checkout -b gh-pages
  echo "<h1>HelloWorld</h1>" > index.html
  git add .
  git commit -m "gh-pages"  
  git push -u origin gh-pages
  ```

### Configuration GitHub Pages

  > Github > ocr_projet_04_blog > Settings > GitHub Pages > Source > Branch:gh-pages

### Envoi des Sources

A la racine utilisateur :

```shell
rm -rf ocr_projet_04_blog/*
cp -R blog/output/* ocr_projet_04_blog/
# pelican -s pelicanconf.py -o ocr_projet_04_blog
cd ocr_projet_04_blog
git add .
git commit -m "Update"
git push origin gh-pages
```

## Utilisation Pipeline

Dans le dossier **blog** :

- Modifier ou ajouter des articles et des pages dans le sous-dossier *content*.
- Envoyer les modifications sur Gitlab :

  ```shell
  git add .
  git commit -m "Update"
  git push origin master
  ```

Voir le Job de la pipeline sur Gitlab :
  > Gitlab > blog > CI / CD > Jobs

Se rendre sur la page Github : [jnuxyz.github.io/ocr_projet_04_blog](https://jnuxyz.github.io/ocr_projet_04_blog/)
