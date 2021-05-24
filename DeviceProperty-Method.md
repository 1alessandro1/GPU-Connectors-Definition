1. Find the `DeviceProperty` path of your GPU, with `gfxutil` that can be found [here](https://github.com/acidanthera/gfxutil/releases/download/1.80b/gfxutil-1.80b-RELEASE.zip) 

The syntax for `gfxutil` is `./gfxutil -f GFX0`, for me that I have a laptop the result is

```
./gfxutil -f IGPU
00:02.0 8086:3ea5 /PCI0@0/IGPU@2 = PciRoot(0x0)/Pci(0x2,0x0)
```

For you the easiest way to check it is to look at the `vendor-id:device-id` which is next to the `PciRoot` path. For me it's

```
8086:3ea5
```
Where `8086` is the vendor-id for intel, and `3ea5` is the `device-id` for my Whiskey-Lake iGPU. If you have a GTX760, you should look for `NVIDIA` vendor (`10de` and the device-id should be `1187` for your GPU) it can be very likely that `gfxutil` spits something like: 

```
PciRoot(0x0)/Pci(0x1,0x0)/Pci(0x0,0x0)
```
if this card is connected to the primary `PCIE X16` slot, and the motherboard makers were not drunk while developing your UEFI firmware

2. Once you found the path, it would be very helpful to have a look at your `IOReg` (if you have one, since it was working before as you said with the GTX760) so that you can identify how many connectors your GPU has and what are their properties

3. Once you know (by looking at your card) which and how many connectors you have, you can start at least to force every connector to `DP` (if this is the connector you're using right now) and once you come back to your deskop you can correctly address every connector.

Seeing an original dump from an original mac, this is the way connectors are defined:

```
	<key>PciRoot(0x0)/Pci(0x1,0x0)/Pci(0x0,0x0)</key>
	<dict>
		<key>@0,connector-type</key>
		<data>
		AAQAAA==
		</data>
        </dict>

```

(In the case above, `AAQAAA==` is the `base64` version of the hex value `00040000`, look down below for the chart of what this value means, DP)

And the value can be either:

```
<02 00 00 00>        LVDS and eDP      - Laptop displays
<10 00 00 00>        VGA               - Unsupported in 10.8 and newer
<00 04 00 00>        DisplayPort       - USB-C display-out are DP internally
<01 00 00 00>        DUMMY             - Used when there is no physical port
<00 08 00 00>        HDMI
<80 00 00 00>        S-Video
<04 00 00 00>        DVI (Dual Link)
<00 02 00 00>        DVI (Single Link)
```

And of course, you may have different 

```
	<key>@0,connector-type</key>
	<data>
	AAQAAA==
	</data>
```

yours might be different basing on what connector you're using to connect to your card. If you have something like 4 connectors, it might be helpful to enforce every connector this way (0,1,2,3), here's an example:

```
	<key>PciRoot(0x0)/Pci(0x1,0x0)/Pci(0x0,0x0)</key>
	<dict>
		<key>@0,connector-type</key>
		<data>
		AAQAAA==
		</data>
		<key>@1,connector-type</key>
		<data>
		AAQAAA==
		</data>
		<key>@2,connector-type</key>
		<data>
		AAQAAA==
		</data>
		<key>@3,connector-type</key>
		<data>
		AAQAAA==
		</data>
	</dict>
```
