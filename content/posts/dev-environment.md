+++
title = "Personal Development Environment"
date = "2020-06-25"
+++

![tmux-example](/images/tmux-example.png)

## Tools

Here's a list of things I need to to and the tools I use to do it

- Development Environment
  - AI
    - https://gist.github.com/2bb4bb6d7a6abaa07cebc7c04d1cafa5
    - [![asciicast](https://asciinema.org/a/674501.svg)](https://asciinema.org/a/674501)
  - Editor
    - [vim](https://www.vim.org/)
      - [Learn VIM](https://vim-adventures.com/)
  - Terminal Multiplexer
    - Let's you turn a single shell session into a bunch of shells, you can put
      them side by side, have tabs, label them, etc.
    - [tmux](https://github.com/tmux/tmux/wiki)
      - [Cheatsheet by Mohamed Hassan](https://gist.github.com/MohamedAlaa/2961058)
      - [The Tao of tmux by Tony Narlock](https://github.com/git-pull/tao-of-tmux)
  - Testing
    - [nodemon](https://nodemon.io/)
      - This will change your life. Its a command line utility that will re-run
        a command when files with certain extensions change. You can use it to
        re-compile (if applicable) and re-test whatever your working on every
        time you save a file! This will also introduce you to a massive reward
        feedback loop like none other. Have fun.
      - `nodemon -e py --exec 'clear; python script.py; test 1'`
- Chat
  - IRC
    - [weechat](https://weechat.org/)
      - [Quickstart](https://weechat.org/files/doc/stable/weechat_quickstart.en.html)
  - Web based
    - [Gitter]()
      - Good for GitHub communities
    - [discord](https://discord.com/)
      - Has peer to peer (webrtc based) video and voice chat
- Documentation
  - Videos and Gifs
    - [Open Broadcaster Software](https://obsproject.com)
      - I use this to record my screen, it's cross platform and open source
    - [ffmpeg](https://ffmpeg.org/)
      - I use this to convert videos I've recorded into gifs to use in README's
        and documentation.
- Meetings
  - [Google Meet](https://meet.google.com)
- Email
  - [mutt](https://weechat.org/)
    - [Quickstart](https://weechat.org/files/doc/stable/weechat_quickstart.en.html)

## dotfiles

Dotfiles is a term for configuration files. The name comes from their usually
being prefixed with a `.`. This is because on UNIX operating systems, files
prefixed with a `.` are *hidden*.

Here are mine: http://github.com/pdxjohnny/dotfiles

## Linux distros

These days I stick mostly to the big three desktop distros, debian, ubuntu, and
fedora. I used to run Solus on my personal, it's out of commission at the moment
and I am waiting until Serpent OS releases `.iso` images to bring it back up
again.

I started using Fedora when I was doing the CR0/4 KVM work. Kristen and Rick told
me that Fedora guarantees that you'll be able to run the
[upstream kernel](./linux-kernel/#terminology). Unless you want to mess with
building `.deb` files on ubuntu so that you have all their configs and patches,
which sounds not fun to me, it's probably best to stick to a well supported distro
that you can just do a clean build of the linux tree and run the kernel on. But
I've been busy trying to get CI ironed out for that project so I haven't gotten
back to running on fedora in a while.

## Reasoning

I forced myself to get used to a primarily terminal based development
environment because I constantly find myself on systems where I have either only
ssh or serial access.

Let me give you some scenarios where this might happen to you (and therefore its
nice to already be used to a terminal based workflow).

- You run Linux, your desktop crashed or froze
  - Press Ctrl-Alt-[F1 through F12] and you'll be presented with a terminal!
  - You're on a deadline (because when else would your desktop crash or freeze)
    or your not sure what will happen / you will loose if you reboot the
    machine.
  - No worries! Just `tmux attach` and you're right back where you were! Commit
    your work, push it and reboot without worry.
- The production server broke
  - You're probably only going to have `ssh` access to this thing. You going to
    want to read the logs and be able to poke around the file system with your
    editor of choice (mine is `vim`) and restart stuff to see what's wrong.
    (Chances are the server won't have `tmux`, but it might have
    [`screen`](https://linuxize.com/post/how-to-use-linux-screen/) which is
    similar so you'll be familiar with the concept).
- You started a VM from the command line using QEMU (likely with
  `-nographic -append "console=ttyS0"`) so you have a terminal into the guest
  Linux machine but it's not a full ssh session, you're likely limited to 80
  columns here. You'll be glad you stuck to 80 columns now that's all you can
  see without whacky "scrolling".

### Alt-Tab

The top reason why I do everything in the terminal though is Alt-Tab.

If you keep all your chat, email, open code repos in tabs within `tmux` then
open one browser window for all the docs, web based chats, etc. The beauty of
this is you only ever have two windows open. Which means whenever you hit
Alt-Tab you never have to guess where your going to end up! This is going to
save you insane amounts of time in the long run. Ditch the clutter. Hakuna
Matata.


## Notes

TAKE NOTES. NOTES ARE A SUPERPOWER. WHAT IS NOT WRITTEN DOWN IS LOST.
WHAT IS NOT ORGANIZED IS LOST. THOSE WHO WANDER ARE LOST WITHOUT NOTES!

### Format

> https://github.com/intel/dffml/discussions/1369#discussioncomment-2675744

```
# YYYY-MM-DD Title or Meeting Name

- NTT
  - Ensure each of these are brought up
- Topic 1
  - Notes go here organized by topic
- TODO
  - Generic future actions
- Next Steps
  - ARs
    - Name will ...
  - Generic near term future actions
```

#### Shorthand

NTT - Need To Talk about these things in the meeting, checklist of things
that must be addressed or brought up

## New dev box bring up

This is me attempting to collect all the things I do when I set up a new
machine for development.

I should just put this in a container and then extract the layer over the
homedir via curl + tar.


```bash
NAME_FIRST_LAST="John Andersen"
EMAIL=johnandersenpdx@gmail.com

# Create homedir venv
mkdir -p ~/.local/.venv
python -m venv ~/.local/.venv || python3 -m venv ~/.local/.venv
. ~/.local/.venv/bin/activate
python -m pip install -U pip setuptools wheel
python -m pip install -U keyring keyrings-alt

# Install GitHub CLI
# https://github.com/cli/cli/blob/trunk/docs/install_linux.md
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update
sudo apt install -y gh

# Log into GitHub
gh auth login

# Configure git
git config --global user.name $NAME_FIRST_LAST
git config --global user.email $EMAIL
git config --global pull.rebase true

# Clone dotfiles
git clone https://github.com/pdxjohnny/dotfiles ~/.dotfiles
cd ~/.dotfiles
./install.sh
echo -e 'if [ -f ~/.pdxjohnnyrc ]; then\n. ~/.pdxjohnnyrc\nfi' | tee "${HOME}/.bashrc"
history -a
exec bash
dotfiles_branch=$(hostname)-$(date "+%4Y-%m-%d-%H-%M")
git checkout -b $dotfiles_branch
git push --set-upstream origin $dotfiles_branch

# Modify dotfiles for host
# Open tmux and copy based the errors into invalid
# cat > /tmp/invalid <<'EOF'
# EOF
# grep -vE $(invalid=$(< /tmp/invalid); invalid=${invalid/$'\n'/ }; echo $invalid | sed -e 's/ /|/g' < ~/.tmux.conf) \
#   | (temp_conf=$(mktemp); cat > $temp_conf \
#      && truncate  --no-create -s 0 ~/.tmux.conf \
#      && tee -a  ~/.tmux.conf < $temp_conf)
sed -i "s/Dot Files/Dot Files: $dotfiles_branch/g" README.md
# Save modifications
cd ~/.dotfiles
git commit -sam "Initial auto-tailor for $(hostname)"
git push

# Install extras
pip install --force-reinstall -U https://github.com/ytdl-org/youtube-dl/archive/refs/heads/master.tar.gz#egg=youtube_dl
```

### TODO

- dataflows

  - tool install per-distro

  - login tasks (`~./bashrc` run something with `&` to background)

    - Check if everything in homedir is either tracked in git or explicitly ignored

      - notify-send or equivalent, easy bash pipe allowed triage
