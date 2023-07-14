# Buffer Oveflow Attack

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

<p> Ctrl+shift+v to paste <br> Ctrl+o <br> File name to write: envexec.sh <br> <br> Chmod+x  envexec.sh will make the script executable <br> <br> Do the same for our vulnerable program, vuln.c  <br></p>

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

<img width="342" alt="image" src="https://github.com/Lanelle1398/BufferOveflow/assets/88471126/26b0e0c4-02d1-403a-b68f-20f049a26560">
<p> <br> Purpose of vuln.c: <br> 
<br> This program takes a character array with a max size of 500 bytes, called buffer.<br>
<br> The string that the user enters into the terminal (argv[1]) is copied and entered into the buffer. <br>
<br> If argv[1] is greater than 500 bytes, it will overflow the buffer. <bt>
<br> Strcpy is vulnerable, because strcpy  by default will continue copying values until it hits a null pointer.<br>  <p>


<img width="396" alt="image" src="https://github.com/Lanelle1398/BufferOveflow/assets/88471126/02fe9fb1-7d94-43bb-9e2e-6856cd63e756">

<p> <br> Now it’s time to execute the buffer overflow:<br>
<br>I compile this with gcc and disable stack protector. <br>
<br>gcc -z execstack -fno-stack-protector -mpreferred-stack-boundary=2 -g vuln.c -o vuln <br>
<br>Doing this removes the safeguards that gcc has to prevent buffer overflow attacks. <br> 
<br>Stack protector works by pushing a canary (a random integer) on the stack right after the function pointer has returned.<br>
<br>The canary value is checked before the system returns and if it has changed, the system is aborted. This is done rather than continuing and returning to wherever the attacker wants the system to point to.<br> </p>


Clean up our bash environment with this command. Strip it of every possible setting. 
<p> <br>The gdb is run with the command vuln.<br>
<br>./envexec.sh -d vuln<br> </p>
	
<img width="468" alt="image" src="https://github.com/Lanelle1398/BufferOveflow/assets/88471126/2d712bfa-4a91-4880-8954-2c3e6193648d">

<p> <br>Run ‘vuln’ program in gdb<br>
<br> gdb vuln <br>
<br> Disambling my code <br><p>
<img width="386" alt="image" src="https://github.com/Lanelle1398/BufferOveflow/assets/88471126/74e5ba24-f812-47b6-b0a2-eef920b29a0f"><p> 
<p><br>Let’s try overflowing the program
Print 512 As in the program. <br>  
<br>‘\x41’ is A in hexadecimal<>br <p>
<img width="468" alt="image" src="https://github.com/Lanelle1398/BufferOveflow/assets/88471126/2f7295bd-bcba-4041-a4be-513a89108f39">
<p></p> <br> We overwrite eip, ebp  (return pointer, base pointer)<br>
<br>Show 200x characters. Take the address of the stack pointer and go back 550 before that address.<br> <p></p>
<img width="468" alt="image" src="https://github.com/Lanelle1398/BufferOveflow/assets/88471126/1efaa8a0-9e14-493c-a205-e26f5b6305e7">


<br><>br








