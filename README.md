# Ledger Compile Environment

    git clone https://github.com/LedgerHQ/ledger-app-builder.git
    docker build -t ledger-app-builder:latest .
    # template plugin
    git clone https://github.com/LedgerHQ/app-boilerplate.git
    
# Compile your plugin in the container
    docker run --name app-boilerplate --rm -ti -v ~/c/app-boilerplate:/app -d ledger-app-builder:latest
    docker exec -it app-boilerplate make
    # will create a plugin app. app.elf in app-boilerplate/obj and app-boilerplate/bin

# Monitor Device Test
    git clone https://github.com/LedgerHQ/speculos.git
    
    # MacOS Replace Dockerfile
    FROM ghcr.io/ledgerhq/speculos-builder:latest AS builder
    with
    FROM ghcr.io/ledgerhq/speculos-builder-aarch64:latest AS builder
    
    docker build -t speculos .
    
    docker run --name monitor-nanos-device --rm -it -v ~/c/app-boilerplate/bin:/speculos/apps -p 1234:1234 -p 5000:5000 -p 40000:40000 -d speculos --sdk 2.0 --apdu-port 40000 --seed xxxxxxxxx --display headless apps/app.elf
    
    open  http://127.0.0.1:5000/
    cla => 1 bytes => app mark 0xE0 is the repo
    ins => 1 bytes => enum {
        GET_VERSION = 0x03,     /// version of the application
        GET_APP_NAME = 0x04,    /// name of the application
        GET_PUBLIC_KEY = 0x05,  /// public key of corresponding BIP32 path
        SIGN_TX = 0x06          /// sign transaction with BIP32 path
    }
    p1 => 1 bytes => display result on device  0 => false  1 => true
    p2 => 1 bytes => router 2
    lc => 1 bytes => 01
    data => max 255 bytes => 00
    
    # GetVersion
    E00300000100 return 0100019000
    response last 2 bytes is StatusCode  0x9000 is ok
    010001 => 01 Major Version  00 => MINOR_VERSION 01 => PATCH_VERSION
    
    # GetAppName
    E00400000100 returns 426f696c6572706c6174659000
    hex(426f696c6572706c617465) => string(Boilerplate)
    
    # GET_PUBLIC_KEY
    data defines the returns path
    E00501000101 returns 41042c1ff030f26416a9da67c32a315452c40338cc2d24184386159cf300b0c1d55f72d53beff5e5dbc9dcbd19ddffa7c626791da50e814cbd5b4bce834e47415fd02053ffe5a809cdfe5ca3d4dbc32bc6cd45c82a017764262da90b065a9328e6ec1a9000
    
    
## Command APDU

See [COMMAND](https://github.com/LedgerHQ/app-boilerplate/blob/master/doc/COMMANDS.md)
- `Lc` length is always exactly 1 byte
- No `Le` field in APDU command
- Maximum size of APDU command is 260 bytes: 5 bytes of header + 255 bytes of data
- Maximum size of APDU response is 260 bytes: 258 bytes of response data + 2 bytes of status word

| Field name | Length (bytes) | Description |
| --- | --- | --- |
| CLA | 1 | Instruction class - indicates the type of command |
| INS | 1 | Instruction code - indicates the specific command |
| P1 | 1 | Instruction parameter 1 for the command |
| P2 | 1 | Instruction parameter 2 for the command |
| Lc | 1 | The number of bytes of command data to follow (a value from 0 to 255) |
| CData | var | Command data with `Lc` bytes |

    
