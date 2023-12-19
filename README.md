# 1M bootleg flash chip documentation

Some Chinese repro cartridges of GBA games come with 1MBit flash chips for
storing save games. These chips are different from the ones that are used in
official Nintendo games, mainly the mainline Pokemon games.
These are not compatible with this type of cartridge and they must be patched
first.

## Working patches

- unbound_v2.1.1.1_1m_bootleg_flash.bps: This is a patch that converts Pokemon
Unbound to correctly use the chip on the repro cart to save the game.
- emerald_1m_bootleg_flash.bps: Pokemon Emerald
- firered_1m_bootlet_flash.bps: Pokemon Fire Red (rev0)

## Patch investigation

These patches were reconstructed from a hack that was present on the cartridge
as purchased. Here is an overview of what changes they apply:

- routine1.bin is written into unused space on the ROM
- `<SwitchFlashBank>` is modified to call `<routine1+0x8>` in thumb mode
- this copies code into EWRAM that writes the bank number to offset `0x09000000`
(tests with the cartridge have shown that bank switching is possible even if it
doesn't happen from within EWRAM, the reason why the Chinese patch does this is
unknown)
- changes `<ReadFlashId>` to always return the ID of the Sanyo 1Mbit chip used
on official cartridges of mainline Pokemon games (`0x1362`)
- modifies `<IdentifyFlash+0x38>`, `<IdentifyFlash+0x3a>` and
`<IdentifyFlash+0x3e>`
- changes `<EraseFlashSector_MX>` to call `<routine1+0x269>`

I have not yet taken a look at the exact inner workings of `routine1`, but from
a first glance it seems like part of it might consist of ARM opcodes. At some
offsets pointers had to be adjusted, both internally, pointing inside `routine1`
and externally, pointing back at the game's flash chip code.
