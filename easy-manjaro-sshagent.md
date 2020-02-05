# Setting up a SSH agent

I have tried a lot of SSH agents and I've come to the conclusion that `gnome-keyring` is the one that provides the best experience. It provides the same experience as the one that comes by default in Ubuntu 18.04 LTS, which is perfect for me.

It asks the password when you are going to use it, not when you open a terminal, or when you login, or other nonsense.

Others require you to create systemd services manually, others require to touch a lot of files, and most of the times you get a bad experience anyway.
For future reference, I tried the following configurations:

- Using `ssh-agent` through `.bashrc` and `.zshrc`. Not a good experience.

- Using `ssh-agent` through a systemd user unit, and adding a variable to `.pam_environment`. Didn't even work.

- Using `ssh-agent` through a systemd user unit, and adding some variables to `~/.config/plasma-workspace/env/ssh-agent.sh`. Worked, stayed with this for a while. Some programs didn't like this and required me to open a terminal and do something with ssh beforehand. This is what I did in that file:

    ```bash
    #!/bin/sh

    export SSH_ASKPASS='/usr/bin/ksshaskpass'
    export SSH_AUTH_SOCK="${XDG_RUNTIME_DIR}/ssh-agent.socket"
    ```

- Using `keychain` and `x11-ssh-askpass`. Don't. Just don't. It's ugly as fuck.

- Using `kwallet` with `ssh-agent` running on a systemd user unit. Very bad experience.

- Others that I don't remember and saw in sites other than Arch's Wiki. Were either hacky or didn't provide a good experience.

You can see all the possible configurations of SSH Agents in the [Arch Wiki Page for SSH agents](https://wiki.archlinux.org/index.php/SSH_keys#SSH_agents)

## Configuring gnome-keyring

Do not let the name deceive you, you can use it under any desktop environment or window manager. It doesn't have many GNOME dependencies so don't worry about installing it.

### Installing

Install both `gnome-keyring` and `seahorse`. The latter is the GUI for the keyring.

```bash
sudo pacman -S gnome-keyring libsecret seahorse
```

### Configuring

I only had to do two things to make it work, one is copying a snippet of bash script in my `.bash_profile` and my `.zshenv` files, so that when I use ssh through the terminal they know of the SSH agent instance. The other is adding a small configuration to `~/.ssh/config` so that keys are added to the agent automatically.

#### First configuration

Add this to both of these files (`.bash_profile` and `.zshenv`, the latter is if you use zsh, of course)

```bash
if [ -n "$DESKTOP_SESSION" ]; then
    eval $(gnome-keyring-daemon --start)
    export SSH_AUTH_SOCK
fi
```

If you use `fish` shell, also add this to `~/.config/fish/config.fish`

```bash
if test -n "$DESKTOP_SESSION"
    set (gnome-keyring-daemon --start | string split "=")
end
```

Obviously, make sure you remove any configuration of the same kind (that modify `SSH_AUTH_SOCK`) you have done previously, otherwise there might be conflicts between SSH agents.

#### Second configuration

Add the following to `~/.ssh/config`

```text
Host *
    AddKeysToAgent yes
    UseKeychain yes
    IdentityFile ~/.ssh/id_rsa
```

You can add these changes to the files in `/etc/skel` and the next time you create a user, they will have these files configured by default.

### Testing

- Just reboot, or logout and log back in, which should also work.

- Open a terminal and do `ssh git@github.com`, a window should pop up asking for your passphrase.

- Close the previous terminal and do it again, if it does not ask for the passphrase again, you are set!

- You might also want to check apps like JetBrains IDEs, Visual Studio Code, or other programs that use Git behind the hood work properly. In my experience, they work flawlessly, asking for the passphrase only when needed.
