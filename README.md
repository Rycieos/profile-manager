# profile-manager

profile-manager syncs git projects to hosts, keeping all profiles
up-to-date. It is designed with bootstrapping and auto-running in mind.

While profile-manager has many features, it was designed to not "do everything
for you". Bootstrapping, auto-running, and plugins are all up to the user. This
is because, while searching for the perfect dot-file manager, I found many that
did what I wanted, but fell short because they forced a certain usage or file
setup that didn't work for me. This project aims to solve those problems.

## Options
 -c|--config [config-file]  
    Load the config file specified instead of the default,
    ~/.config/profile-manager/config  
 -q|--quiet  
    Don't print messages as plugins are installed and updated.  
 -f|--force  
    Force re-linking files even if the plugin wasn't updated.  

## Bootstrapping

Bootstrapping allows for one command setups of new machines. A bootstrap would
install all wanted plugins and setup the auto-running to keep everything
up-to-date.

As mentioned above, profile-manager doesn't come with any built-in boostrapping
power, just the ability to be bootstrapped easily.

Most likely, you would have all of your dot-files and other projects on a git
hosting service like GitHub. This makes bootstrapping very easy, as you can
download the files from those services.

Since profile-manager is idempotent, all you need to do is run it with your
config file. A simple bootstrapping example is this:
```
bash <(curl -s https://raw.githubusercontent.com/Rycieos/profile-manager/master/profile-manager) -c <(curl -s https://raw.githubusercontent.com/Rycieos/dot-files/master/profile-manager/config)
```
This loads the program from the web, loading the config file into it. You can
replace my config with yours.

An even simpler version is to put the above line into a file, as seen in the
bootstrap.sh file in this repo, and put that file somewhere with a short url.
I have done this with my bootstrap script, which means I can just run:
```
bash <(curl https://www.rycieos.com/pu)
```
to install on a machine. This line I can memorize, allowing me to have my
configs up and running less than a minute after first touching the keyboard.

## Auto-updating

There are two ways I will mention to auto-run this, but the options are
endless.

### bashrc
This is the way I do it. This means every login will start an update.
Add a few lines like this to your bashrc:
```
if [[ -x ~/bin/profile-manager ]]; then
  profile-manager -q &
fi
```
This runs it quiet, so it doesn't bug whatever you are trying to do, and in a
sub-shell, so it doesn't slow your login down.

### cron
You can add a line to cron to run it every so often. Adding the --quiet flag
will make cron only mail you if anything goes wrong. A line like this:
```
0 * * * * /home/user/bin/profile-manager --quiet
```
should do the trick; set the timing how you like.

## Config

The config is where you specify what plugins, or repos, to install and how to
do it. Here is a full-featured example:
```
voom() {
  repo_url="https://github.com/Rycieos/voom.git"
  files=(
    ["voom"]="~/bin/voom"
  )
  depends=(
    "vim"
  )
  voom_post_install() {
    voom
  }
  voom_post_update() {
    voom update
  }
}
```

Every repo must be its own function.

Every repo must have `repo_url` set. This is the address used for the initial
clone. Don't use the ssh protocol unless you will be setting up your ssh keys
on every machine before running your bootstrap.

To be useful, every repo must have lines in the `files` array. The first part
of the line is the filename of the file you want to have linked, relative to
the repo root. The second part is the location that you want it to be linked
to. For example, the "voom" file above is a script in the repo root. I want to
be able to run it when I want, so I link it to "~/bin/voom". The link will
force replace files, so make sure you don't have any naming conflicts.

Optionally, the `depends` array can be set. Each value in the array is a
command that is searched for. If any are not found on the machine, the repo
will not be installed.

There are post install and update hooks, that will run after their respective
actions. The must be prefixed with the name of the plugin, like in the example
above.

See [my config](https://github.com/Rycieos/dot-files/blob/master/profile-manager/config) for more examples.

To be most useful, you will want to add profile-manager itself to the config,
along with your config for profile manager. For example:
```
profile-manager() {
  repo_url="https://github.com/Rycieos/profile-manager.git"
  files=(
    ["profile-manager"]="~/bin/profile-manager"
  )
}

dot-files() {
  repo_url="https://github.com/Rycieos/dot-files.git"
  files=(
    ["profile-manager/config"]="~/.config/profile-manager/config"
  )
}
```

## vim

profile-manager could be used to sync vim plugins. For each plugin, add a line
like such:
```
["."]="~/.vim/bundle/[plugin-name]"
```
In this way, it works similarly to [voom](https://github.com/airblade/voom).

If you run profile-manager automatically, then it would update the plugins
automatically as well.

