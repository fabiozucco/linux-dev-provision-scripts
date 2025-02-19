# Linux Developer Provision Scripts

I did this because everytime I reset my PC I forget to install something. This document prevents it from happening again.

Assuming that it ships with a clean Debian 9, this is the roadmap to a working development environment.

##### sudo

Debian doesn't come with sudo by default like Ubuntu, we gonna need it soon.

```shell
su -
apt-get install -y sudo
adduser USER sudo
# restart
```

##### additional packages

Packages that sooner or later will be needed in this setup.

```shell
sudo apt-get -fy install curl wget git vim zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev software-properties-common libffi-dev apt-transport-https libgdbm-dev libncurses5-dev automake libtool bison libffi-dev dirmngr libgmp-dev pkg-config libgmp-dev libpq-dev ca-certificates gnupg2
```

##### zsh

My favorite shell.

Plugins:

`plugins=(heroku rake git ruby gem ng rvm docker history node npm systemd ssh-agent sudo bundler command-not-found zsh-syntax-highlighting history-substring-search gnu-utils github yarn common-aliases debian)`

```shell
cd $HOME
sudo apt-get -y install zsh
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
# prompts password
wget https://github.com/powerline/powerline/raw/develop/font/PowerlineSymbols.otf
wget https://github.com/powerline/powerline/raw/develop/font/10-powerline-symbols.conf
mkdir -p $HOME/.fonts/ $HOME/.config/fontconfig/conf.d
mv PowerlineSymbols.otf ~/.fonts/
fc-cache -vf ~/.fonts/
mv 10-powerline-symbols.conf ~/.config/fontconfig/conf.d/
vim ~/.zshrc
```

##### Proxy Settings

Depends if I'm under a proxied network.

```shell
export DEFAULT_PROXY=user:pass@10.0.0.1:1000

export HTTP_PROXY=http://$DEFAULT_PROXY
# export HTTPS_PROXY=https://$DEFAULT_PROXY
export FTP_PROXY=$HTTP_PROXY
export NO_PROXY="localhost, 127.0.0.1"

export http_proxy=$HTTP_PROXY
# export https_proxy=$HTTPS_PROXY
export ftp_proxy=$FTP_PROXY
export no_proxy=$NO_PROXY
```

##### Node Version Manager

NodeJS and npm via nvm.

```shell
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.4/install.sh | bash
source $HOME/.nvm/nvm.sh
nvm ls-remote && nvm install --lts
sudo ln -s $(which node) /usr/bin/node
sudo ln -s $(which npm) /usr/bin/npm
```

##### Yarn

```shell
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
sudo apt-get update && sudo apt-get -fy install yarn
```

Setup yarn global bin path, otherwise global packages won't work.

```shell
# $HOME/.zshrc
export YARN_GLOBAL_PATH=$(yarn global bin)
export PATH=$PATH:$YARN_GLOBAL_PATH
```

##### Ruby on Rails
```shell
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3
curl -sSL https://get.rvm.io | bash -s stable
source ~/.rvm/scripts/rvm
rvm install 2.4.2
rvm use 2.4.2 --default
ruby -v
rm -f $HOME/.gemrc && touch $HOME/.gemrc && echo "gem: --no-ri --no-rdoc" >> $HOME/.gemrc
rm -f $HOME/.irbrc && touch $HOME/.irbrc && echo "require 'awesome_print'\nAwesomePrint.irb!" >> $HOME/.irbrc
gem install bundler rails
```

Initial commands for normal and API only apps:

```
# for full apps
rails new myapp \
  --skip-action-cable \   # skip rails web sockets, cuz it suck balls
  --skip-coffee \         # skip coffeescript, cuz ES5+ is better
  --skip-test \           # skip default test suite, cuz rspec is better
  --skip-system-test \    # skip system tests, cuz capybara is better
  --database=postgresql \ # set a database compatible with most of cloud services
  --webpack=react         # set webpack for react, angular if it's a SPA
# or for apis
rails new myapp \
  --api \
  --skip-test \
  --skip-system-test \
  --skip-action-cable \
  --database=postgresql
```

Maximizes file system inotify watchers for gem listen.

```shell
echo fs.inotify.max_user_watches=524288 | sudo tee /etc/sysctl.d/40-max-user-watches.conf && sudo sysctl --system
```

Adds ruby version to the zsh.

```shell
# $HOME/.zshrc
RPROMPT="\$(~/.rvm/bin/rvm-prompt s i v g)%{$fg[yellow]%}[%*]"
```

##### Postgresql

```shell
sudo apt-get update && sudo apt-get -fy install postgresql postgresql-common postgresql-client
sudo -u postgres createuser leonardo -s
sudo -u postgres psql
\password leonardo
```

##### Git
```shell
git config --global color.ui true
git config --global user.name "Leonardo Falk"
git config --global user.email "wontputmyemailhere@"
ssh-keygen -t rsa -b 4096 -C "wontputmyemailhere@"
cat ~/.ssh/id_rsa.pub # and copy to github
ssh -T git@github.com # test connection
```

##### Docker

```shell
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian stretch stable"
sudo apt-get update && sudo apt-get install -fy docker-ce
sudo usermod -aG docker $USER # skip sudo to run docker
```

##### Docker Compose

```shell
sudo curl -L https://github.com/docker/compose/releases/download/1.19.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

##### VirtualBox

```shell
sudo add-apt-repository -y "deb http://download.virtualbox.org/virtualbox/debian stretch contrib"
curl -L https://www.virtualbox.org/download/oracle_vbox_2016.asc | sudo apt-key add -
sudo apt-get update && sudo apt-get install -yf virtualbox-5.1
```
