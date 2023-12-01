Actual details of how the copyprotection works:

mmrgb_cpi_spi is called every time the system wants to transact data with the PIC over the SPI bus.
Bytes on the outbound packet at indices 1, 2, 3, 4, 5, 6, 7, 8 and 17 are copied and passed to a
function called cp_prepare_byte() which performs hashing based on tables that are unique to that
particular game. After a good amount of wranging, which, as the sourcecode tells us is "fakery
to throw off the trail :)", the system computes the expected response.

The PIC sends the copy protection response in its packets at index 11, mixed in with the switch
matrix data. The game then calls cp_byte() which compares the PIC's response to the expected
response computed earlier. If there's a match, it increases the confidence counter. Otherwise,
confidence is reset to 0. A value of 0x11 will not increase confidence because "0x11 is reported
with the Mono PIC, don't let this bump up confidence, just in case..."

Both rgbpic_installed() and color_is_on() check the confidence count. It is enough to neutralize
two checks in color_is_on() to force color on all the time, which is what I did. However, this
does not allow the user to control whether the colors should be disabled, in case the dastardly
pirate who applies the crack to his game wants to go back to monochrome mode. In which case,
he'd have to change rgbpic_installed() to return 1 and then crack color_is_on() in a cleaner
way. This would allow him to toggle colors on or off as needed.

The source code can be recovered using a hex editor. Search your installer image for the string:
"Let's use this for 'copy-protect' such as it is"
