slow sudo issue 

I was running Pop!\_OS jammy 22.04 x86\_64 on X11.
At the time, I had everything configured the way I wanted.
Of particular note, I installed Howdy, an open-source 'Windows Hello' type of sign-in feature that could also be used for authorization for admin purposes. I also installed Homebrew for Linux and oh-my-posh using Homebrew.
With this in mind, after I configured the terminal to my liking (including using Gogh to changee my terminal colors), I would have a very specific issue that would happen.

The issue:
On suspend, Linux puts the machine into an S3 sleep state. I was using Pop!\_OS on a laptop, so this would happen every time I close my laptop.
Upon waking my device from suspend, most functioning would be normal. 
However, when opening an interactive terminal session, it would take about 20 seconds for my terminal prompt to show up and allow me to type.
This would only happen after waking up from sleep after I configured things to my liking, so I surmied the issue would either involve my system configuration or some issue with the sleep state.
To emphasize, if I rebooted my laptop, the prompt would show up quickly. The issue would only occur after the laptop woke from sleep/suspend

Problem Solving: .bashrc
I looked at my .bashrc file to see if there were any lines I don't recall adding myself. I saw nothing.
When looking into how to observe the behavior of the .bashrc file as it was running, I was introduced to the command `set -x`.
When added to the top of a shell script, `set -x` prints every command run to the terminal before it is executed so that I can see where things are stalling.
With this first tool, I could see that it was stalling at the line:
`eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"`.
This is the line that prepares the Homebrew environment for a given terminal session. 
Commenting out the line fixed the long wait problem, but it prevented any program installed through Homebrew to stop working.
Thus, I had to investigate what was going on with `brew shellenv` that was creating this long wait time.

Rabbithole: brew
I tried many fixes before getting on the right track with solving the issue, the main of which surrounded the idea of attempting to cache what the command does, moving the line to other places or making it so that it is only run when needed, etc. However, a prime example of this would be moving the line `eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"` to my .bash\_profile, which on Pop!\_OS is just .profile. This didn't work. 
Aside from checking the brew github for similar issues, I also tried:
Looking into iotop to see if there were any odd read/writes happening
Checking dmesg for odd behaviors from S3 suspend (`sudo dmesg | grep 'S3\|suspend'`
etc.

The issue with these approaches are that seemingly, every instance of the interactive shell *needs* to run this line in order for things to be set up properly. The suggestion to move it to the .bash\_profile was a very [compelling answer](https://github.com/orgs/Homebrew/discussions/4660) found on github with someone experiencing a similar issue, which required me to read a section of the install script to support their claim, which made sense to me at the time. Though it did not solve my problem.

>ohai "Next steps:"
>case "${SHELL}" in
>  */bash*)
>    if [[ -n "${HOMEBREW_ON_LINUX-}" ]]
>    then
>      shell_rcfile="${HOME}/.bashrc"
>    else
>      shell_rcfile="${HOME}/.bash_profile"
>    fi
>    ;;

Ultimately, what got me to my next step was doing the same thing I did to my .bashrc file, which was to add `set -x` to the brew script, located at `home/linuxbrew/.linuxbrew/bin/brew`.
As a side note, I was able to learn that the scripts in this binary folder are full of symbolic links to the scripts that are in the Homebrew stable branch (git pulled to my local machine via the install script, I assume).

Running brew shellenv showed me that the script was stalling at a section involving the idea of resetting the timestamp for sudo. More specifically, the script was stalling on the following output from `set -x` being enabled on the script:

>+/usr/bin/sudo --reset-timestamp

Which I traced to the section:

># Avoid picking up any random `sudo` in `PATH`. #
>if [[ -x /usr/bin/sudo ]] #
>then #
>  SUDO=/usr/bin/sudo #
>else #
># Do this after ensuring we're using default Bash builtins. #
>  SUDO="$(command -v sudo 2>/dev/null)" #
>fi #
>
># Reset sudo timestamp to avoid running unauthorized sudo commands #
>if [[ -n "${SUDO}" ]] #
>then #
>  "${SUDO}" --reset-timestamp 2>/dev/null || true #
>fi #
>unset SUDO #

Problem Solving: sudo -k
The line `sudo --reset-timestamp` as it says, resets the sudo timestamp, which means that the next time sudo is run in a session, the password must be entered. 
This removes the amount of time that can be granted, after using sudo, that commands that would need sudo permissions don't need to have the password be entered to run them. 
`sudo --reset-timestamp` is equivalent to `sudo -k`.
Once discovering this was where the hanging was happening, I commented out the above section to see how the initialization time would change.
Commenting this section out made loading near instant (as it should be), so I left it commented out for the time being and then resolved to figure out the issue with `sudo -k` taking so long to run.
When I would run `sudo -k` in isolation with my current setup, it would always take a while to load. 
I wasn't sure how long and if it was consistent, so I did `time sudo -k` to make easier comparisons.
After a certain point, I would run this so often that I wrote a script to run `time sudo -k` any defined number of times:

```bash
#!/bin/bash

# Check if an argument is provided
if [ $# -eq 0 ]; then
    echo "Usage: $0 <number_of_iterations>"
    exit 1
fi

iterations=$1

# Run the command the specified number of times
for ((i=1; i<=$iterations; i++)); do
    echo "Iteration $i:"
    time sudo -k #> /tmp/time_output_$i 2>&1
done

# Filter and print the "real" lines
#cat $HOME/time_output_*.real | sort -n

# Clean up temporary files
#rm /tmp/time_output_*

```

I was able to glean from this script that `sudo -k` would consistently take about 20 seconds exactly to run.


The first thing I tried was to uninstall Howdy. 
Since Howdy would use the face scanning feature in place of sudo, I thought it would be a good place to start in reducing the time it would take to flush credentials.
This somewhat worked.
After uninstalling Howdy, the time it would take to run `sudo -k` became less consistent, but it was more often than not less than 20 seconds.
Now, `sudo -k` would range from 20 seconds down to it's normal time of around a fraction of a second. 
With this in mind, `sudo -k` would still run on average closer to about 10 seconds consistently.
I tried to investigate other things I might have installed, using `sudo apt list --installed` and `history | grep "sudo apt install"` to investigate, but nothing from those lists stood out to me as needing to use the sudo permission in a way that would drastically affect the timing like Howdy did. 

Rabbithole: sudo
I spent a lot of time trying to figure out if I could fix sudo in some way to cut down on the time. 
This included, but is not limited to:
Trying to uninstall sudo and purge sudo from the system and then reinstalling, which lead to me having to learn about the setuid bit and using that on usr/bin/sudo (`chmod u+s /usr/bin/sudo`/`chmod 4755 /usr/bin/sudo`) from a live environment
Trying to determine exactly what `sudo -k` does by learning about the components it interacts with like PAM (Pluggable Authentication Modules), the sudoers file 
Looking into `visudo` to see if there were any odd configurations in the sudo config files (`/etc/sudoers.d`/`/etc/sudoers`)
Learning about `pkexec` to change permissions and run administratively in the absence of `sudo`
Using `strace` and `strings` to try to make sense of when `sudo -k` was run and the `sudo` file itself, respectively
And more, including notable commands such as `df $(which sudo)`
Setting up an Ubuntu virtual machine and a Pop!\_OS virtual machine and configuring the terminal the same way outside of installing Howdy, but after trying to replicate the issue, `sudo -k` would still run quickly.

It seemed like I was reaching many dead ends over this time, because nothing was changing the timing for `sudo -k`

The solution:
I had previously come across solutions suggesting that the reason for the terminal being slow when using commands that involve sudo was because of a misconfiguration with the `/etc/hosts` file not having the host name in it. 
When I installed Pop!\_OS, I didn't change the hostname (the name after `$USER@` when using the average command prompt) from what was given on the initial installation.
Since I never changed the hostname, and this solution said that if you *changed* your hostname that you would have to change it in `/etc/hosts` (to check your hostname, you can just type `hostname` or `cat /etc/hostname` into the terminal) I assumed that this solution would not be applicable to me, and it didn't make any sense as to why it would.
I dislike randomly changing files if I don't understand why I'm changing them or the logical throughline of how my actions would have lead to an outcome, so I refrained from doing it until I felt I was out of options.
After applying this change, evereything worked.

I opened `/etc/hosts` and did the following:
```
127.0.0.1	localhost	VALUE_OF_HOSTNAME
::1		localhost	VALUE_OF_HOSTNAME
```

I don't understand why this works. 
More specifically, I don't understand why I never touched the hosts file prior to now, but the issue was fixed through me editing the hosts file.

I checked my Ubuntu 24.04 virtual machine, and the hosts file is slightly different to the Pop!\_OS virtual machine in the sense that the host name is in the hosts file, but appears differently, but the Pop!\_OS virtual machine hosts file has exactly the same hosts file as on my bare metal machine, and doesn't have the same long sudo issues.

Now that the issue is fixed, `sudo -k` runs as fast as expected, and I uncommmented the `brew` file.
I chose to keep Howdy uninstalled, because it still seemed to have added a consistent overhead of at least 10 seconds to `sudo -k`.

Sources:
[set -x](https://www.pullrequest.com/blog/understanding-and-using-set-x-and-set-e/)
[sudo -k](https://man7.org/linux/man-pages/man8/sudo.8.html)
[solution suggestion 1](https://askubuntu.com/questions/322514/terminal-command-with-sudo-takes-a-long-time)
[solution suggestion 2](https://github.com/pop-os/pop/issues/199)
[solution suggestion 3](https://github.com/pop-os/systemd/issues/5)
[solution suggestion 4](https://github.com/pop-os/distinst/issues/264)
[solution insight resource](https://amyangfei.me/2022/02/20/diagnose-slow-sudo/)

