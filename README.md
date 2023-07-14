# BufferOveflow

The first step is to disable memory randomization:
cat /proc/sys/kernel/randomize_va_space

This line normally gives us 2. 2 is the default setting on Linux kernels. This value means that address space is randomized, including the position of the stack, virtual dynamic shared object page, shared memory regions, and data segments.

We can disable this by typing:
sudo bash -c 'echo "kernel.randomize_va_space = 0" >> /etc/sysctl.conf' 
 (we append this in the etc sudo control file)

sudo sysctl -p
when I run this command, it shows me that randomization is turned off.
I can see that randomization is turned off because of the output  “ kernel.randomize_va_space =0”

Type in the following command to verify that randomization is turned off:
cat /proc/sys/kernel/randomize_va_space
We get ‘0’

Next we need to disable debugging core dumps. 
The maximum core dump size is enforced by ulimit. 
If ulimit is zero, core dump is disabled entirely. 
If you give core dump a limit and the core dump generated is more than the limit, the data would be truncated and you might not get the traces.
So, it is best to set core dump size as unlimited.

<img width="468" alt="image" src="https://github.com/Lanelle1398/BufferOveflow/assets/88471126/f3dd8a20-822f-4b99-b63b-150308a476fb">


I get into nano .
I paste envexec.sh script inside of nano
```
#!/bin/sh

while getopts "dte:h?" opt ; do
  case "$opt" in
    h|\?)
      printf "usage: %s -e KEY=VALUE prog [args...]\n" $(basename $0)
      exit 0
      ;;
    t)
      tty=1
      gdb=1
      ;;
    d)
      gdb=1
      ;;
    e)
      env=$OPTARG
      ;;
  esac
done

shift $(expr $OPTIND - 1)
prog=$(readlink -f $1)
shift
if [ -n "$gdb" ] ; then
  if [ -n "$tty" ]; then
    touch /tmp/gdb-debug-pty
    exec env - $env TERM=screen PWD=$PWD gdb -tty /tmp/gdb-debug-pty --args $prog "$@"
  else
    exec env - $env TERM=screen PWD=$PWD gdb --args $prog "$@"
  fi
else
  exec env - $env TERM=screen PWD=$PWD $prog "$@"
fi
```
Ctrl+shift+v to paste
Ctrl+o
File name to write: envexec.sh
Chmod+x  envexec.sh will make the script executable

Do the same for our vulnerable program, vuln.c 

```
#include <stdio.h>
#include <string.h>

int main (int argc, char** argv)
{
	char buffer[500];
	strcpy(buffer, argv[1]);

	return 0;
}
```
Then I ‘ls’ to make sure envexec.sh and vuln.c were added.


