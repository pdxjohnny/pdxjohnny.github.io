+++
date = 2020-07-07T19:00:00Z
lastmod = 2020-07-07T19:00:00Z
title = "Time Saving Tricks and Hacks"
subtitle = ""
+++

## dos2unix in Python

Remove carriage returns from CRLF (carriage return, line feed: `\r\n`).

```console
$ python -c 'import pathlib, sys; p = pathlib.Path(sys.argv[-1]); p.write_bytes(p.read_bytes().replace(b"\r", b""))' file.txt
```

## Get list of available versions

```console
$ python -m pip install dffml==
```

## Download file with Python from command line with progress

```console
$ python3 -c 'import sys, functools, urllib.request; print(urllib.request.urlretrieve(sys.argv[-1], reporthook=lambda n, c, t: print(f"{round(((n*c)/t) * 100, 2)}%", end="\r", file=sys.stderr))[0])' https://storage.googleapis.com/laurencemoroney-blog.appspot.com/rps.zip
```

[![asciicast](https://asciinema.org/a/357044.svg)](https://asciinema.org/a/357044)

## Display only blocks of text with certain text in them

Use grep to displays blocks of text. Only display blocks with certain text
inside them.

```console
$ git grep -A 25 -E 'dffml train|dffml accuracy|dffml predict' | python -c 'import sys; print("--".join([i for i in sys.stdin.read().split("--") if not "model-directory" in i]).strip())'
```

## Display a file as plain text in a browser

```console
$ (echo -e 'HTTP/1.0 200 OK\n' && cat myfile.txt) | nc -Nlp 8000
```

If you have `socat` that's better, as chrome sometimes requests the favicon then netcat closes the listening socket. `socat` will serve multiple connections.

```console
$ (echo -e 'HTTP 1.0\n200 OK\n' && cat myfile.txt | tee /dev/stderr | socat - TCP-LISTEN:8000,fork,reuseaddr
```

Redisplay on reload.

- `-N` to netcat mean close socket on EOF from STDIN

```console
$ nodemon -e md --exec "clear; while test 1; do (echo -e 'HTTP/1.0 200 OK\n' && cat open-architecture.md) | nc -Nlp 8000; sleep 0.5; done; test 1"
```

## Do a vim commands on files in an automated way

Remove all newlines from end of file

```console
$ for file in $(echo examples/notebooks/*.ipynb); do vim -b '+set noeol' '+wq' $file; done
```

## Open a list of files in tmux panes

- https://github.com/tmux/tmux/wiki/Advanced-Use

```console
$ for file in $(git ls-files | grep -E '.*\.cpp|.*\.h|.*\.ino'); do tmux split-window -h "vim ${file}"; tmux next-layout; done
```

[![asciicast](https://asciinema.org/a/441107.svg)](https://asciinema.org/a/441107)

## Close all but the active tmux pane

```console
$  for pane in $(tmux list-pane | grep -v active | sed -e 's/:.*//g'); do export pane=$(tmux list-pane | grep -v active | sed -e 's/:.*//g' | head -n 1); tmux kill-pane -t $pane ; done
```

## Scrape page for links

```console
$ curl -sfL "https://pypi.org/simple/dffml/" | python -u -c 'import sys, json, bs4; print(json.dumps({link.get_text(): link.get("href") for link in bs4.BeautifulSoup(sys.stdin.read(), "html.parser").find_all("a")}))' | python -m json.tool
{
    "dffml-0.4.0.post0-py3-none-any.whl": "https://files.pythonhosted.org/packages/37/d5/6dc945d453cbdeb15db4249fe09e07bdd2e750a6f256fd893c81ced7bbbb/dffml-0.4.0.post0-py3-none-any.whl#sha256=031dd3a4ca57d46f568f7dd2711a223a26bc343f5f2ac36e1af881ead19e05b6",
    "dffml-0.4.0.post0.tar.gz": "https://files.pythonhosted.org/packages/b0/42/a151555fe3b45926fa041813f8513d883180bdb9e8def64d2d5260609743/dffml-0.4.0.post0.tar.gz#sha256=f6f898c504450e3514dd5791b31bcba21ad9edfc3e896ac5da9cbe3181af5d2b"
}
```

## Sign and verify data using ssh keys

References:
- https://www.agwa.name/blog/post/ssh_signatures

```console
$ podman run -v $HOME/.ssh/:/root/.ssh:ro -v $PWD:/usr/src/workdir -w /usr/src/workdir --rm -ti --entrypoint ssh-keygen lscr.io/linuxserver/openssh-server -Y sign -f /root/.ssh/mykeyname -n file CHANGELOG.md
$ (printf 'alice@example.com ' && cat ~/.ssh/mykeyname.pub) | tee allowed_signers
$ cat CHANGELOG.md | podman run -v $HOME/.ssh/:/root/.ssh:ro -v $PWD:/usr/src/workdir -w /usr/src/workdir --rm -i --entrypoint ssh-keygen lscr.io/linuxserver/openssh-server -Y verify -f allowed_signers -I alice@example.com -n file -s CHANGELOG.md.sig
Good "file" signature for alice@example.com with RSA key SHA256:RQsUY/opy5KWg+pesXcjI9I3I1z8udgkFAlOjDrv8cw
```

## git stash show patch

Shows just applied patch

```console
$ git stash show -p
```


## wsl start sshd

From powershell:

```console
$ Start-Job -ScriptBlock{wsl -u root -e mkdir -pv /run/sshd ; wsl -u root -e /usr/sbin/sshd -D}
```

## WSL Forward Port

Run the following from an Administrator PowerShell session

```powershell
PS C:\WINDOWS\system32> netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=2222 connectaddress=((wsl -u root -- sh -c "ip a | grep inet\ | grep -v 127.0.0").Split()[5].Split("/")[0]) connectport=22

PS C:\WINDOWS\system32> netsh advfirewall firewall add rule name="Open Port 2222 for WSL2" dir=in action=allow protocol=TCP localport=2222
Ok.

PS C:\WINDOWS\system32> netsh interface portproxy show v4tov4

Listen on ipv4:             Connect to ipv4:

Address         Port        Address         Port
--------------- ----------  --------------- ----------
0.0.0.0         2222        172.23.21.232   22
```

## GitHub CLI set secrets

```console
$ grep oauth_token: < ~/.config/gh/hosts.yml | sed -e 's/.*oauth_token: //g' | gh secret set MY_TOKEN
```

**../secrets**

```
SECRET_NAME secret value
```

```console
$ while IFS= read -r line; do gh secret set -R github.com/intel/dffml "$(echo $line | sed -e 's/ .*//')" --body "$(echo $line | sed -e 's/[^ ]* //')"; done < ../secrets
```

## Return repos from sourcegraph query

```console
$ curl 'https://sourcegraph.com/search/stream?q=context%3Aglobal%20repo%3A%5Egithub%5C.com%2Forg-name%2F.*%20log4j%20&v=V2&t=literal&dl=0&dk=html&dc=1&display=1500' | tee /tmp/org-name
$ python -c 'import sys, json, itertools; print("\n".join(list(set(filter(bool, itertools.chain(*map(lambda data: list(map(lambda content: content.get("repository", ""), data)) if isinstance(data, list) else [], filter(bool, map(json.loads, map(lambda line: "{}" if not line.startswith("data: ") else line.split(maxsplit=1)[-1], sys.stdin))))))))))' < /tmp/org-name
```

## Commit one file at a time

```console
$ git add $(git status --porcelain=2 | grep -v \? | awk '{print $NF}' | head -n 1) && git commit -sm "$(git status --porcelain=2 | grep -v \? | awk '{print $NF}' | sed -e 's/dffml_source_mongodb\///g' -e 's/examples\/shouldi\///g' -e 's/\//: /g' -e 's/_/ /g' -e 's/\.py//g' | head -n 1): Format with black"
```

## Run command on line in vim

Source: https://stackoverflow.com/questions/26809543/how-to-use-an-external-command-in-vim-to-modify-the-selection

```
'<,'> !python3 -c 'import sys, datetime; print("\n".join([line_datetime[0].strip() for line_datetime in sorted([(line, datetime.datetime.strptime(line.split()[-1], "\%Y-\%m-\%dT\%H:\%M:\%SZ")) for line in sys.stdin], key=lambda line_datetime: line_datetime[1])]))'
```

```
How to use an external command in Vim to modify the selection

Something I've found useful in other editors is the ability to:

take the selected text
run an external command and pass the selection to its stdin
take the external commands stdout and replace the current selection with it.
This way you can write useful text tools which operate on the selection using any language that can do basic io.

How can this be done with vim?

(Directly in the command line, or via a key binding?)

---

:'<,'>!command

'<,'> represents the (linewise) visual selection and is automatically inserted when you hit : and have something selected.

Example:

If you select a line containing:

print("Hello!")

and run the Vim command:

:'<,'>!python

the text will be replaced with Hello!.

If you want to set this to a key-binding (F5 to evaluate for example)

vnoremap <F5> :!python<cr>
```

- Example vim session insert, put this text, then press Escape key and type: `:'<,'> !bash`

```
mkdir -p schema/image/container/build/
cat > schema/image/container/build/README.md<<'EOF'
# Container Image Build Manifest
EOF
```

Use `tee` to replace output with existing and save sections of a file to smaller files

```
:'<,'> !tee output.txt
```

Line select and calculate hash

```
curl -sfL https://download.fedoraproject.org/pub/fedora/linux/releases/36/Server/x86_64/iso/Fedora-Server-netinst-x86_64-36-1.5.iso | sha256sum -
:'<,'> !bash
```

Find replace multiple things at the same time

```
:'<,'> !sed -e 's/support/readme/g' -e 's/Support/Readme/g' -e 's/SUPPORT/README/g'
```

## Dump GitHub comments to markdown file

```console
$ gh issue view https://github.com/intel/dffml/issues/1279 --json comments | jq -r '.comments[].body' | tr -d '\r' | sed -e 's/[[:space:]]*$//' -e 's/^#/\n\n#/g' | tee ~/comments.$(date "+%4Y-%m-%d-%H-%M").md
```

## Print the date in a YYYY-MM-DD-HH-SS format

```console
$ date "+%Y-%m-%d-%H-%M"
2022-02-02-06-34
```

## Reproducible archive via ssh

Source: https://reproducible-builds.org/docs/archives/

```console
$ tar -C /some/dir/with/stuff -c --sort=name --mtime="2015-10-21 00:00Z" --owner=0 --group=0 --numeric-owner --pax-option=exthdr.name=%d/PaxHeaders/%f,delete=atime,delete=ctime myfile.within.some.dir.with.stuff.exe | sshpass -p "$(python -m keyring get $USER password)" ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null "$USER@129.168.1.123" python -c '
import io
import os
import sys
import tarfile
import hashlib
import pathlib
import unittest
import tempfile

artifacts_contents = sys.stdin.buffer.read()

unittest.TestCase().assertEqual(
    hashlib.sha256(artifacts_contents).hexdigest(),
    "2bd972e3c980ee7dd376ca2c5988e2234d325d505f2124146abad226f1d163bb",
    "Artifact archive hash mismatch",
)

with tempfile.TemporaryDirectory() as tempdir:
    os.chdir(tempdir)
    with tarfile.open(fileobj=io.BytesIO(artifacts_contents), mode="r") as fileobj:
        fileobj.extractall()
        for path in pathlib.Path().rglob("*"):
            print(path.resolve())
'
```

## Converting datetimes from one timezone to another in Python

Example has local timezone as `-08:00` (PST).

```python
>>> from datetime import datetime, timedelta, tzinfo
>>> (datetime.fromisoformat('2022-02-13T04:30') + timedelta(hours=28)).isoformat() + "-08:00"
'2022-02-14T08:30:00-08:00'
>>> (datetime.fromisoformat('2022-02-13T04:30') + timedelta(hours=28) + timedelta(hours=8) + timedelta(hours=5, minutes=30)).isoformat() + "+05:30"
'2022-02-14T22:00:00+05:30'
```

## XZ compressed asciinema recording

Dotfiles: https://github.com/pdxjohnny/dotfiles/blob/cae7a366d7766bb1a82c478f0aedc6a829630677/.asciinema_source

Record (for remote: `ssh -t hostname`)

```console
$ export REC_TITLE=""; export REC_HOSTNAME="$(hostname)"; python3.9 -m asciinema rec --idle-time-limit 0.5 --title "$(date -Iseconds): ${REC_HOSTNAME} ${REC_TITLE}" --command "sh -c 'tmux a || tmux'" >(xz --stdout - > "$HOME/asciinema/${REC_HOSTNAME}-rec-$(date -Iseconds).json.xz")
```

Check size

```console
du -h ~/Downloads/ascii*
```

Playback

```console
xz --stdout -d - < ~/Downloads/asci* | python -m asciinema play -s 20 -
```

Upload **TODO** title from recording first line, this is an example where (Alice shell / dataflows) parallel stream processing shines due to `tee >(export title=$(read))` not being available for `${title}` in place of `asciinema` with no-ghost bash. With ghost being with Alice, with these parallel / concurrent sub streams.

```console
$ asciinema upload $(ls ~/asciinema/rec-$(hostname)-*ndjson | tail -n 1)
$ url=$(unxz -d < $(ls ~/asciinema/$(hostname)-* | tail -n 1) | python -m asciinema upload /dev/stdin 2>&1 | grep https | awk '{print $NF}'); echo "[![asciinema](${url}.svg)](${url})" | xclip -selection c
$ unxz -d < $(ls ~/asciinema/$(hostname)-* | tail -n 1) | python -m asciinema upload /dev/stdin 2>&1 | grep https | awk '{print $NF}' | xclip -selection 
$ git diff | gh gist create -d $(unxz -d < $(ls ~/asciinema/$(hostname)-* | tail -n 1) | python -m asciinema upload /dev/stdin 2>&1 | grep https | awk '{print $NF}') -f with-rec.patch -
```

## OBS Studio

https://snoober.home.blog/

## wsl ssh

```console
$ wsl -u root -- mkdir -pv /run/sshd; wsl -u root -- /usr/sbin/sshd -D -o ListenAddress=0.0.0.0
```

## Parse time with timezone

```console
$ python -c 'import sys, datetime; print(datetime.datetime.strptime(sys.argv[-1], "%d %b %Y %H:%M:%S %z"))' '11 Jan 2022 00:44:19 +0800'
2022-01-11 00:44:19+08:00
```

## date command

```console
$ date "+%4Y-%m-%d-%H-%M"
```

## File transfer with progress

Machine sending file.

```console
$ nc -Nlp 9999 < filename
```

Machine receiving file.

```console
$ dd status=progress bs=1M if=<(cat < /dev/tcp/example.com/9999) of=/mnt/c/Users/$USER/Downloads/filename
```

## Work on one branch and make change on other branch push to github and run CI

```console
$ nodemon -e yml --exec "clear; export branch=feedface; git branch -D $branch; git checkout upstream/main-test -b $branch; python -c 'print(\"A\" + (\"R\" * (2**8)))' > ahoy-there-matey-its-me-file; git add . && git commit -sam "${branch}" && git push -d origin "${branch}"; git push -fu origin $(git status | head -n 1 | awk '{print $NF}') && gh pr create --base main-test --title "$branch" -F /dev/null"
```

## Download and install an autotools project with a vendored `autoreconf -i`

```console
$ export tempdir=$(mktemp -d); (cd $tempdir && curl -sfL 'https://www.kernel.org/pub/software/scm/git/git-2.36.0.tar.gz' | tar xz && cd git* && ./configure --prefix=$tempdir && make -j $(($(nproc)*4)) && ./git version && echo $tempdir | tee /tmp/git_pwd && sudo chmod a+rx -R $tempdir) && sudo chmod a+r /tmp/git_pwd
```

A non autoconf project using https://www.gnu.org/prep/standards/html_node/Directory-Variables.html#Directory-Variables for
`prefix      ?= /usr/local`

```console
$ export tempdir=$(mktemp -d); (cd $tempdir && curl -sfL https://github.com/stefanhaustein/TerminalImageViewer/archive/refs/tags/v1.1.1.tar.gz | tar xz && cd Terminal*/src/main/cpp && make -j $(($(nproc)*4)) && make install prefix=$tempdir)
install -D tiv /tmp/tmp.ONSIPt2W5E/bin/tiv
```

## Near local github actions debug

```console
$ gh run rerun --failed $(gh run list | head -n 2 | tail -n 1 | awk '{print $(NF-2)}')
$ gh run watch --exit-status -i 1 $(gh run list | head -n 2 | tail -n 1 | awk '{print $(NF-2)}')
$ gh run view $(gh run list | head -n 2 | tail -n 1 | awk '{print $(NF-2)}')
$ gh run view --log $(gh run list | head -n 2 | tail -n 1 | awk '{print $(NF-2)}')
```

```console
gh run view --log-failed $(gh run list --workflow feedface.yml | head -n 1| awk '{print $(NF-2)}')
```

## Find the binaries

**TODO** CVE Bin Tool approach

```console
$ file $(find $tempdir -name git)
/tmp/tmp.2Wd0hLb8oN/bin/git:                                      ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=c1357c209d816a9738b0af78d017a6c2bfba71e7, with debug_info, not stripped
/tmp/tmp.2Wd0hLb8oN/git-2.36.0/bin-wrappers/git:                  POSIX shell script, ASCII text executable
/tmp/tmp.2Wd0hLb8oN/git-2.36.0/git:                               ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=c1357c209d816a9738b0af78d017a6c2bfba71e7, with debug_info, not stripped
/tmp/tmp.2Wd0hLb8oN/git-2.36.0/contrib/mw-to-git/bin-wrapper/git: POSIX shell script, ASCII text executable
/tmp/tmp.2Wd0hLb8oN/libexec/git-core/git:                         ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=c1357c209d816a9738b0af78d017a6c2bfba71e7, with debug_info, not stripped
```

## UNIX anyone can execute and read anything in these dirs

```console
$ chmod -R a+rx $dir
```

## Debugging actions self-hosted runners

```console
$ gh run rerun --failed $(gh run list | head -n 2 | tail -n 1 | awk '{print $(NF-2)}') && sleep 1 && gh run watch --exit-status -i 1 $(gh run list | head -n 2 | tail -n 1 | awk '{print $(NF-2)}') || gh run view --log $(gh run list | head -n 2 | tail -n 1 | awk '{print $(NF-2)}')
```

## Used the menu button on the keyboard for the first time ever today

https://support.google.com/chrome/answer/10483214?hl=en

## Edit a file only if autoformatting passes

Can be used to hand edit xxd as well if you wanted to

```console
$ cp old-admin.json old-old-admin.json; cp ~/.local/admin.json old-admin.json && python -m json.tool old-admin.json > admin.json || cp old-admin.json admin.json && cp admin.json staged.json && vim staged.json && python -m json.tool < staged.json > admin.json && cp admin.json ~/.local/admin.json
```


## Quick pop shell from python

```python
import os
directory = "/tmp"
os.system(f"bash -c 'cd {directory}; pwd; exec bash'")
```

## Capture output of shell stderr and out to file by datestamp

Combine with bash history to create Alice herstory as fully connected view.

```console
$ nodemon -e py --exec 'clear; alice please contribute -log debug -repos https://github.com/pdxjohnny/testa -- recommended community standards 2>&1 | tee .output/$(date +%4Y-%m-%d-%H-%M).txt; test 1'
```

```console
$ grep -n -C 5 -i contribute_readme_md $(find .output/ | sort | tail -n 1)
```

## Python reverse lines in file

```console
$ python -c 'import sys; print("".join(list(filter(bool, sys.stdin))[::-1]))' < commits.txt | tee commits-ordered.txt
```

## Cherry pick the last two commits

```console
$ git cherry-pick branchname~2..branchname
```

## Print git log with links to commits on github

```console
$ export remote="$(git remote get-url $(git remote))" && git log --pretty=oneline -n 2 | sed -e "s#^#${remote}/commit/#"
```

## You can drag files (videos even) right into a markdown document to uploda them to github

## Download a python package with curl and verify the contents using the SHA

Sometimes downloading a package with pip will fail.

```console
$ ulimit -c unlimited
$ python -m pip download torch
Collecting torch
  Downloading torch-1.12.1-cp39-cp39-manylinux1_x86_64.whl (776.4 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━╸ 776.3/776.4 MB 13.0 MB/s eta 0:00:01Killed
```

Look for the path to the download you want.

```console
$ curl -sfL https://pypi.org/simple/torch/ | grep torch-1.12.1-cp39-cp39-manylinux1_x86_64.whl
    <a href="https://files.pythonhosted.org/packages/1e/2f/06d30fbc76707f14641fe737f0715f601243e039d676be487d0340559c86/torch-1.12.1-cp39-cp39-manylinux1_x86_64.whl#sha256=9b356aea223772cd754edb4d9ecf2a025909b8615a7668ac7d5130f86e7ec421" data-requires-python="&gt;=3.7.0" >torch-1.12.1-cp39-cp39-manylinux1_x86_64.whl</a><br />
```

Download the package.

```console
$ curl -fLOC - https://files.pythonhosted.org/packages/1e/2f/06d30fbc76707f14641fe737f0715f601243e039d676be487d0340559c86/torch-1.12.1-cp39-cp39-manylinux1_x86_64.whl
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  740M  100  740M    0     0  85.1M      0  0:00:08  0:00:08 --:--:--  106M

Verify the SHA appended to our downloaded URL from our initial command.

```console
$ sha256sum -c - <<<'9b356aea223772cd754edb4d9ecf2a025909b8615a7668ac7d5130f86e7ec421  torch-1.12.1-cp39-cp39-manylinux1_x86_64.whl'
torch-1.12.1-cp39-cp39-manylinux1_x86_64.whl: OK
```

Install the package

```console
$ python -m pip install ./torch-1.12.1-cp39-cp39-manylinux1_x86_64.whl
```

Now it should appear to pip as installed

```console
$ python -m pip install -U pip setuptools wheel
Defaulting to user installation because normal site-packages is not writeable
Requirement already satisfied: pip in /.pyenv/versions/3.9.13/lib/python3.9/site-packages (22.2.1)
Collecting pip
  Downloading pip-22.2.2-py3-none-any.whl (2.0 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 2.0/2.0 MB 10.3 MB/s eta 0:00:00
Requirement already satisfied: setuptools in /.pyenv/versions/3.9.13/lib/python3.9/site-packages (63.2.0)
Collecting setuptools
  Downloading setuptools-65.3.0-py3-none-any.whl (1.2 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 1.2/1.2 MB 16.5 MB/s eta 0:00:00
Requirement already satisfied: wheel in /.pyenv/versions/3.9.13/lib/python3.9/site-packages (0.37.1)
Installing collected packages: setuptools, pip
Successfully installed pip-22.2.2 setuptools-65.3.0

[notice] A new release of pip available: 22.2.1 -> 22.2.2
[notice] To update, run: pip install --upgrade pip
$ pip install torch==1.12.1
Defaulting to user installation because normal site-packages is not writeable
Requirement already satisfied: torch==1.12.1 in ./.local/lib/python3.9/site-packages (1.12.1)
Requirement already satisfied: typing-extensions in ./.local/lib/python3.9/site-packages (from torch==1.12.1) (4.3.0)
```

## Read markdown in terminal

```console
curl -sfL https://github.com/openai/whisper/raw/main/README.md | python -m rich.markdown /dev/stdin | less -r
```

## Windows disable taskbar icon blinking

Source: https://answers.microsoft.com/en-us/msteams/forum/all/how-to-stop-teams-from-blinking-in-taskbars-when/decc661a-7c4a-4f7e-a729-afdee2f14824

> Rudy (can't fail): Please edit the value of ForegroundFlashCount from the HKEY_CURRENT_USER\Control Panel\Desktop registry to the value of 1, then the icon will flash once. Which can reduce the number of icon flashing. Note: Setting to 0 will make the icon continuously flash. If you still need to stop Teams' icon flash, for this Windows OS related question, we will move this thread back to the original category. Meanwhile, if you don't want to use Teams client, actually, you can let it run in background via selecting the "Open application in background" options in Settings > General.

## Display date with timezone

```console
$ date -Iseconds
2022-10-05T22:30:42-07:00
```

## Reboot to BIOS

Source: https://twitter.com/ADurrante/status/1578052630043140101?s=20&t=Ypq4nyTINjL_gPctxvvN1A

```console
$ shutdown /r /fw /f /t 0
```

## Git push to current branch

```console
$ git push -u origin $(git branch | grep -E '^\*' | sed -e 's/\* //')
```

## `ssh` port forward remote port to enable connection via local port to port on remote server

```console
$ ssh -nNT -L 8000::8000 target
```

## `ssh` port forward local port to enable connection to port on remote server to access local port

```console
$ ssh -nNT -R 127.0.0.1:6000:0.0.0.0:6000 target
```

Ensure `GatewayPorts` is `yes` server side

**/etc/ssh/sshd_config**

```
AllowTcpForwarding yes
GatewayPorts yes
```

## JSON to YAML with Python CLI

Source: https://github.com/intel/dffml/blob/8847989eb4cc9f6aa484285ba9c11ff920113ed3/docs/arch/0008-Manifest.md

```console
$ python -c "import sys, pathlib, json, yaml; print(yaml.dump(json.load(sys.stdin)))" < manifest.json
```

## YAML to JSON with Python CLI

```console
$ python -c "import sys, pathlib, json, yaml; print(json.dumps(yaml.safe_load(sys.stdin.read())))" < manifest.json
```

## Post WIP diff to Pull Request

```console
$ (echo '```diff' && git diff && echo '```') | gh pr comment -F -
```

## Mirror site with wget

> Source: https://handyman.dulare.com/advanced-wget-website-mirroring/

```console
$ wget --mirror --convert-links --adjust-extension --page-requisites  http://www.mywebsite.com/
```

## Remove bullshit docker images

```console
$ docker rm $(docker ps -qa)
$ docker rmi $(docker images | grep \<none\> | awk '{print $3}')
```

## Print all lines and variables in a Python file as it executes

https://github.com/alexmojaki/snoop

```console
$ python -m pip install snoop
$ sed -e 's/import os/import snoop\n&/g' -e 's/def main/@snoop\n&/g' ~/dffml/.github/actions/create_manifest_instance_build_images_containers/images_containers_manifest.py
```

## Remap fields with jq

- https://github.com/fadado/JBOL/blob/master/doc/JQ-Distilled.md
- https://github.com/fadado/JBOL

```console
$ echo '{"mykey": {"a": 42}}' | jq 'to_entries[] | {(.key): {"alice": (.value."a")}}' | jq -s 'add'
{
  "mykey": {
    "alice": 42
  }
}
```

### Blocklist with jq

```console
$ cat inputs.txt | jq --argjson blocklist "$(cat blocklist.txt | jq -cnR '[inputs | select(length>0) | sub("\\*.*";"") | sub("@.*";"")]')" -nR '[inputs | select( . as $in | $blocklist | index($in) | not)]'
```

## SSH through socks proxy with netcat

- https://superuser.com/questions/454210/how-can-i-use-ssh-with-a-socks-5-proxy

```console
$ ssh -R 80:localhost:8080 -o ProxyCommand="nc -X 5 -x 127.0.0.1:6000 %h %p" nokey@localhost.run
```

```console
$ ssh -nT -R 80:localhost:8080 nokey@localhost.run 2>/dev/null | grep --line-buffered 'tunneled with tls' | python -c 'import sys; print(sys.stdin.readline().split()[-1])' | tee public-url.txt &
```

## Curl and PowerShell to Python

```python
import sys, http.server

http.server.BaseHTTPRequestHandler.do_POST = lambda self: list(
    [
        self.send_response(200),
        self.send_header("Content-type", "text/plain"),
        self.send_header("Content-length", 3),
        self.end_headers(),
        self.wfile.write(b"OK\n"),
        sys.stdout.buffer.write(self.rfile.read()),
        sys.exit(0),
    ]
)
http.server.HTTPServer(
    ("0.0.0.0", 8080), http.server.BaseHTTPRequestHandler
).serve_forever()
```

```console
$ python -uc 'import sys, http.server; http.server.BaseHTTPRequestHandler.do_POST = (lambda self: list([self.send_response(200), self.send_header("Content-type", "text/plain"), self.send_header("Content-length", 3), self.end_headers(), self.wfile.write(b"OK\n"), sys.stdout.buffer.write(self.rfile.read()), sys.exit(0)])); http.server.HTTPServer(("0.0.0.0", 8080), http.server.BaseHTTPRequestHandler).serve_forever()' | python -um json.tool | tee data.json
```

```bash
curl -X POST -d @file.ext http://localhost:8080/
```

```powershell
$body = Get-Content -Path data.json -Encoding UTF8 -Raw
Invoke-WebRequest -Uri http://localhost:8080/ -Method POST -Body $body
```

## Digital Ocean

I ❤️ DigitalOcean 🐳

- https://github.com/tmux/tmux/wiki/Advanced-Use#piping-pane-changes
- https://dokku.com/docs/getting-started/install/digitalocean/

### Create a Docker Droplet

```bash
export COMPUTE_DOMAIN=chadig.com && export COMPUTE_SUBDOMAIN=scitt.eve export COMPUTE_NAME=scitt-eve

doctl compute ssh-key import $(hostname) --public-key-file "${HOME}/.ssh/id_rsa.pub"; (set -xu && doctl compute droplet create --ssh-keys "$(ssh-keygen -l -E md5 -f ~/.ssh/id_rsa.pub | awk '{print $2}' | sed -e 's/MD5://1')" --image "$(doctl compute image list-distribution --no-header --format Slug | grep fedora | tail -n 1)" --size $(doctl compute size list --no-header --format Slug | head -n 2 | tail -n 1) --region sfo3 --droplet-agent=true --tag-name scitt "${COMPUTE_NAME}" && time bash -xeuo pipefail -c 'STATUS=new; while [[ "x${STATUS}" = "xnew" ]]; do STATUS=$(doctl compute droplet get --no-header --format Status ${COMPUTE_NAME}); done' && export COMPUTE_IPV4="" && while [[ "x${COMPUTE_IPV4}" = "x" ]]; do export COMPUTE_IPV4=$(doctl compute droplet list --no-header --format PublicIPv4 "${COMPUTE_NAME}"); done && doctl compute domain records create --record-name "${COMPUTE_SUBDOMAIN}" --record-ttl 3600 --record-type A --record-data "${COMPUTE_IPV4}" "${COMPUTE_DOMAIN}" && while ! ssh -i "${HOME}/.ssh/id_rsa.pub" -o StrictHostKeyChecking=no "root@${COMPUTE_IPV4}" "set -x && dnf -y install git tmux python3 python3-pip && tmux new-session -d -s http_server 'python -m http.server 80'"; do sleep 1; done)

doctl compute droplet delete --force $(doctl compute droplet get --no-header --format ID "${COMPUTE_NAME}")

export COMPUTE_IPV4=$(doctl compute droplet list --no-header --format PublicIPv4 "${COMPUTE_NAME}")
ssh -t -i "${HOME}/.ssh/id_rsa.pub" -o StrictHostKeyChecking=no "root@${COMPUTE_IPV4}" "tmux a"
```

## tar

### Extract single file or set of files

```console
$ tar -xzv --to-stdout --wildcards --no-anchored 'path/from/root/*.ext'
```

[![asciicast](https://asciinema.org/a/620287.svg)](https://asciinema.org/a/620287)


## Bash

### Arrays

```bash
#!/usr/bin/env bash
# Bash Array Example
# @lice: Write a python sphinx extension which transforms source code in bash,
# python, and javascript, with plugins for more languages for source AST to
# docutils class instances. Create a demo for it using this file as an example
# source file to build to using the new plugin. Output in HTML.
set -exuo pipefail

declare -a example_array=()

example_array[${#example_array[@]}]="Value for example_array index 0"

for value in "${example_array[@]}"; do
  echo $value
done
```

## ffmpeg

- https://pdxjohnny.github.io/ffmpeg/
