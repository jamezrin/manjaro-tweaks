# Installing libtinfo.so.5

If you try to use `clang-format` in Arch Linux you might find yourself with an error 
saying that the library `libtinfo.so.5` couldn't be found.

There are plenty of proposed solutions, some involve symlinking other libraries, but in my 
experience that didn't work and the solution ended up being installing just one package from the AUR.

That package is `ncurses5-compat-libs`. 
If you use `yay` you can install it with `yay -S ncurses5-compat-libs`.

## Chain of symlinks

If you are curious, this package fixes this issue by creating a symlink to `/usr/lib/libncurses.so.5`.

And that is also a symlink, which points to `/usr/lib/libncursesw.so.5`.

And that one, is also a symlink, this time pointing to the regular file `/usr/lib/libncursesw.so.5.9`.

**This regular file is not present without the package we just installed.**

