iperf-reliable: Adding reliable data transfer service to iperf3
==============================================================

Authors: 
- Hamid Anvari (hanvari@ualberta.ca)
- Paul Lu (paullu@ualberta.ca)

Summary
-------

iperf-reliable is an augmentation of the well-known 
iperf3
network benchmarking tool,
adding a reliable file transfer service.

The original iperf3 includes a `-F` switch allowing 
to read from, or write to, a file for a benchmarking test.
However, that is not a reliable transfer service.

iperf-reliable introduces the following features:
- Reliable data transfer
- Partial data transfer through seeking a location in the source file
- Appending the received data to the end of the destination file (rather than overwriting the whole file)

The reliable and partial data transfer 
facilities are only limited to the case
of single data stream.
It does not account for parallel data streams.

A few modifications and adjustments to
enable accurate byte counting and file transfer
have already been contributed to the original 
iperf3 repository (
[#1036](https://github.com/esnet/iperf/pull/1112),
[#1113](https://github.com/esnet/iperf/pull/1113),
[#1114](https://github.com/esnet/iperf/pull/1114),
[#1115](https://github.com/esnet/iperf/pull/1115),
[#1116](https://github.com/esnet/iperf/pull/1116)
).
However, the reliable and partial file transfer 
services were not directly targeted to the scope of
the iperf3 project, hence we made them available here
as a separate open-source project.


Building iperf-reliable
---------------

The build process for iperf-reliable is identical
to original iperf3 benchmarking tool.
Please follow the instructions from official iperf3 
repository here:
https://github.com/esnet/iperf


Test Cases / How to Run
----------

For all test cases the following assumptions are made: 
- two nodes with hostnames `sender` and `receiver` that are connected over a network path.
- `sender` has a sufficiently large (e.g. 14GB) data file readable and ready for transfer located at `/dev/shm/src.file`
- `receiver` has `/dev/shm/` writable with sufficient space

### Case#1 - Default behavior to be unreliable data transfer!

```bash
# start server
receiver$ ./iperf3 -s -F /dev/shm/dst.file -V -d
# send file
sender$ ./iperf3 -c receiver -F /dev/shm/src.file -d
# get checksums and compare. assert non-equal
sender$ md5sum /dev/shm/src.file 
receiver$ md5sum /dev/shm/dst.file
# cleanup
receiver$ rm /dev/shm/dst.file
```

### Case#2 - Full (reliable) data/file transfer using -r switch

```bash
# start server
receiver$ ./iperf3 -s -F /dev/shm/dst.file -V -d -r
# send file
sender$ ./iperf3 -c receiver -F /dev/shm/src.file -d
# get checksums and compare. assert equal
sender$ md5sum /dev/shm/src.file 
receiver$ md5sum /dev/shm/dst.file
# cleanup
receiver$ rm /dev/shm/dst.file
```

### Case#3 - Partial file transfer with time-interval (-t) in reliable mode (-r)

```bash
# start server
receiver$ ./iperf3 -s -F /dev/shm/dst.file -V -d -r
# send file
sender$ ./iperf3 -c receiver -F /dev/shm/src.file -t5 -d
# make a copy of src.file and truncate its size to match the size transferred for server (in output of last command)
sender$ cp /dev/shm/src.file /dev/shm/src.partial
sender$ truncate --size=<bytes-count> /dev/shm/src.partial
# get checksums and compare. assert equal
sender$ md5sum /dev/shm/src.partial 
receiver$ md5sum /dev/shm/dst.file
# cleanup
sender$ rm /dev/shm/src.partial
receiver$ rm /dev/shm/dst.file
```

### Case#4 - Partial file transfer with seeking in file (-F name,seek#)

```bash
# start server
receiver$ ./iperf3 -s -F /dev/shm/dst.file -V -d -r
# send file
sender$ ./iperf3 -c receiver -F /dev/shm/src.file,400000000 -t5 -d
# make a copy of src.file to start at byte 400000000 (as in last commnad) and compare with destination file
sender$ dd bs=400000000 skip=1 if=/dev/shm/src.file of=/dev/shm/src.trimmed
# get checksums and compare. assert equal
sender$ md5sum /dev/shm/src.trimmed
receiver$ md5sum /dev/shm/dst.file
# cleanup
sender$ rm /dev/shm/src.trimmed
receiver$ rm /dev/shm/dst.file
```

### Case#5 - Multi-part file transfer with server appending to a single file (-F name,append)

```bash
# start server
receiver$ ./iperf3 -s -F /dev/shm/dst.file,append -V -d -r
# send file, for a few seconds (not sufficient to transfer the whole file)
sender$ ./iperf3 -c receiver -F /dev/shm/src.file -t5 -d
# consequent send file, seeking by number of bytes sent in the last command, with no time or bytes termination condition (i.e. sending to the end of file)
sender$ ./iperf3 -c receiver -F /dev/shm/src.file,<bytes-count> -d
# get checksums and compare. assert equal
sender$ md5sum /dev/shm/src.file
receiver$ md5sum /dev/shm/dst.file
# cleanup
receiver$ rm /dev/shm/dst.file
```


Bug Reports
-----------

Before submitting a bug report, please make sure you're running the
latest version of the code, and confirm that your issue has not
already been fixed.  Then submit to the iperf3 issue tracker on
GitHub:

https://github.com/hanvari/iperf-reliable/issues

In your issue submission, please indicate the version of iperf3 and
what platform you're trying to run on (provide the platform
information even if you're not using a supported platform, we
*might* be able to help anyway).  Exact command-line arguments will
help us recreate your problem.  If you're getting error messages,
please include them verbatim if possible, but remember to sanitize any
sensitive information.

If you have a question about usage or about the code, please do *not*
submit an issue.  Please use one of the mailing lists for that.

Known Issues
------------

A set of known issues is maintained on the iperf3 Web pages:

...

Related Publications
-----

* to be completed


Copyright
---------

This code is distributed under a BSD style license, 
see the LICENSE file for complete information.
