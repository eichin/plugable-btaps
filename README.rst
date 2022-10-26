2022-10-26 python3 port
=======================

There are a couple of other python 3 ports out there; futurize does
90% of the work and some bytes/bytearray work covers most of the
rest.  This one exists mostly as a place to hang an explanation of
why, if you're having trouble with it, it's not actually anything
wrong with btaps itself...

However - if you're trying to use this code in Ubuntu 22.04, and
especially if you're getting "Bad address" problems (or if you get as
far as using `strace`, seeing `EFAULT` problems) it turns out that the
version of pybluez (0.23-4build1) that shipped in ubuntu 22.04 had
just missed python started to enforce use of `size_t` instead of `int`
and failed outright with

::
    SystemError: PY_SSIZE_T_CLEAN macro must be defined for '#' formats

0.23-5 in 22.10/kinetic has some early fixes, cherry picked from
Debian - but they were incomplete.

The right thing to do is grab pybluez 0.30 or later, those appear to
have cleaned this up (but are not yet in ubuntu, as of kinetic.)  If
you need to patch together something (or that's just *easier* for
you), `apt-get source` and look at `pybluez-0.23/bluez/btmodule.c`;
every place you find a `PyArg_ParseTuple` that takes an `&len`
parameter, look for the misdeclaration of that `len` as `int` and
change it to `Py_ssize_t`.  I believe just changing `sock_send` and
`sock_sendall` were enough to get *this* package working; if you need
more than that, go for 0.30 or later instead.

Links (riddle trail):
 - https://bugs.python.org/issue40943 caused https://github.com/pybluez/pybluez/issues/426
 - https://github.com/pybluez/pybluez/pull/427 landed 2022-01-02 (not in 22.04)
 - https://packages.ubuntu.com/kinetic/python3-bluez
   - http://changelogs.ubuntu.com/changelogs/pool/universe/p/pybluez/pybluez_0.23-5/changelog
   - https://www.mail-archive.com/debian-bugs-dist@lists.debian.org/msg1853933.html
   - https://launchpad.net/~satadru-umich/+archive/ubuntu/updates/+sourcepub/13645941/+listing-archive-extra

Plugable PS-BTAPS1 Library and CLI
==================================

Description
___________
This project is a library and command-line interface for communicating with and programming a `Plugable PS-BTAPS1 Bluetooth Home Automation Switch`_.

libbtaps.py 
    Serves as a Python implementation of the BTAPS protocol, either to be used directly in your programs, or to be used as a reference in developing your own implementation in a different language.
btaps.py 
    A simple command-line UI that implements all the features exposed by libbtaps.py.

::

    USAGE:   python btaps.py [Bluetooth address]
    EXAMPLE: python btaps.py 00:00:FF:FF:00:00

Implemented Functionality
_________________________
The following functions of the Plugable PS-BTAPS1 are currently present in the library:
 - Setting Switch On/Off
 - Reading current status of switch (name, on/off state, timer settings)
 - Creating, modifying and deleting timers
 - Changing the device's name
 - Updating the device's date and time to your PC's current date and time
 
TO DO
_____
The following features and items are still to come:
 - NFControl (Device proximity on/off functionality)
 - Security PIN
 - Better error handling
 - Better documentation
 - Mac OS X support

OS Support
__________
Due to PyBluez limitations, this library will currently only work on Linux and Windows systems.

Dependencies
____________
 - `Python 2.7.x`_
 - PyBluez_

Installation
____________
First, install PyBluez using the appropriate link or command for your OS:

**Windows**
    Download and install `PyBluez for Python 2.7`_

**Ubuntu/Debian**::

    sudo apt-get install python-bluez

**Fedora**::

    sudo yum install pybluez

**Arch**::

    sudo pacman -S python2-pybluez

Then, simply pip install our module:::

    pip install plugable-btaps

libbtaps Docs and Examples
__________________________
Find some usage examples and documentation for libbtaps in our `GitHub Wiki`_

Troubleshooting
_______________
When I try to pip install plugable-btaps I get a compilation error:
    This means that you have not installed PyBluez and pip is trying to compile PyBluez from source, but you don't have the necessary compilation dependencies installed on your system.
    Install PyBluez as outlined above.

.. _Plugable PS-BTAPS1 Bluetooth Home Automation Switch: http://plugable.com/products/ps-btaps1/
.. _PyBluez: https://code.google.com/p/pybluez/
.. _Python 2.7.x: https://www.python.org/
.. _PyBluez for Python 2.7: https://code.google.com/p/pybluez/downloads/detail?name=PyBluez-0.20.win32-py2.7.exe
.. _GitHub Wiki: https://github.com/bernieplug/plugable-btaps/wiki/libbtaps-Documentation-and-Examples
