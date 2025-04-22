# jackmidiola
Interface between JACK MIDI and Open Lighting Project

This application connects to jackd as a client, presenting a single MIDI input port. It connects to olad as a client and dispatches DMX512 messages base on the received MIDI messages. It can react to MIDI note-on and/or MIDI CC commands. Note-on commands are 7-bit which loses 1-bit resolution, halving the quantity of values. CC may be 7-bit or 14-bit and may use simple CC mapping with limited quantity of slots and universes or NRPN to give access to all 512 slots and up to 512 universes. Universes are sequential but the base universe may be defined (default is to start at universe 1).

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

Use `jackmidiola -h` to see command line options:

```
Usage: jackmidiola [options]

Options:
  -h --help        Show this help.
  -u --universe    First universe (default: 1).
  -n --note        Listen for MIDI note-on (disabled by default).
  -c --cc          Listen for MIDI CC (enabled by default but disabled if not specified when note-on is enabled).
  -x --exclude     Do not listen on MIDI channel (1..16 Can be provided multiple times).
  -j --jackname    Name of jack client (default: midiola)
  -m --mode        MIDI mode:
    cc7   : CC 0..127 control slots 1..128. MIDI channel = universe (default).
    cc14  : CC 0..31 (MSB) 32..63 (LSB) control slots 1..32. MIDI channel = universe. Sent when LSB received.
    nrpn7 : NRPN 0..511 control slots 1..512 in first universe, NRPN 512..1023 control second universe, etc. MIDI channel offsets universe (x32).
    nrpn14: Same as nrpn7 with 8-bit DMX data, sent when LSB is received from CC38.
  -v --version     Show version.
  -V --verbose     Set verbose level:
    0: Silent
    1: Show errors
    2: Show info (default)
    3: Show debug
```

By default, jackmidiola starts in cc7 mode, listening on all MIDI channels, ignoring MIDI note-on commands with a DMX512 universe offset of 1.

MIDI note-on and cc7 modes are similar. The MIDI channel defines the DMX512 universe, e.g. MIDI channel 1 represents DMX512 universe 1, MIDI channel 2 represents DMX512 universe 2, etc. The note or CC number defines the DMX512 slot, 1..128. The note velocity or CC value defines the DMX512 value, 0..127. This gives a very simple mechanism to access 128 slots in 16 universes with 7-bit resolution.

cc14 mode is similar to cc7 mode but limits the quantity of DMX slots to 32. It uses CC 0..31 to define the most significant 7 bits of the value and CC 32..63 to define the least significant bit. (Values > 63 set the least significant bit.) The DMX512 value is only set when the LSB MIDI command is received. This gives a simple mechanism to access 32 slots in 16 universes with 8-bit resolution.

nrpn7 mode uses MIDI NRPN commands to define the DMX512 slot. NRPN parameters 0..511 represent the DMX512 slots in the first universe. NRPN parameters 512..1023 represent the DMX512 slots in the next universe, etc. MIDI data entry CC (6), data increment (96) and data decrement (97) adjust the DMX512 value. This gives a mechanism to access all DMX512 slots of up to 512 universes with 7-bit resolution.

nrpn14 mode is similar to nrpn mode but with full 8-bit resolution. MIDI data entry CC (6) sets the most significant 7-bits and LSB data entry CC (38) sets the least significant bit. (Values > 63 set the least significant bit.) The DMX512 value is only set when the LSB MIDI command is received. This gives a mechanism to access all DMX512 slots of up to 512 universes with 8-bit resolution.

DMX512 universe may be offset with the `-u` or `--universe` option, e.g. `jackmidiola -u 10` would use DMX512 universes starting at 10.

By default, jackmidiola listens on all MIDI channels. Individual channels may be disabled by using any quantity of `-x` or `--exclude` options, e.g. `jackmidiola -x4 -x16` would listen on MIDI channels 1..3 and 4..15.

To react to MIDI note-on commands, add the `-n` or `--note` command line option, e.g. `jackmidiola -m`. This will enable note-on and disable CC. To enable both, also add the `-c` or `--cc` options, e.g. `jackmidiola -m -c`.

The amount of information shown during execution is controlled with the `-V` or `--verbose` option. By default, the configuration is shown at start-up and runtime errors are displayed. Increasing the verbose level increases the amount of output. Note that erros and debug are are sent to stderr whilst inof is sent to stdout. Verbose 0 disables all output except that created by upstream libraries, e.g. jack, ola, etc.

## Use Cases

This application was designed to add DMX512 output to zynthian but may be used where an operating system is running jackd and olad to interconnect any jackd MIDI client to olad. The author is aminable to feature requests and bug reports. Please use GitHub issues.
