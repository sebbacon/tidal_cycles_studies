# Getting started

It's worth reading [this page](https://tidalcycles.org/docs/getting-started/tidal_start/) to start, which outlines how you interact with the system. In brief:

- You type stuff in an editor and eval lines one at a time
- The editor has hooks into the Tidal Cycles library
- Which plays sounds in SuperDirt
- Which is a synth/sample thing running in SuperCollider

I followed the instructions (macos) in the [official docs](https://tidalcycles.org/docs/) but it didn't get me far.

It got me as far as installing `cabal`, which is the `pip` of Haskell, but failed because it expected the Pulsar edited to be installed (the docs claim the bootstrap command does this, but it didn't).

It _did_ successfully install SuperCollider, which is one of the dependencies.

So then I installed Tidal Cycles like this:

    cabal install --lib tidal --package-env .

(the last 2 args set up something like a virtual env in the current directory, as far a I understand it).

Then I could start supercollider in the terminal:

    sclang

It doesn't like the fact that bluetooth mikes are a different sample rate from speakers. For now I turn off input like this at the sclang prompt:

    Server.default.options.numInputBusChannels = 0;

Or, easier, is to remember to select the builtin mike as the current input device in the system sounds settings.

Sometimes I've had to select the headphones as the output device, too:

    Server.default.options.device = "3";

If SuperDirt isn't already installed, you can do it from the sclang prompt like this:

    include("SuperDirt");

Now I can start the server:

    SuperDirt.start;
    s.boot

# Configure VS Code

Install the Haskell highlighting extension; and then associate the tidal extension with that in `settings.json`:

    "files.associations": {
        "*.tidal": "haskell"
    },

The VS Code extension has a pretty cool sample browser built-in; you have to tell it where the SuperDirt samples are. For me it looked like this:

    "tidalcycles.sounds.paths": [
        "/Users/sebbacon/Library/Application Support/SuperCollider/downloaded-quarks/Dirt-Samples"
    ]

So altogether it's these settings:

    {
        "settings": {
            "tidalcycles.sounds.paths": [
                "/Users/sebbacon/Library/Application Support/SuperCollider/downloaded-quarks/Dirt-Samples"
            ]
        },
        "files.associations": {
            "*.tidal": "haskell"
        },
    }

# Configure SuperCollider startup

My startup file for supercollider is at `~/Library/Application Support/SuperCollider/startup.scd`. To locate it, the docs say you should go via _File > Open startup file_ in the SuperCollider IDE (the non-cli app).

Mine is currently the one suggested in the startup docs, plus an enforced sampleRate and the input turned off:

    (
    s.reboot { // server options are only updated on reboot
        // configure the sound server: here you could add hardware specific options
        // see http://doc.sccode.org/Classes/ServerOptions.html
        s.options.numBuffers = 1024 * 256; // increase this if you need to load more samples
        s.options.memSize = 8192 * 32; // increase this if you get "alloc failed" messages
        s.options.numWireBufs = 64; // increase this if you get "exceeded number of interconnect buffers" messages
        s.options.maxNodes = 1024 * 32; // increase this if you are getting drop outs and the message "too many nodes"
        s.options.numOutputBusChannels = 2; // set this to your hardware output channel size, if necessary
        s.options.numInputBusChannels = 0; // set this to your hardware output channel size, if necessary
        s.options.sampleRate = 44100;
        // boot the server and start SuperDirt
        s.waitForBoot {
            ~dirt = SuperDirt(2, s); // two output channels, increase if you want to pan across more channels
            ~dirt.loadSoundFiles;   // load samples (path containing a wildcard can be passed in)
            // for example: ~dirt.loadSoundFiles("/Users/myUserName/Dirt/samples/*");
            // s.sync; // optionally: wait for samples to be read
            ~dirt.start(57120, 0 ! 12);   // start listening on port 57120, create two busses each sending audio to channel 0

            // optional, needed for convenient access from sclang:
            (
                ~d1 = ~dirt.orbits[0]; ~d2 = ~dirt.orbits[1]; ~d3 = ~dirt.orbits[2];
                ~d4 = ~dirt.orbits[3]; ~d5 = ~dirt.orbits[4]; ~d6 = ~dirt.orbits[5];
                ~d7 = ~dirt.orbits[6]; ~d8 = ~dirt.orbits[7]; ~d9 = ~dirt.orbits[8];
                ~d10 = ~dirt.orbits[9]; ~d11 = ~dirt.orbits[10]; ~d12 = ~dirt.orbits[11];
            );
        };

        s.latency = 0.3; // increase this if you get "late" messages
    };
    );
