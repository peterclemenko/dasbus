# dasbus
This DBus library is written in Python 3, based on GLib and inspired by pydbus. Find out more in
the [documentation](https://dasbus.readthedocs.io/en/latest/).

The code used to be part of the [Anaconda Installer](https://github.com/rhinstaller/anaconda)
project. It was based on the [pydbus](https://github.com/LEW21/pydbus) library, but we replaced
it with our own solution because its upstream development stalled. The dasbus library is
a result of this effort.

[![Build Status](https://travis-ci.com/rhinstaller/dasbus.svg?branch=master)](https://travis-ci.com/rhinstaller/dasbus)
[![Documentation Status](https://readthedocs.org/projects/dasbus/badge/?version=latest)](https://dasbus.readthedocs.io/en/latest/?badge=latest)
[![codecov](https://codecov.io/gh/rhinstaller/dasbus/branch/master/graph/badge.svg)](https://codecov.io/gh/rhinstaller/dasbus)

## Requirements

* Python 3.6+
* PyGObject 3

You can install [PyGObject](https://pygobject.readthedocs.io) provided by your system or use PyPI.
The system package is usually called `python3-gi`, `python3-gobject` or `pygobject3`. See the
[instructions](https://pygobject.readthedocs.io/en/latest/getting_started.html) for your platform
(only for PyGObject, you don't need cairo or GTK).

The library is known to work with Python 3.8, PyGObject 3.34 and GLib 2.63, but these are not the
required minimal versions.

## Installation

Install the package from [PyPI](https://pypi.org/project/dasbus/). Follow the instructions above
to install the required dependencies.

```
pip3 install dasbus
```

Or install the RPM package on Fedora 31+.

```
sudo dnf install python3-dasbus
```

Or install the DEB package on Ubuntu 22.04+ and Debian 11+.

```
sudo apt install python3-dasbus
```

## Examples

Show the current hostname.

```python
from dasbus.connection import SystemMessageBus
bus = SystemMessageBus()

proxy = bus.get_proxy(
    "org.freedesktop.hostname1",
    "/org/freedesktop/hostname1"
)

print(proxy.Hostname)
```

Send a notification to the notification server.

```python
from dasbus.connection import SessionMessageBus
bus = SessionMessageBus()

proxy = bus.get_proxy(
    "org.freedesktop.Notifications",
    "/org/freedesktop/Notifications"
)

id = proxy.Notify(
    "", 0, "face-smile", "Hello World!",
    "This notification can be ignored.",
    [], {}, 0
)

print("The notification {} was sent.".format(id))
```

Handle a closed notification.

```python
from dasbus.loop import EventLoop
loop = EventLoop()

from dasbus.connection import SessionMessageBus
bus = SessionMessageBus()

proxy = bus.get_proxy(
    "org.freedesktop.Notifications",
    "/org/freedesktop/Notifications"
)

def callback(id, reason):
    print("The notification {} was closed.".format(id))

proxy.NotificationClosed.connect(callback)
loop.run()
```

Asynchronously fetch a list of network devices.

```python
from dasbus.loop import EventLoop
loop = EventLoop()

from dasbus.connection import SystemMessageBus
bus = SystemMessageBus()

proxy = bus.get_proxy(
    "org.freedesktop.NetworkManager",
    "/org/freedesktop/NetworkManager"
)

def callback(call):
    print(call())

proxy.GetDevices(callback=callback)
loop.run()
```

Define the org.example.HelloWorld service.

```python
class HelloWorld(object):
    __dbus_xml__ = """
    <node>
        <interface name="org.example.HelloWorld">
            <method name="Hello">
                <arg direction="in" name="name" type="s" />
                <arg direction="out" name="return" type="s" />
            </method>
        </interface>
    </node>
    """

    def Hello(self, name):
        return "Hello {}!".format(name)
```

Define the org.example.HelloWorld service with an automatically generated XML specification.

```python
from dasbus.server.interface import dbus_interface
from dasbus.typing import Str

@dbus_interface("org.example.HelloWorld")
class HelloWorld(object):

    def Hello(self, name: Str) -> Str:
        return "Hello {}!".format(name)

print(HelloWorld.__dbus_xml__)
```

Publish the org.example.HelloWorld service on the session message bus.

```python
from dasbus.connection import SessionMessageBus
bus = SessionMessageBus()
bus.publish_object("/org/example/HelloWorld", HelloWorld())
bus.register_service("org.example.HelloWorld")

from dasbus.loop import EventLoop
loop = EventLoop()
loop.run()
```

See more examples in the [documentation](https://dasbus.readthedocs.io/en/latest/examples.html).

## Inspiration

Look at the [complete examples](https://github.com/rhinstaller/dasbus/tree/master/examples) or
[DBus services](https://github.com/rhinstaller/anaconda/tree/master/pyanaconda/modules) of
the Anaconda Installer for more inspiration.
