# Laelaps


## Notice
1. To test a new firmware, create a directory in the ```proj``` directory. The general naming rule is ```proj_devicename_appname```.
2. The last worked version of Angr is `8.19.2.4`.


## Installation
Make sure Python 3 is used.

1. Install [Python virtualenvwrapper](https://virtualenvwrapper.readthedocs.io/en/latest/) and create a virtual environment `laelaps`. All the following steps are operated inside this virtual environment. So, execute this command first.
```
$ workon laelaps
```
2. Build qemu-3.0.0 and put `qemu-system-arm` in PATH.
```
$ sudo apt-get build-dep -y qemu
$ mkdir ../build && cd ../build
$ ../qemu-3.0.0/configure --python=python3 --target-list="arm-softmmu" --disable-vnc --disable-curses --disable-sdl --disable-hax --disable-rdma --enable-debug
```
3. Download [ARM GCC toolchain](https://developer.arm.com/open-source/gnu-toolchain/gnu-rm/downloads) and put it in PATH.
4. Install angr. 
   - Install angr dependencies.
     ```
     $ pip install angr==8.19.2.4; pip uninstall angr
     ```
   - Download angr's source code from PyPI and install angr from the source code.
     ```
     $ wget https://files.pythonhosted.org/packages/35/19/07442cc5789f6c40eae7ea2bd34a04402fa94f9e3d94cba0ab8354d231cf/angr-8.19.2.4.tar.gz
     $ tar xf angr-8.19.2.4.tar.gz
     $ cd angr-8.19.2.4
     $ pip install -e ./
     ```
5. Patch angr. Apply the following patch.
```
--- a/angr/state_plugins/plugin.py
+++ b/angr/state_plugins/plugin.py
@@ -20,7 +20,11 @@ class SimStatePlugin(object):
         """
         Sets a new state (for example, if the state has been branched)
         """
-        self.state = state._get_weakref()
+        from angr.state_plugins import SimStateHistory
+        if isinstance(self, SimStateHistory):
+            self.state = state._get_strongref()
+        else:
+            self.state = state._get_weakref()

     def set_strongref_state(self, state):
         pass
```
6. Install the following dependencies with `pip`.
```
numpy
pygdbmi==0.9.0.0
intervaltree
posix_ipc>=1.0.0
capstone>=3.0.4
keystone-engine
parse
configparser
npyscreen
enum34
unicorn
```
7. Install avatar2.
```
$ cd avatar2
$ pip install -e ./
```
8. Install concolic.
```
$ cd concolic
$ pip install -e ./
```
9. Run tests in `proj` directory.


## Example
To get started, here is an example of using `Laelaps` to run the firmware inside `proj_nxp_frdmk66f_adc`.

```
$ workon laelaps
$ cd proj/proj_nxp_frdmk66f_adc
$ ./driver.py
```
After a while, qemu reaches the breakpoint `0x694`, which is set up after the usage of adc peripheral. Then Laelaps can be stopped by executing the shell script in another terminal.
```
$ ./scratch/kill.sh
```

When running the firmware with `uart`, the output is stored in the file `logfiles/debug.txt`. For example, when running the firmware inside `proj_nxp_frdmk66f_rtos_hello`, the *hello world* can be output.
```
$ workon laelaps
$ cd proj/proj_nxp_frdmk66f_rtos_hello
$ ./driver.py
```
In another terminal, execute
```
$ tail -f logfiles/debug.txt
```
Then, after a while, *hello world* is output. In the end, stop laelaps by executing
```
$ ./scratch/kill.sh
```

Certain logs can be found in the following directories:
- logfiles
- myavatar