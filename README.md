# GitHub Action for WP Engine Git Deployments

An action to deploy your repository to a **[WP Engine](https://wpengine.com)** site via git. [Read more](https://wpengine.com/git/) about WP Engine's git deployment support.

## Example GitHub Action workflow

```
name: Github Action with WP Engine Git Deploy

on:
  push:
    branches:
     - production
     - staging

jobs:
  build-and-deploy:
    name: Build and Deploy to WP Engine
    runs-on: ubuntu-18.04
    env:
      WPENGINE_SSH_KEY_PRIVATE: ${{ secrets.WPENGINE_SSH_KEY_PRIVATE }}
      WPENGINE_SSH_KEY_PUBLIC: ${{ secrets.WPENGINE_SSH_KEY_PUBLIC }}
    steps:
    - uses: actions/checkout@v1
    
    - name: Build Stuff
      run: |
        lando start &&
        cd wp-content/themes/mythemename && lando yarn build:production

    - name: Set deployment gitignores then Commit
      run: |
        rm wp-content/themes/mythemename/.gitignore
        rm wp-content/uploads/.htaccess
        cp dev/deployment/gitignore .gitignore &&
        timestamp=$(date "+%Y.%m.%d-%H.%M.%S") &&
        git status &&
        git add -A &&
        git commit -m "Github Actions Commit $timestamp"

    - name: Deploy to Production with GitHub Action for WP Engine Git
      if: github.ref == 'refs/heads/production'
      uses: campaignupgrade/github-action-wpengine-git-deploy@0.1.2
      env:
        WPENGINE_SITE_NAME: myinstallname
        LOCAL_BRANCH: 'production'

    - name: Deploy to WPEngine Staging Site
      if: github.ref == 'refs/heads/staging'  # if pushed to this branch
      uses: campaignupgrade/github-action-wpengine-git-deploy@master
      env:
        WPENGINE_SITE_NAME: 'mystagingname' # deploy to this WPE site
        LOCAL_BRANCH: 'staging' # deploy this branch's code
```

## Environment Variables & Secrets

### Required

| Name | Type | Usage |
|-|-|-|
| `WPENGINE_SITE_NAME` | Environment Variable | The name of the WP Engine site you want to deploy to. |
| `WPENGINE_SSH_KEY_PRIVATE` | Secret | Private SSH key of your WP Engine git deploy user. See below for SSH key usage. |
|  `WPENGINE_SSH_KEY_PUBLIC` | Secret | Public SSH key of your WP Engine git deploy user. See below for SSH key usage. |

### Optional

| Name | Type  | Usage |
|-|-|-|
| `WPENGINE_ENVIRONMENT` | Environment Variable  | Defaults to `production`. You shouldn't need to change this, but if you're using WP Engine's legacy staging, you can override the default and set to `staging` if needed. |
| `LOCAL_BRANCH` | Environment Variable  | Set which branch in your repository you'd like to push to WP Engine. Defaults to `master`. |

### Further reading

* [Defining environment variables in GitHub Actions](https://developer.github.com/actions/creating-github-actions/accessing-the-runtime-environment/#environment-variables)
* [Storing secrets in GitHub repositories](https://developer.github.com/actions/managing-workflows/storing-secrets/)

## Setting up your SSH keys

1. [Generate a new SSH key pair](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/) as a special deploy key. The simplest method is to generate a key pair with a blank passphrase, which creates an unencrypted private key.
2. Store your public and private keys in your GitHub repository as new 'Secrets' (under your repository settings), using the names `WPENGINE_SSH_KEY_PRIVATE` and `WPENGINE_SSH_KEY_PUBLIC` respectively. In theory, this replaces the need for encryption on the key itself, since GitHub repository secrets are encrypted by default.
3. Add the public key to your target WP Engine environment.
4. Per the [WP Engine documentation](https://wpengine.com/git/), it takes about 30-45 minutes for the new SSH key to become active.
