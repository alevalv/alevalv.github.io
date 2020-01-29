---
title: KeepassXC and secret service, a small walk-through.
layout: post
---

[KeepassXC](https://keepassxc.org/) is an offline password manager, similar to lastpass, it will allow us to store and generate our passwords safely. These passwords are encrypted and unlocked with a master password. As of release 2.5.0, KeepassXC implements the freedesktop.org secret service protocol (you can find its definition [here](https://specifications.freedesktop.org/secret-service/latest/)), this is a [D-Bus](https://en.wikipedia.org/wiki/D-Bus) API that will allow applications to store and read secrets. A lot of things use it, for example, chrome or firefox will to store their secret data in it (cookies, saved passwords, etc). These secrets can also be accessed with `secret-tool` command, provided by `libsecret`, allowing you to create scripts that can pull passwords automatically if the database is open.

## KeepassXC Configuration

To be able to query your local KeepassXC database using the secret service, you need to enable it on the general settings (Tools > Settings):

![KeepassXC secret service settings](/assets/secret-service-keepassxc/keepassxc_settings_secret_service.png)

You should create a new group/folder to be shared with the secret service, that way we can keep a small set of credentials that are visible with secret service, since this API does not have any type of authentication or verification, any software running in your pc have access to data stored in it.

Having this, go to database settings (Database > Database Settings) and set this group to be available via secret service:

![KeepassXC database secret service settings](/assets/secret-service-keepassxc/database_settings_secret_service.png)

## Using libsecret secret-tool

`secret-tool` provides a set of commands to store, delete and query secrets from the existing secret service provider. To obtain a password from an existing key/value, you can use this command:

```sh
secret-tool lookup <key> <value>
```

In this case `<key>` is a value that is defined in the Advanced > Additional Attributes of an entry. When you query the secret service, it will check on all entries available for the key-value pair. For example, in this entry that my database has, I added a new `account` attribute with a **unique** value:

![Entry with additional attribute](/assets/secret-service-keepassxc/entry_attribute_key_value.png)

Then, I can query this entry password with:

```sh
secret-tool lookup account KP2A_URL_4
```

If secret tool cannot find your given key/value pair, it will return an empty string and will exit with code value 1.

This is especially useful when you run `secret-tool` inside a script, since it will make the script fail (if you configure your bash script with `set -e`, to fail on error 👌).

## Unlocking your GPG key with KeepassXC automatically.

If you use a [GPG key](https://en.wikipedia.org/wiki/Pretty_Good_Privacy) (which are very useful but kinda complicated), you can also automate/skip the password input with secret service.

[GnuPG](https://gnupg.org/) uses [pinentry](https://gnupg.org/related_software/pinentry/index.html) to obtain the password of your GPG key. Pinentry will first try to obtain the password from the secret service, and if it does not find it, then it will show you the password input prompt, like this one:

![pinentry-qt](/assets/secret-service-keepassxc/pinentry-password.png)

You can avoid this prompt entirely if you add an entry in your database that can be queried, To do this, follow these steps:

1. Change your pinentry program to `pinentry-gtk-2` (you can also use `pinentry-gnome3`, if you use GNOME), editing `~/.gnupg/gpg-agent.conf`:

```
pinentry-program /usr/bin/pinentry-gtk-2
```

We need to do this since this pinentry implementation can store the password in the secret service.

2. Having your KeepassXC database open and with secret service active, try to unlock the key using gpg directly, for example:

```sh
echo asdf | gpg --armor --sign
```

Will try to encrypt asdf, and it will display the pinentry prompt:

![pinentry-gtk-2](/assets/secret-service-keepassxc/pinentry-gtk-2.png)

Copy your GPG key password here, and **do not forget to check** the **Save in password manager** option. This will create a new entry in your secret service folder in KeepassXC with the GPG password. So next time that you need the gpg key, it will be retrieved from KeepassXC (this does not work everywhere, e.g. [Kleopatra](https://kde.org/applications/utilities/org.kde.kleopatra) will still need the password input).

Finally, you can remove/undo the change in `~/.gnupg/gpg-agent.conf` and reload your gpg-agent:

```sh
gpg-connect-agent reloadagent /bye
```

Now try again to unlock your GPG key and it should not ask for password.

That is all I have to share about the secret service and KeepassXC, I'm sure that there is more that can be done with it. Thank you for reading 😸.
