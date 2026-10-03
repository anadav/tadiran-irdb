# tadiran-irdb

Flipper Zero `.ir` files for **Tadiran** air conditioners, using the **TAC 297** remote.
They might also work with the GP08 V2 remote, but that hasn't been tested.

The folders follow the [Flipper-IRDB](https://github.com/Lucaslhm/Flipper-IRDB) layout
(`ACs/<Brand>/`). That means the repo can be browsed by Flipper-IRDB-aware apps such as
[IR Blaster](https://github.com/iodn/android-ir-blaster) (GitHub Store), and the files can be
copied straight to a Flipper Zero.

## Files

| File | Signals | Contents |
|------|--------:|----------|
| [`ACs/Tadiran/Tadiran_1345.ir`](ACs/Tadiran/Tadiran_1345.ir) | 9 | `Off`, plus Cool and Heat at 20/22/24/26 °C, auto fan |
| [`ACs/Tadiran/Tadiran_1345_Cool.ir`](ACs/Tadiran/Tadiran_1345_Cool.ir) | 61 | `Off`, plus every Cool state |
| [`ACs/Tadiran/Tadiran_1345_Heat.ir`](ACs/Tadiran/Tadiran_1345_Heat.ir) | 61 | `Off`, plus every Heat state |
| [`ACs/Tadiran/Tadiran_1345_full.ir`](ACs/Tadiran/Tadiran_1345_full.ir) | 301 | `Off`, plus every mode × fan × temperature |

An AC remote sends its whole state (mode, fan speed and temperature) in every button press.
That's why there is one signal per combination rather than separate up/down buttons.

Signal names have the form `<Mode>_<Fan>_<Temp>`, for example `Cool_Auto_24` or
`Heat_Low_20`:

- Mode: `Cool`, `Heat`, `Dry`, `Fan` (fan only), `Auto` (heat/cool)
- Fan: `Low`, `Mid`, `High`, `Auto`
- Temp: 16–30 °C

All signals are raw captures at 38 kHz.

## Using with IR Blaster (Android)

1. Go to Settings → GitHub Store, add `https://github.com/anadav/tadiran-irdb`, and open
   `ACs/Tadiran`.
2. Import `Tadiran_1345.ir` as a new remote. IR Blaster imports every signal in the file, so
   start with the compact file. To add more buttons to that remote, import the Cool or Heat
   file with "Add buttons to existing remote".
3. Test `Off` first, while the AC is running.

## Source and known quirks

The codes come from [SmartIR](https://github.com/smartHomeHub/SmartIR) (MIT),
file [`codes/climate/1345.json`](https://github.com/smartHomeHub/SmartIR/blob/5ec5215398c97abb3a1763880690f1e6718b7506/codes/climate/1345.json).
SmartIR records them as Broadlink packets; [`smartir_to_flipper.py`](smartir_to_flipper.py)
converts them to µs timings without changing them.

These quirks come from the source data and are kept unchanged:

- In `Dry` mode the four fan speeds send the same code at each temperature. The fan setting
  probably doesn't apply in dry mode.
- Six SmartIR captures are broken. Decoding the protocol (IRremoteESP8266's
  [Amcor](https://github.com/crankyoldgit/IRremoteESP8266/blob/master/src/ir_Amcor.h)
  format) shows:
  - `Cool_Auto_17`, `Auto_Auto_20` and `Fan_Mid_23` are bit-shifted and fail the checksum,
    so the AC ignores them.
  - `Cool_High_17` and `Cool_High_18` are swapped.
  - `Cool_High_28` actually sends 29 °C.
- `Fan_Auto_*` actually sends Low fan; the real remote has no Auto speed in Fan mode.

[tadiran-remote](https://github.com/anadav/tadiran-remote) builds the codes from the decoded
protocol instead, so it doesn't have these problems.

## Regenerating

```sh
./smartir_to_flipper.py                     # fetches the pinned SmartIR file
./smartir_to_flipper.py --source 1345.json  # or use a local copy
```

The script needs only the Python 3 standard library. It reads the SmartIR file at a pinned
commit, so the output is byte-identical every time. Before writing anything it runs these
checks:

- each file has the expected number of signals;
- `Off` starts with roughly `8500 4560 1675 558` µs;
- every signal has an even number of timings and stays under Flipper's 1024-timing limit;
- every file stays under IR Blaster's 512 KB import limit;
- every file parses correctly with a copy of IR Blaster's `.ir` reader.

## License

MIT. See [LICENSE](LICENSE). The IR codes are © SmartIR contributors.
