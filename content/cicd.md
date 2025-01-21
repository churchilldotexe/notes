---
title: "Leveraging github actions for ci/cd"
date: 2025-01-17
description: "a notes for using github action technique as ci/cd to lessen the load of the vps"
tags:
  - laravel
  - php
  - vps
  - ssh
---

# Introduction

Github action is a github feature that allows you to automate your workflow by spinning up a server and run your project in there. In that server, it will build ,check,lint,format and test(if set) your project and will report/log the error if it fails or will log if successful. lastly, it will do a clean up for the assets and server. The last part is the important one because we can leverage the assets
built by github and sync it to our vps server.
This way, we dont have to do a build on our vps server and also we can ensure that the built assets are properly checked.
This gives a really good developer experience since you dont have to ssh to your server and build the project yourself everytime there is a change in the project/ or everytime we push to main branch.

## Github actions setup

In the current project, create a new file in the `.github/workflows` directory with the following name: `deploy.yml`

```yml
name: Check and Deploy laravel project to Production Server

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Setup PHP
        uses: shivammathur/setup-php@v3
        with:
          php-version: 8.3

      - name: Install Composer dependencies
        run: composer install --no-dev --no-interaction --no-progress --prefer-dist --optimize-autoloader

      - name: Install node.js
        uses: actions/setup-node@v3
        with:
          node-version: "22"

      - name: Install NPM dependencies
        run: npm ci

      - name: Build NPM assets
        run: npm run build

      - name: Synchronize assets to server
        uses: easingthemes/ssh-deploy@v2.1.5
        env:
          SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
          SOURCE: "."
          REMOTE_HOST: ${{ secrets.VPS_HOST }}
          REMOTE_USER: ${{ secrets.VPS_USER }}
          TARGET: ${{ secrets.TARGET }}

      - name: Run Remote/Artisan Commands
        uses: appleboy/ssh-action@v0.1.6
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd ${{ secrets.TARGET }}
            php artisan migrate --seed --force
            php artisan storage:link
            php artisan optimize # caching config files,routes,events,etc..
```

### `name`

The name of the workflow. It is recommended to give a meaningful name to the workflow.

### `on`

The event that will trigger the workflow.
Its like, when this workflow will run.
In this case, we are triggering the workflow on push to the main branch and on pull request to the main branch.

`jobs`: the jobs that will be executed in the workflow when the it triggers.
It is like a dictionary that contains the steps that will run.

`deploy` : is a `key` that is defined in the jobs. these key will have a certain steps or configurations to execute. Some of the **key** requires the previous steps

`runs-on`: the operating system that will run the workflow.

> [!TIP]
>
> If you are syncing the github actions to your vps, it is beneficial to use the same operating system and version that one that is running in your vps.

### `steps`

The step by step set of instructions that will be executed in the workflow.
Think of this as the step by step instruction of the commands and stuff that you need to do when deploying an application to production.

- `name` : the name of the step. It is better to give a name of what are the steps that you need to do.

- `uses` : Is a way to reference and run an action to the workflow.

  - `action` is just a reusable code that perform a certain task.

    - `with` : is just a parameter for the action. It is a way to configure the action on **how** you want to run it.

  **Example actions:**

  `actions/checkout@v3`: In this process we need to checkout the code from your github repo to the github workflows in order for it to gain access to the files and directory of that project.
  This is important because github actions need to run,build,format,lint and test the project.

  `shivammathur/setup-php@v2`: This action will instal phpcli and its extension together with composer on your github actions environment.

  ```bash
  with:
     php-version: 8.3
     extensions: ... #optionally specify extensions to install
     tools: composer:v2 # optionally specify composer version
  ```

  `easingthemes/ssh-deploy@v2.1.5` : This action will synchronize the compiled files and assets built by github actions to the our vps server.
  [Checkout env section](#env) for more information about the env of this action

  `appleboy/ssh-action@v0.1.6` : This action will run the commands on the vps server through github workflows by accessing your ssh and run the indicated script.

  - **code**:

    ```bash
    with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
    script: |
          cd ${{ secrets.TARGET }}
          php artisan migrate --seed --force
          php artisan storage:link
          php artisan optimize
    ```

    just like in the in the ssh-deploy action's [env](#env) the secrets here are the same because this action will need to access your ssh in order to run the scripts.

    `scripts`:
    `|` is the pipe operator that will help us run multiple commands or block of commands

    `php artisan migrate --force`: by default, if you run migrate in the production server php/laravel will prompt you about it. **force** , is use here in order to avoid that prompt.

    `php artisan optimize`: this command will optimize your projects files by caching config files,routes,events,etc..

- `run` : is a way to run a terminal command to the workflow.

  - `composer install --no-dev --no-interaction --no-progress --prefer-dist --optimize-autoloader`
    : This will run the command composer install with the following flags:

  `--no-dev` : will not install the dev dependencies since this we are setting up workflow for production.

  `--no-interaction` : will not ask any question during the installation process.

  `--no-progress` : will not show the progress of the installation process to reduce the clutter during the workflow process.

  `--prefer-dist` : will prefer the dist version of the package instead of the source code. This will significantly improve the installation since it will download the zip or tar version of the package and will not include metadata,git,test files unlike source package.
  This will also cache the by composer for faster repeated installs.

  `--optimize-autoloader` : will optimize the autoloader to reduce the loading time of the project to improve performance.
  This essentially means that to tell composer to generate a classmap which will significantly improve performance when we need to use the class.

- ### `env`

  Is a way to share a sensitive value to the workflow.

  - **Setting up env in github**:

    1. go to your project **github repository**
    2. then go to **Settings** tab
    3. next, at the side nav bar go to **Secrets and variables** tab
    4. under it click the **Actions** tab
    5. in the **Repository secrets** section, click the **New repository** button and add your secrets.
    6. the format will be:
       `Name:` is the secret key without the `secrets` e.g `secrets.TARGET` -> `TARGET`

       `Value:` is the value of the secret or env variable

  ```bash

  env:
     SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
     SOURCE: "."
     REMOTE_HOST: ${{ secrets.VPS_HOST }}
     REMOTE_USER: ${{ secrets.VPS_USER }}
     TARGET: ${{ secrets.TARGET }}
  ```

  `SSH_PRIVATE_KEY` : is the **private** key(not .pub) that we will use to connect to the vps server. which is located in `.ssh/`

  `SOURCE` : is the directory that we want to synchronize to the vps server.
  In this case, we are synchronizing the whole project to the vps server.
  `REMOTE_HOST` : is the vps server ip address.(use `ip addr` to get the ip address or check your vps provider's dashboard)
  `REMOTE_USER` : is the username of the vps server.
  `TARGET` : is the directory that we want to synchronize to the vps server. Since most of laravel project uses nginx it will be in `/var/www/html/<your-project-dir>`

When commit and push the code to the main branch, the workflow will run and the project will be deployed to the vps server if it didnt receive any error.
You'll know if its successful if you can see a check emoji (✅) beside the name of the workflow job
