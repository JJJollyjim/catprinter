# Commands

## Command format

Each command consists of the following byte sequence:

| Byte | Description 
|------|-------------
| `0x51` | Magic number 
| `0x78` | Magic number 
| *opcode* | Command number 
| *flag* | `0x00` when sent by host, `0x01` when sent by printer
| *length* | Length of the command's parameters, in bytes
| `0x00` | Seems to always be zero
| *data* | *length* bytes' worth of command parameters
| *crc* | Standard CCITT CRC-8 of *data*
| `0xFF` | Magic number

## `0xA0` - Reverse feed

The complement to command `0xA1`. There's no reference to it in blueUtils; WerWolv seems to have discovered this through experimentation.

## `0xA1` - Forward feed

Two bytes of parameters: first byte is the number of pixel rows' worth of paper to advance, second byte is `0x00` (not, as previously thought, the high byte of a short).

## `0xA2` - Print (uncompressed)

The length of the parameters is one eighth of the print width. Cat printers have a print width of 384 dots, so they should get 48 bytes of data.

The data consists of a bitmap of a single row of pixels, from left to right, with each byte packed with the least significant bit first. A `1` bit means to **darken** that pixel, the opposite to what's typical of display formats.

TODO: explore details regarding when auto line feed does or does not happen

## `0xA3` - Device status

When sent to the printer, the parameter is a single `0x00` byte.

A notification should come back from the printer with a command code of `0xA3` and a status byte in the parameter. Each of the four low bits represents a different status flag; however, it's not clear that all four flags are used by an existing printer.

| bit | symbol | English string
|-----|-------------|---------------
| `xxxxxxx1` | `no_paper` | "No paper."
| `xxxxxx1x` | `paper_positions_open` | "The bin is open."
| `xxxxx1xx` | `too_hot` | "It's too hot, please let me rest for a while."
| `xxxx1xxx` | `no_power_please_charge` | "I'm running out of power, the print will fade, please charge first."

I don't know if more than one flag can be set at once, but blueUtils can only detect each flag if all the following flags are `0`.

As far as I've been able to tell, the GB01 only uses the low power flag. It doesn't detect an open lid or a lack of paper, and it seems to respond to overheating by turning itself off.

The messages are all translated to many different languages, and most of them have been updated since earlier versions, despite that iPrint only uses them for logging and internal messaging.

## `0xA4` - Quality(?)

One byte parameter, with unclear purpose. 

There are eight different entries for this in the command list:

| name | value
|------|------
|`quality1`|`0x31`
|`quality2`|`0x32`
|`quality3`|`0x33`
|`quality4`|`0x34`
|`quality5`|`0x35`
|`speed_thin`|`0x22`
|`speed_moderation`|`0x23`
|`speed_thick`|`0x25`

There aren't any references to the `speed` entries; I suspect they might not reflect the current version of the protocol.

blueUtils chooses an entry based on printer model and contrast setting, in the short but hard-to-follow function `BluetoothOrder.getBlackening()`. On the GB01, it uses `quality3` at all times, but it has dead code to use `quality4` for "deepen". On other known cat printers, it uses `quality2` for "thin", `quality3` for "moderation", and `quality5` for "deepen".

## `0xA6` - "Lattice" control

Eleven bytes of parameter data. There are two defined sets of values:

| symbol | bytes (hex)
|--------|------
| `printLattice` | `AA 55 17 38 44 5F 5F 5F 44 38 2C`
| `finishLattice` | `AA 55 17 00 00 00 00 00 00 00 17`

The first set gets sent before printing an image, and the second set gets sent after. I've always assumed this has something to do with auto line feed, but I haven't actually checked.

One thing they both have in common: if you skip the extremely magic-number-y `AA 55` at the beginning, the last byte is a simple sum of the other eight.

I think it's fairly notable that, despite multiple app versions, full of special cases for specific printers, phones, and frontends, these two sets of values seem to always be used verbatim.

## `0xA8` - Device info

Similar to `0xA3`, send this to the printer with a single `0x00` parameter to get a reply.

The printer will respond with a corresponding `0xA8` notification containing what appears to be the printer's firmware revision number, as a string.

## `0xA9` - Update device

One byte paramater, `0x00`. Purpose unknown.

## `0xAA` - WiFi settings

I haven't looked into this.

## `0xAE` - Flow control

blueUtils doesn't send this command; it only receives it. The parameter is one byte: `0x10` to stop transmitting, or `0x00` to resume transmitting. The stop-transmission command is meant to be followed immediately, on the packet layer, but that doesn't appear to be possible to do in most BLE APIs.

## `0xAF` - Energy

Two byte parameter: a sixteen bit integer with the low byte first.

This command tells the printer how dark to print, presumably by changing how much energy gets put into the heating element.

The blueUtils device table has three settings for this for each printer, for each of "thin", "moderation", and "deepen". The values for known cat printers, in decimal, are:

* **GB01:** `8000`, `12000`, and `17500`
* **GB02:** `8600`, `12000`, and `16000`
* **GT01:** `12000` for all three

The value for the GT01 does not appear to be taken directly from the device table, but rather, as far as I can tell, it makes up a whole other table mathematically, which it then indexes by a value it gets from an activation server.

## `0xBB` - Device ID

Used to either store or retrieve a device ID used in activation requests. 

Of known cat printers, this is only used on the GT01.

(It's also used on hypothetical cat printers GB03, GB04 and GT02.)

## `0xBD` - Speed setting

I've put the numbers here in decimal, since they look cleaner that way.

iPrint sends this command with a parameter of `25` before feeding blank paper. Before printing, it sends this command with a parameter chosen based on the printer model and whether it's in image or text mode.

This is how it works with known cat printers:

* **GB01:** `35` in image mode, `25` in text mode --- UNLESS the phone is a Huawei EML-AL00, VKY-AL00, or VTR-AL00, in which case it uses `60` for image mode, and `45` for text mode.

* **GB02:** `26` in image mode, `25` in text mode.

* **GT01:** `30` in image mode, `25` in text mode --- unless the localized string `app_info` matches `"jingzhunxuexi_wand"`, in which case it uses `55` for image mode, and `45` for text mode.

## `0xBE` - Drawing mode

One byte parameter, `0x00` for image mode, `0x01` for text mode. Note, image data gets sent to the printer in the same format in both modes --- it just tells the printer whether dithering or threshold/edge-detection is being used. Honestly, I have no idea why it needs to know.

## `0xBF` - Print (compressed)

This command is now understood,

TODO: but not by me yet.
