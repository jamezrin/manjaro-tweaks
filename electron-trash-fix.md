# Electron Trash Fix

If you have ever used an Electron-based application that deletes files or directories by moving them to the trash bin
under KDE or other Qt-based DEs, it's probable that you have experienced freezes while doing such action.

I have experienced these freezes while using Visual Studio Code on my two computers, both having Manjaro KDE as its
operating system.

Searching for a fix for this issue I stumbled upon the issue [vscode#54493](https://github.com/microsoft/vscode/issues/54493).
Unfortunately this issue was closed with a supposed fix but it came back somehow or it wasn't really fixed. 
Anyhow, I managed to find a fix myself. 

Apparently, the backend system for trash bin operations is `gio` by default, but apparently, at least under KDE, it doesn't seem
that way and it seems like it defaults to something else.

I discovered this by playing around with the `ELECTRON_TRASH` variable, and I found that setting it to what it's supposed
to be the default (`gio`) fixes the issue entirely. I guess it is set `kioclient5` or `kioclient`. 

After finding this I posted the solution  [comment in an issue](https://github.com/microsoft/vscode/issues/90034#issuecomment-582115953)
I created on VS Code, but I am pretty sure it's something that has to be fixed in Electron. 

## The fix

You can fix this issue by creating a file that gets executed automatically on startup/login. 
I chose to do it in `~/.config/plasma-workspace/env/electron-trash-gio.sh`

Edit that file and put this line

```bash
export ELECTRON_TRASH=gio
```

After that, reboot or logout and log back for it to apply.