## How to Build Hardware
```
$cd hardware/synthesis/icesugar
$make generate
$make compile
$icesprog bin/toplevel.bin
```

## How to Build Software
```
$cd software/standalone/blinkAndEcho
$make BSP=Ice40up5kbevn
$icesprog -o 0x100000 build/blinkAndEcho.bin
```
