---
layout: post
title:  Using pass in a team
date:   2016-12-05 20:05:15 +0100
categories: pass password-manager
---

The “standard unix password manager” [pass](https://www.passwordstore.org/ "the standard unix password manager"){:target='blank'} is a great tool if you want to have full control over your password store and want it to be accessible by various systems. If you want to share the password store across your team, pass requires some more steps to set it up. This article describes how to organize and encrypt such a shared password store.

#### Requirements

In order to run the commands below you’ll need to have the `pass` and `gpg` tools installed. You will also need your own private/public key and your teammate’s public keys set up. For more information check the [pass](https://www.passwordstore.org/ "the standard unix password manager"){:target='blank'} and [gpg](https://www.gnupg.org/documentation/guides.html "GnuPG User Guides"){:target='blank'} documentation.

#### Let’s get started

Assume a company called _Acme_ with team members:

 * Alice (alice@acme.org)
 * Jane (jane@acme.org)
 * Bob (bob@acme.org)

Alice will manage and encrypt the password store in a way that it’s decipherable by Jane and Bob.

Let’s start with initializing an empty password store:

{% highlight bash %}
$ pass init alice@acme.org
{% endhighlight %}

This will create a `.password-store` folder in the user’s home directory. All passwords can be organized in files & folders inside of the `.password-store` folder. A shared password store should be organized in it’s own subfolder which can be done using the -p option:

{% highlight bash %}
$ pass init -p acme alice@acme.org
{% endhighlight %}

This command will create the subfolder `acme` in the `.password-store` folder.

Let’s create a sample password:

{% highlight bash %}
$ pass generate acme/sharedpass 23
{% endhighlight %}

This will generate a new password in the `acme` subfolder. By now, only Alice can decrypt the password.

#### Using multiple gpg ids

The `pass init -p acme` command creates a `.gpg-id` file in the `acme` folder. This file includes the public gpg ids which will be used to encrypt the password files. In order to share a password store with your teammates you need to specify their gpg ids in the `.gpg-id` file and re-encrypt the passwords.

Open the `~/.password-store/acme/.gpg-id` file in an editor and add the public gpg ids:

{% highlight bash %}
alice@acme.org
jane@acme.org
bob@acme.org
{% endhighlight %}

Now, it’s important to (locally) sign the keys of your teammates using `gpg`:

{% highlight bash %}
$ gpg --edit-key jane@acme.org

gpg> lsign
gpg> y
gpg> save
{% endhighlight %}

Repeat the steps for every entry in the `.gpg-id` file. Of course, only sign the keys if you fully trust them.

After this, re-initialize the shared password store with:

{% highlight bash %}
$ pass init -p acme $(cat ~/.password-store/acme/.gpg-id)
{% endhighlight %}

This will re-encrypt the passwords in the `acme` subfolder using the gpg ids specified in `~/.password-store/acme/.gpg-id`.

Now you are ready to share the encrypted password store with your teammates. Alice, Jane and Bob can decrypt the password with:

{% highlight bash %}
$ pass acme/sharedpass
{% endhighlight %}

#### Sharing options

Since the password store is organized using plain files & folders, you can just copy the `acme` folder to a device and share it with your team. They would need to have pass installed with an (empty) password store and copy the `acme` folder to their “.password-store” folder.

Another way of sharing the password store is by using `git`. Alice, who created the `acme` subfolder, can just add this folder to git and push it to a remote server.

{% highlight bash %}
$ cd ~/.password-store/acme
$ git init
$ git add .
$ git commit -m "Add acme password store"
$ # Set up remote
$ git push
{% endhighlight %}

The teammates can then clone the repo to their `.password-store`.

{% highlight bash %}
$ cd ~/.password-store
$ git clone https://git.example.org/acme
{% endhighlight %}

#### Updating the password store

Whenever you add a new password to the password store, you need to re-initialize (and re-encrypt) the password store with:

{% highlight bash %}
$ pass init -p acme $(cat ~/.password-store/acme/.gpg-id)
{% endhighlight %}

The command also needs to be run if you edit the `.gpg-id` file (adding or removing users).

If you use git, then commit and push the changes so that your teammates can update the password store.

#### Summary

In order to create a shared password store you need to:

* Create a subfolder in your password store:  
  `pass init -p acme alice@acme.org`

* Add your teammate’s public gpg ids to the `acme/.gpg-id` file

* Locally sign the public keys  
  `gpg --edit-key jane@acme.org`
  `gpg> lsign`

* Re-encrypt the password store with:  
  `pass init -p acme $(cat ~/.password-store/acme/.gpg-id)`
