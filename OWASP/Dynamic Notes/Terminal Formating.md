# Terminal Formating
```shell
script /dev/null -c bash
<CTRL_Z>
stty raw -echo; fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 46 columns 204
```