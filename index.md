---
layout: default
title: Invoker - A Process Manager
---

## What is Invoker?
Invoker is a utility belt for managing processes in development environment.
Use it for managing multiple processes with ease.

Use it for developing web applications on different local domains without
`/etc/hosts` hacks.

<iframe width="640" height="480" src="http://www.youtube.com/embed/uu6NZrYvSQ8" class="demo-video" frameborder="0" allowfullscreen></iframe>

<div class="supported-runtimes">
  <div class="logos">
    <img src="images/python-grey.png">
    <img src="images/node-grey.png">
    <img src="images/ruby-grey.png">
  </div>
  <div class="in-double-line"></div>
  <div class="runtime-message">
    Works with <em>Python</em>, <em>Node</em> or any <em>Ruby</em> application.
  </div>
</div>

<a name="usage"></a>
## How to use it?

First we need to install invoker gem to get command line utility called invoker, we can do that via:

{% highlight bash %}
~> gem install invoker
{% endhighlight %}

Currently it only works with Ruby 1.9.3 and 2.0.

You need to start by creating a `ini` file which will define processes you want to manage using invoker. An example `ini` file is included in the repo.

{% highlight bash %}
[rails]
directory = /home/gnufied/god_particle
command = bundle exec rails s -p 5000

[dj]
directory = /home/gnufied/god_particle
command = bundle exec ruby script/delayed_job

[events]
directory = /home/gnufied/god_particle
command = bundle exec ruby script/event_server
{% endhighlight %}

After that you can start process manager via:

{% highlight bash %}
~> invoker start invoker.ini
{% endhighlight %}

Above command will start all your processes in one terminal with their stdout/stderr merged and labelled. You
can also start `Invoker` by daeomonizing it, via:

{% highlight bash %}
~> invoker start invoker.ini -d
{% endhighlight %}

<a name="tld"></a>
## .dev TLD support for local apps

You can access http services managed by invoker via `command_label.dev` domain locally.

To make it work though, you need to run following command, just once from anywhere:

{% highlight bash %}
~> sudo invoker setup # read below if you are migrating from Pow
{% endhighlight %}

If you decide to remove Invoker, you can remove things installed by Invoker using command

{% highlight bash %}
~> sudo invoker uninstall
{% endhighlight %}

Now because invoker is making your app server available on a domain. It requires
control over port(or needs to know the port) on which your applications will be listening. This can be simply done by replacing specific port number in `ini` file with `$PORT` or by specifying
a port key. For example:

{% highlight bash %}
[terminal]
directory = /home/gnufied/ayr-terminal
port = 3000
command = node app

[cms]
directory = /home/gnufied/mezzainine
command = python manage.py runserver $PORT --noreload

[typo]
directory = /home/gnufied/typo
command = bundle exec rails s -p $PORT
{% endhighlight %}

Now these services can be accessed via `http://terminal.dev` , `http://cms.dev`
`http://typo.dev`. You can also access them via wildcard subdomains such as `*.*.dev`.

You can also access any external http process via `.dev` DNS by running
following command:

{% highlight bash %}
~> invoker add_http wordpress 8080
{% endhighlight %}

Above command will make wordpress available on `wordpress.dev` even if
wordpress was original not started by Invoker. You can access any randomly
started process via Invoker like this.

<a name="https_support"> </a>
## Https support

Invoker uses a self-signed certificate to make all your web applications available via
`https` as well. You absolutely don't have to do anything. Access your webapps on `https://rails.dev`
and enjoy!


<a name="procfile"></a>
## Procfile support

Since version `1.0.3` Invoker has added support
for `Procfile`. Now If you already have a `Procfile` you need not even create a `ini`
file for using Invoker.

The only thing to remember is, your `Procfile`
must have `$PORT` in command - for `.dev` domain feature to work

{% highlight bash %}
rails: cd $HOME/rails_app && bundle exec rails s -p $PORT
cms: cd $HOME/cms && python manage.py runserver $PORT
{% endhighlight %}

<a name="process"></a>
## Process managment

Additionally Invoker allows you to manage individual processes. You can start/stop/restart
different processes managed by invoker without affecting others.

{% highlight bash %}
# Will try to stop running delayed job by sending SIGINT to the process
~> invoker remove dj

# If Process can't be killed by SIGINT send a custom signal
~> invoker remove dj -s 9

# add and start running
~> invoker add dj

# List currently running processes managed by invoker
~> invoker list

# Restart process given by command Label
~> invoker reload dj

# Restart process given by command label using specific signal for killing
~> invoker reload dj -s 9

# tail logs of specified process. This is specially useful if you started
# invoker daemonized or you want to watch logs of just one process discarding
# others.
~> invoker tail dj

{% endhighlight %}

You can also enable OSX notifications for crashed processes by installing terminal-notifier gem. It is not a
dependency, but can be useful if something crashed and you weren't paying attention.

<a name="pow"></a>
## Migrating from Pow

If you are migrating from Pow then first step before running `invoker setup` is to uninstall Pow:

{% highlight bash %}
curl get.pow.cx/uninstall.sh | sh
{% endhighlight %}

After that you should run `invoker setup`
as usual. If for some reason you can't uninstall Pow via above
command (in case you installed Pow using homebrew), then you
should remove Pow daemon manually.

After running `invoker setup` you will
have to <em>switch off wi-fi and then switch it on </em> for
resettng OSX network configuration.

<a name="versions"></a>
## Using Invoker with rbenv or rvm

The way `rbenv` and `rvm` work sometimes creates problems when you are trying to use a process supervisor like invoker. There are couple of things to keep in mind,
If you are running invoker with Ruby version x, but your application requires Ruby version Y:

When using rbenv and zsh, remember that `.zshrc`
is not read for commands run via `zsh -c`. So first
add:

{% highlight bash %}
~> cat > ~/.zshenv
eval "$(rbenv init -)"
{% endhighlight %}

and then run it using:

{% highlight bash %}
command = RBENV_VERSION=2.0.0-p0 zsh -c "bundle exec rails s"
{% endhighlight %}

Unless version of Ruby using which you are running invoker command and version of Ruby you are using in the application is same, you almost always will want to use
`zsh -c` or `bash -c`. RVM in particular requires a login shell and hence sometimes you may have to use `bash -lc`. For example:

{% highlight bash %}
command = bash -lc "rvm 2.0.0-p0 do bundle exec rails s"
{% endhighlight %}

<a name="faq"></a>

## Credits

Invoker is hugely inspired by <a href="http://pow.cx">pow</a> and <a href="https://github.com/ddollar/foreman">Foreman</a>.
It stands on the shoulder of awesome work done by folks who created Pow & Foreman. The Codemancers team is immensely grateful to them.