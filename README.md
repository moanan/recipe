# Blogging with MacOS

## Setup Hexo
```
brew install node
npm install -g hexo-cli
npm install hexo-deployer-git --save
```
## Setup Theme Maupassant
```
cd /your_blog_dir/
git clone https://github.com/tufu9441/maupassant-hexo.git themes/maupassant
npm install hexo-renderer-pug --save
npm install hexo-renderer-sass --save
```

## Usage
```bash
# create new post
hexo new 2020-01-01-whatever
# start local server
hexo s
# generate static files
hexo g
# deploy to github pages
hexo d
```

# Blogging with Docker
The currently using [Maupassant Theme](https://github.com/tufu9441/maupassant-hexo) requires `hexo-renderer-pug` and `hexo-renderer-sass` which often mess up with Node versions and dependencies. 

To solve this, I use a modified [Docker hexo-cli](https://github.com/martindsouza/docker-hexo):
```
docker pull moanan/hexo_maupassant:mode14.16.0-alpine3.10
# yes, mode is a typo
```
where I specified the working node version:
```
FROM node:14.16.0-alpine3.10 as hexo-base
```
and added Maupassant dependencies:
```
npm install hexo-renderer-pug --save && \
 npm install hexo-renderer-sass --save && \
```

## Usage
Add the following to create (and persist) alias to `~/.bash_profile` (or `~/.zshrc` if using zsh):

```bash
alias hexo_m="docker run -it --rm \
  -v `pwd`:/opt/node_app/app \
  -p 4000:4000 \
  -p 3000:3000 \
  hexo_maupassant"
```

You can then run all hexo (now `hexo_m`) commands as you normally would with the caveat that the container ***can only see the current directory or any of its children***. i.e. don't run commands like `hexo init ../../my-new-blog`

Port | Desc
--- | ---
`3000` | Optional: For [`hexo-browsersync`](https://github.com/hexojs/hexo-browsersync) to work
`4000` | Draft `localhost` port


### MacOS Users

#### `node_modules`

Since Docker for Mac uses a VM behind the scenes, referencing the current directory's `node_modules` folder may be very slow. If that's the case, delete the hosts (i.e. your laptop's) `node_modules` folder and the container has its own copy which will be faster.

#### `git`

I have tried to use the [`hexo-deployer-git`](https://github.com/hexojs/hexo-deployer-git) in the VM but struggled with the container accessing my `~.ssh` folder (from a permissions perspective) to access the private key for Github login. To work around this can manually deploy with following steps:

** Setup ** (do this once)

```bash
git clone --single-branch --branch gh-pages <git-repo> .deploy_git

# Example
git clone --single-branch --branch gh-pages git@github.com:moanan/recipe.git .deploy_git

```

** Deploy **

```bash
hexo clean
hexo generate
rm -rf .deploy_git/*
cp -r public/* .deploy_git/
cd .deploy_git
git add *
git commit -m "Site updated manually"
git push
```
