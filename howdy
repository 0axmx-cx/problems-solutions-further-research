Howdy is an open-source implementation of the Windows Hello facial recognition feature to log into a user account in lieu of a password.
[[https://github.com/boltgolt/howdy/issues]]
  
As I understand it:
It is supposed to be a system-wide program installation, and on certain linux systems is available in a repository to download.
However, the installer uses python/pip. 
Python, on some linux distros, is managed by the distribution's package manager.
Due to the adoption of PEP 668, this breaks the installer when used under certain contexts/certain distros.
[[https://peps.python.org/pep-0668/]]

I have seen many offer the solution to this is to add the flag "--break-system-packages"
[[https://stackoverflow.com/questions/75602063/pip-install-r-requirements-txt-is-failing-this-environment-is-externally-mana]]
However, it is warned against for obvious reasons, and in the hopes of running a stable linux system, running a command that seems to deliberately saction the potential breaking of components seems silly.
Furthermore, I don't think I'd be able to use that in this context because pip is being called internally from the installer, so unless I changed the installer directly, I wouldn't be able to allow for that anyway.

There also seems to be a difference in how this works across distros. In Pop!OS 22.04, howdy installed without incedent. However, on Kubuntu 24.04, I ran into the following error near the end of the installation process, preventing the installation from completing fully:

``` error: externally-managed-environment

× This environment is externally managed
╰─> To install Python packages system-wide, try apt install
    python3-xyz, where xyz is the package you are trying to
    install.
```

It seems like the only way for this issue to be fixed is for the developers of howdy to implement some change in the code to change or avoid how pip is being utilized. Using a virtual environment would make sense as well generally speaking, but again, since the issue is coming from within the installer, with a program that is meant to be installed system-wide, using a virtual environment would not provide the functionality needed for howdy to run properly. 

As mentioned before, howdy installed fine on Pop!OS 22.04, but had issues with Ubuntu 24.04.
[[https://support.system76.com/articles/setup-face-recognition/]]
Thus, I think it would be interesting to investigate the differences in Python between these two, as it seems PEP 668, or the Python that has it incorportated was integrated into more recent linux distributions only.

This is a very good discussion to read and note.
[[https://discuss.python.org/t/the-most-popular-advice-on-the-internet-for-error-externally-managed-environment-is-to-force-packages-to-be-system-installed/56900/37]]
It is related to a wider issue of how people problem solve and offers ideas and insights into different perspectives on the issue. But it doesn't offer any meaningful solutions to the issue at hand. Regardless, I think it is a good read for understanding various perspectives in the computing space, and it's easy to see how this can be extrapolated to not only computing in general, but education in general. I will make a note to go more in depth about the different perspectives offered in the discussion and my thoughts on them.
