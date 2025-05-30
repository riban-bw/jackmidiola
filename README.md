# jackmidiola
Interface MIDI via JACK to DMX512 via Open Lighting Project.

This application connects to `jackd` as a client, presenting a single MIDI input port. It connects to `olad` as a client and dispatches DMX512 messages to OLA, based on the received MIDI messages. It can react to MIDI note-on and/or MIDI CC commands. Note-on commands are 7-bit, which loses 1 bit of resolution, halving the number of values. CC may be 7-bit or 14-bit and may use simple CC mapping with a limited number of slots and universes, or NRPN to give access to all 512 slots and up to 512 universes. Universes are sequential, but the base universe may be defined (default is to start at universe 1).

## Dependencies

This application interfaces with `jackd` and `olad`, so both need to be installed and running. Tested with jackdmp 1.9.21 and olad 0.10.9.

The application is written in the C++ programming language (because ola bindings are for C++, not C) but you may notice some C style coding methods, e.g. use of fixed size buffers, use of printf, etc. (I would have written in C if the ola library had C bindings.)

Build libraries required:

- jack
- ola
- olacommon
- protobuf

On Debian based systems, `sudo apt install libjack-jackd2-dev libola-dev`.

Of course you also need a c++ compiler and supporting libraries. On Debian based systems, `sudo apt install build-essential`.

## Building

CMake is used to build the project, but a bash script is included to further simplify the process.

To use CMake:

- Create a directory called `build`
- Change to the `build` directory
- Run `cmake ..`
- Run `make`
- Run `make install` to install.

To use the Bash script, simply run `build.sh`.

The executable file `jackmidiola` will be created in the build directory.

## Usage

Running `jackmidiola` without command-line parameters will start the application using the default configuration. It will listen for JACK MIDI commands on its MIDI input port and send messages to OLA.

Use `jackmidiola -h` to see command-line options:

```
Usage: jackmidiola [options]

Options:
  -h --help        Show this help.
  -u --universe    First universe (default: 1).
  -n --note        Listen for MIDI note-on (disabled by default).
  -o --noteoff     Listen for MIDI note-off (disabled by default).
  -c --cc          Listen for MIDI CC (enabled by default but disabled if not specified when note-on is enabled).
  -x --exclude     Do not listen on MIDI channel (1..16). Can be provided multiple times.
  -j --jackname    Name of JACK client (default: midiola).
  -m --mode        MIDI mode:
    cc7   : CC 0..127 control slots 1..128. MIDI channel = universe (default).
    cc14  : CC 0..31 (MSB), 32..63 (LSB) control slots 1..32. MIDI channel = universe. Sent when LSB received.
    nrpn7 : NRPN 0..511 control slots 1..512 in first universe, NRPN 512..1023 control second universe, etc. MIDI channel offsets universe (×32).
    nrpn14: Same as nrpn7 with 8-bit DMX data, sent when LSB is received from CC38.
  -v --version     Show version.
  -V --verbose     Set verbose level:
    0: Silent
    1: Show errors
    2: Show info (default)
    3: Show debug
```

By default, `jackmidiola` starts in `cc7` mode, listening on all MIDI channels, ignoring MIDI note-on commands, with a DMX512 universe offset of 1.

MIDI note-on and `cc7` modes are similar. The MIDI channel defines the DMX512 universe, e.g., MIDI channel 1 represents DMX512 universe 1, channel 2 represents universe 2, and so on. The note or CC number defines the DMX512 slot (1..128). The note velocity or CC value defines the DMX512 value (0..127). This provides a simple mechanism to access 128 slots in 16 universes with 7-bit resolution.

`cc14` mode is similar to `cc7` mode but limits the number of DMX slots to 32. It uses CC 0..31 to define the most significant 7 bits of the value and CC 32..63 to define the least significant bit. (Values > 63 set the least significant bit.) The DMX512 value is only set when the LSB MIDI command is received. This provides a simple mechanism to access 32 slots in 16 universes with 8-bit resolution.

`nrpn7` mode uses MIDI NRPN commands to define the DMX512 slot. NRPN parameters 0..511 represent the DMX512 slots in the first universe. Parameters 512..1023 represent the slots in the next universe, etc. MIDI Data Entry CC (6), Data Increment (96), and Data Decrement (97) adjust the DMX512 value. This allows access to all DMX512 slots of up to 512 universes with 7-bit resolution.

`nrpn14` mode is similar to `nrpn7`, but with full 8-bit resolution. MIDI Data Entry CC (6) sets the most significant 7 bits, and LSB Data Entry CC (38) sets the least significant bit. (Values > 63 set the least significant bit.) The DMX512 value is only set when the LSB MIDI command is received. This allows access to all DMX512 slots of up to 512 universes with 8-bit resolution.

The DMX512 universe may be offset using the `-u` or `--universe` option. For example, `jackmidiola -u 10` would start at universe 10.

By default, `jackmidiola` listens on all MIDI channels. Individual channels may be excluded using any number of `-x` or `--exclude` options. For example, `jackmidiola -x4 -x16` would exclude channels 4 and 16, listening on 1..3 and 5..15.

To react to MIDI note-on commands, add the `-n` or `--note` option, e.g., `jackmidiola -m`. This enables note-on and disables CC. To enable both, also add the `-c` or `--cc` option, e.g., `jackmidiola -m -c`. Note-off is ignored unless `-o` or `--noteoff` option is specified in which case, MIDI note-off commands will send value 0 to the corresponding DMX512 slot.

The amount of information shown during execution is controlled with the `-V` or `--verbose` option. By default, the configuration is shown at startup and runtime errors are displayed. Increasing the verbosity level increases the amount of output. Note that errors and debug messages are sent to `stderr`, while info is sent to `stdout`. Verbose level 0 disables all output except that generated by upstream libraries, such as JACK and OLA.

## Use Cases

This application was designed to add DMX512 output to Zynthian but may be used wherever an operating system is running `jackd` and `olad` to interconnect any JACK MIDI client with `olad`. The author is amenable to feature requests and bug reports. Please use [GitHub issues](https://github.com/riban-bw/jackmidiola/issues).

## Usueful Notes

I use Debian based distributions which include OLA as an installable package. To get OLA running and configured:

- To install OLA: `sudo apt install ola`.
- To start OLA: `sudo systemctrl start olad`
- To configure, point a web browser at http://hostname:9090/new/#. This shows the _new_ interface that has the advantage over the old one that you can disable plugins.
- Click on the "Plugins" tab then enable required plugins and disable any conflicting plugings, e.g. to get a FTDI USB interface working, enable FTDI USB DMX and disable Serial USB, Enttec Open DMX, StageProfi. This seems to be because they all use /dev/ttyUSB0 (although that port does not actually appear so maybe it captures and renames it). If you need to use a mix of these plugins, you will need to edit their configurations, e.g. /etc/ola/ola-ftdidmx.conf.
- Click on the "Universes" tab then add a universe. It is common practice to start with universe 1 which is the default used by _jackmidiola_. The universe has a Id (integer), Name (any text) and a plugin (select from checkboxes). The plugin selection defines which physical interface (or network protocol) to use for that universe. If you leave the name blank, it defaults to "Universe 1". I currently only have a single FTDI USB device so have not tested with multiple universes.

The OLA web interface provides monitoring and control of DMX512 devices. Selecting the required universe from the "Universes" tab displays a set of tabs including:

- Overview - shows the value of each DMX512 slot.
- Faders - shows the value of each DMX512 slot and a fader to control the value, paged to fit on the screen. Note that the faders are old drawbar / quadrant fader style, drag down to increase value.
- Settings - change the input and output ports used. You can enable multiple output ports to send duplicate data to multiple devices.

To remove a universe you disable all ports in the settings and save. This is not intuitive and it is not evident that the universe has been removed until you access the universes tab again.

## Licensing

The MIT license was chosen for this project to allow wide usage. The MIT license is compatible with a broad range of licenses and use cases. Please observe the license conditions. It's appreciated (though not required) to acknowledge the project, so feel free to mention it.
