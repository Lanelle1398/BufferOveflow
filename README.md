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

