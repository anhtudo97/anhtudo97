## Deploy Node.js to VPS using Github Actions
![](https://raw.githubusercontent.com/danielwetan/node1/master/nodejs-github-actions.png)


> Steps to deploy Node.js to VPS using PM2 and Github Actions


#### 1. Clone repo to VPS folder

```bash
git clone https://github.com/danielwetan/node1.git
```

#### 2. Setup CI workflows

location: `node1/.github/workflows/ci.yml`

example: https://github.com/danielwetan/node1/blob/master/.github/workflows/ci.yml

```yaml
# This workflow will do a clean install of node dependencies, 
# build the source code and run tests across different versions of node
# For more information see: 
# https://help.github.com/actions/language-and-framework-guides/using-nodejs-with-github-actions

name: Node.js CI

on:
  pull_request:
    branches: [ master ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Use Node.js 12
      uses: actions/setup-node@v2
      with:
        node-version: 12.x
    - run: npm i
    - run: npm run build --if-present
    - run: npm test
```

#### 3. Setup CD workflows

location: `node1/.github/workflows/cd.yml`

example: https://github.com/anhtudo97/login-register-animation/blob/master/.github/workflows/main.yml

```yaml
# This is a basic workflow to help you get started with Actions

name: Node.js CD

# Controls when the action will run. 
on:
  # Triggers the workflow on push or pull request events but only for the master branch
  push:
    branches: [ master ]

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
    - name: Deploy using ssh
      uses: appleboy/ssh-action@master
      with:
        host: ${{ secrets.HOST }}
        username: ${{ secrets.USERNAME }}
        key: ${{ secrets.PRIVATE_KEY }}
        port: 22
        script: |
          cd ~/home/danielwetan/apps/node1
          git pull origin master
          git status
          npm install --only=prod
          pm2 restart node1
```

#### 4. Setup Private Key and Public Key

```bash
ssh-keygen -t rsa -b 4096 -m PEM -C "github-actions-node1"

# view the key value
cat id_rsa-github-node1.pub # Public Key
cat id_rsa-github-node1 # Private Key
```

#### 5. Copy Public Key to `authorized key`

```bash
cat id_rsa-github-node1.pub

# Copy the output to
vi ~/.ssh/authorized_keys
```

#### 6. Copy Private Key
```bash
# Type and copy the output
cat ~/.ssh/id_rsa
```

#### 7. Setup Github Secret keys

```bash
# Github Secret location
Settings -> Secrets -> Actions -> New repository secret

PRIVATE_KEY = "Copy generated private key from vps to github secret"
HOST = "YOUR SERVER ADDRESS, example: 172.41.91.123" 
USERNAME = "YOUR SERVER USERNAME, example: daniel"
```

---
Reference:

[https://www.youtube.com/watch?v=pvPJARjqAa8](https://www.youtube.com/watch?v=pvPJARjqAa8)

[https://github.com/appleboy/ssh-action](https://github.com/appleboy/ssh-action)
