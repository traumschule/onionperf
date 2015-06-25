# OnionPerf

OnionPerf is a utility to track the performance of hidden services in Tor.

OnionPerf uses multiple processes and threads to download random data
through Tor while tracking the performance of those downloads. The data is
served and fetched on localhost using two TGen (traffic generator)
processes, but is transferred through Tor using a temporary Tor Hidden
Service process and a client process. Tor control information and TGen
performance statistics are logged to disk and can be later analyzed (using
onionperf-analyze.py) to visualize changes in Tor performance over time.

For more information, see https://github.com/robgjansen/onionperf.

### Install Dependencies

  + onionperf: python, python-stem
  + tor: tor, libevent, openssl
  + tgen: git, cmake, glib, igraph

(You can build a custom Tor instead of using the package manager if you 
want, and will be able to point OnionPerf at your custom Tor binary later.)

Fedora/RedHat:

```
# for onionperf
sudo yum install python python-stem tor git gcc cmake make glib2 glib2-devel igraph igraph-devel
# for onionperf-analyze.py
sudo yum install python numpy scipy python-matplotlib
```

Ubuntu/Debian:

```
# for onionperf
sudo apt-get install python python-stem tor git gcc cmake make libglib2.0 libglib2.0-dev libigraph0 libigraph0-dev
# for onionperf-analyze.py
sudo apt-get install python python-matplotlib python-numpy python-scipy
```

### Build TGen Traffic Generator

The traffic generator currently exists in the Shadow simulator repository,
but we will build tgen as an external tool and skip building both the full 
simulator and the tgen simulator plugin.

```
git clone https://github.com/shadow/shadow.git
cd shadow/src/plugin/shadow-plugin-tgen
mkdir build
cd build
cmake .. -DSKIP_SHADOW=ON -DCMAKE_MODULE_PATH=`pwd`/../../../../cmake/
make
```

### Run OnionPerf

You will need to give onionperf the path to your custom 'tor' and 'tgen'
binary files if they do not exist in your PATH environment variable.

```
git clone https://github.com/robgjansen/onionperf.git
cd onionperf
./onionperf -h
./onionperf --tor=/home/rob/tor/src/or/tor --tgen=/home/rob/shadow/src/plugin/shadow-plugin-tgen/build/tgen
```

This will store all local state, including log files, to the `.onionperf`
directory. Let it run, and by default it will download files over Tor every
60 minutes, and log the results.

### Analyze Results

You can run the client tgen log file through `onionperf-analyze.py` to
visualize changes in performance over time.

```
python onionperf-analyze.py .onionperf/client/tgen/tgen.log
```

This will save a new PDF in the current directory.


### Implementation Notes

OnionPerf has two timers to be aware of:

The **heartbeat timer** runs in the main process. Once configuration is complete, the main process loops through the following:
  1. parse the tgen log file and count the number of completed downloads,
  1. check that all helper processes are alive,
  1. sleep for 3600 seconds,
  1. repeat.

The **download timer** runs in the tgen client download manager child process. The download manager:
  1. downloads files by cycling through the file sizes (default sizes are 50 KiB, 1 MiB, and 5 MiB) until it attempts `burst_num` total downloads,
  1. pauses for `burst_interval` seconds.

Both `burst_num` and `burst_interval` are adjustable flags to OnionPerf and are listed in the help menu (`onionperf --help`).

### Contribute

GitHub pull requests are welcome and encouraged!
