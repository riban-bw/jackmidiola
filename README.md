# jackmidiola
Interface between JACK MIDI and Open Lighting Project

## Dependencies

This application interfaces with jackd and olad so both need to be installed and running.

Build libraries required:

- jack
- ola
- olacommon
- protobuf

## Building

CMake is used to build the project but a bash script is included to further simplify the process.

To use cmake:

- Create a directory called "build".
- Change to the build directory.
- Run `cmake ..`.
- Run `make`.

To use the bash script, simply run `build.sh`.

The executeable file `jackmidiola` will be created in the build directory.

## Usage

Running `jackmidiola` without command line parameters will start the application using default configuation. It will listen for JACK MIDI commands on its MIDI input port and send messages to OLAd.

Use `jackmidiola -h` to see command line options.
