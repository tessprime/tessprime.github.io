---
title: "Extcap"
date: 2023-10-15T12:26:52-07:00
draft: true
---

So the Wireshark explicit documentation on extcap is somewhat sparse,
so I'm writing down my own investigation on how the protocol works.

For reference, here is the Wireshark documentation:

https://www.wireshark.org/docs/man-pages/extcap.html
https://www.wireshark.org/docs/wsdg_html_chunked/ChCaptureExtcap.html

Extcap is intended to provide a way to create Wireshark interfaces that
are provided by an external program. Wireshark will invoke all programs
in the wireshark extcap directories at startup and ask it to list it's interfaces.
<!--more-->
So I've written a simple python program that is systematically writing
down what is being passed to it. There is an example python program which
I'm using to get a basic idea of what the responses need to be:

https://gitlab.com/wireshark/wireshark/-/blob/master/doc/extcap_example.py

## Finding Extcap programs

Wireshark explores a number of extcap directories (which can be found in
`Help>About Wireshark>Folders` panel). It goes through each file in the folder and
applies the rule "is the file executable?". If it is, then wireshark will attempt
to execute it with parameters.

On both Linux and Windows, Wireshark relys on gtk's glib (which has also
been ported to windows) to determine if a file is executable:

https://sourcegraph.com/gitlab.com/wireshark/wireshark/-/blob/extcap.c?L236-260

The documentation for the relevant function, `g_file_test` is here:

https://docs.gtk.org/glib/func.file_test.html

and states

> On Windows, there are no symlinks, so testing for G_FILE_TEST_IS_SYMLINK will always return FALSE.
> Testing for G_FILE_TEST_IS_EXECUTABLE will just check that the file exists and its name indicates
> that it is executable, checking for well-known extensions and those listed in the PATHEXT environment variable.

So Windows relies on `PATHEXT`, which on my machine is `.COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC`. So
a powershell script will not work unless I add it to `PATHEXT`. This explains the conflicting reports as to whether `.py` works
or not. Anyhow, relying on `.bat` seems to the sane thing. To ensure that the correct path gets used, you do
something like

```
echo @off
python %~dp0python_script.py
```

Where `%~dp0` is windows batch for "Take arg %0, and expand it to a full path". Arg 0 is the full path
to the batch file.

So all `.bat` and `.exe` files in the extcap directories will be found and executed.

## Invocation
Once a file has been determined to be an "executable", the program is invoked with the following arguments:
```sh
['C:\\Users\\froeb\\AppData\\Roaming\\Wireshark\\extcap\\test.py', '--extcap-interfaces', '--extcap-version=4.0']
```
The wireshark docs give a good summary of what this response should look like (overstdin).

If successful, this is then immediately followed with
```sh
['C:\\Users\\froeb\\AppData\\Roaming\\Wireshark\\extcap\\test.py', '--extcap-config', '--extcap-interface', 'TestExample1']
```
and
```sh
['C:\\Users\\froeb\\AppData\\Roaming\\Wireshark\\extcap\\test.py', '--extcap-dlts', '--extcap-interface', 'TestExample1']
```
If you then double click it, then the progam is invoked 3 times, once again calling `--exctcap-config` and `--extcap-dtls`
but this time actually invoking with
```sh
['C:\\Users\\froeb\\AppData\\Roaming\\Wireshark\\extcap\\test.py', '--capture', '--extcap-interface', 'TestExample1', '--fifo', '\\\\.\\pipe\\wireshark_extcap_TestExample1_20231015130917']
```
The `fifo` file is expected to be written to; Wireshark will simply wait for pcap entries to be written to it (or until the program exits).
If you attempt to read from the fifo, the python program will just block; I'm assuming that Wireshark might sometimes write to it, but
am unsure how that signalling works at the moment.
