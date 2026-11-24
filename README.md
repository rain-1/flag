# Flag read counter

On Linux with kernel BTF, fentry support, and bpftrace 0.23 or later, run
`sudo ./watch-flag`. The command creates an empty `flag.txt` if needed.
To use an existing file elsewhere, run `sudo ./watch-flag /path/to/flag.txt`.
Once bpftrace attaches, run `cat flag.txt` in another terminal. The cumulative
`@read_attempts` count is printed once a second and on Ctrl-C.

The count is system-wide and follows the file's inode, including reads through
hard links, symlinks to the file, and already-open descriptors. Restart the
watcher after replacing the file with a new inode. The count starts at zero
on each run.

Each read that reaches the kernel's file-permission hook counts once, including
reads that return EOF or fail after reaching the hook. A readable file mapping
counts once when it reaches the mmap hook. Opening a file without reading it,
attempts rejected before either hook (including a denied open), and memory
loads through an existing mapping are outside this coverage. A command such as
`cat` can issue several reads.

To enable UDP notifications, pass a destination host and port:

```sh
sudo ./watch-flag --udp 127.0.0.1 9000
```

The watcher sends a datagram containing the text `flag-read` for each counted
event and continues printing the cumulative total. Each datagram contains only
that fixed text. UDP delivery is best effort, and under load the tracing event
buffer can drop notifications while the kernel counter continues to increment.
