# GAOMON S620 600 to 800 Hz firmware

Custom firmware for the GAOMON S620 graphics tablet. It reports at around **600 Hz**
instead of the stock rate near 290, and will go to roughly **800 Hz** if you are willing
to give up pen hover height for it.

Flash it in your browser at **<https://tablet.mikuuu.xyz>**. Nothing to install.

> This repository is the writeup. The flasher itself is a web page and is not
> distributed here, so there is no code to clone.

## Measured

| build | rate | noise X | noise Y |
| --- | --- | --- | --- |
| stock | 293.7 Hz | 2.66 | 4.21 |
| settle-only patches, published elsewhere | ~530 Hz | not measured | not measured |
| default, 20 drive periods, smoothing off | 610.2 Hz | 3.63 | 3.63 |
| 10 drive periods, smoothing on | 777.8 Hz | 0.61 | 0.61 |
| 8 drive periods, smoothing on | 836.3 Hz | 0.61 | 1.21 |

Noise is in raw tablet units, detrended so the pen can be moving. Ten units is roughly
2 px across a 50 mm mapped area. Every figure comes from one device running
`OEM02_T18e_241030`, so treat them as data points rather than a spec.

Two caveats worth knowing. 0.61 is the lowest value the estimator can express, so those
rows mean "at the floor" rather than an exact number. And the default configuration,
20 drive periods with smoothing left on, has not been measured directly; it sits
somewhere between the 610 row and the 778 row.

## Two choices, and they are independent

**Smoothing.** The tablet has its own position filter, and you can keep it or bypass it.
Keeping it is the default. It is speed adaptive: it reads how far the pen moved each
report and sets its weight from that, filtering hardest when the pen is slow and backing
off when you move fast. The whole chain costs under 6.8&micro;s of a 3398&micro;s report,
about 2.5 Hz, so it is dropped for latency and never for rate.

Because the delay is counted in samples rather than milliseconds, doubling the report
rate halves what the filter costs you. At 610 Hz it adds about 1.6 ms when the pen is
moving fast, against 3.4 ms for the same filter on stock firmware.

Every build published before 2026-09-20 bypassed it unconditionally, which is where the
"shakier lines than stock" reports came from.

**Report rate.** 610 Hz is the default and the only setting with real use behind it. The
other two shorten the drive burst further, which buys rate and spends hover height.

## Why it gets past 530 Hz

Every report begins by driving a burst into a coil to power the pen, which has no
battery. The S620 does this with twelve routines that bit-bang a square wave on PA15,
each opening with `movs r4,#29`, the number of drive periods.

At 72 MHz those bursts are about 979&micro;s of an 1838&micro;s fixed path. Published
patches only shorten the four analog settle waits, which runs out near 530 Hz because by
then most of the report period is the bursts themselves. Shortening the bursts is what
goes underneath that ceiling.

Accuracy is spent in the settle waits, not the bursts. One of the four is the analog
pre-amplifier settling window, and sampling before it settles produced nearly all the
position noise earlier builds paid. Holding it at 6&micro;s, a fifth of stock, costs
about 16 Hz and returns the noise to factory levels.

## What it costs you

**Hover range, and this is the one that matters.** The pen is passive and takes all its
power from the burst, and coupling falls off steeply with height. 20 drive periods is 69%
of stock energy and the cursor can already drop out where stock firmware still tracked.
10 periods is 34% and 8 is 28%.

None of this shows up in the noise figures, because those were recorded with the pen on
or near the surface. If you hover rather than rest the pen, stay on the default, and go
up to 24 or 29 drive periods in Advanced if you need more height than that.

Pressure keeps its own smoothing on purpose. Without it the tip-down threshold chatters
on light contact and the pen clicks on and off while you are barely touching the surface.

## Safety

Erase, prove the whole span reads back as `0xFF`, write, then compare every byte. The
bootloader and the calibration page are never written, so a bad application image is
always recoverable with the express key combo. The patch is built only from a verified
factory image, and every site is checked against the bytes it expects to replace before
anything is written.

It can still brick your tablet. You accept that risk yourself, and I am not responsible
for damage to your hardware.

## More detail

The full writeup is at <https://tablet.mikuuu.xyz/notes.html>: the twelve excitation
routines and their addresses, the four settle sites and how often each fires per report,
the timing model and where it stops being trustworthy, and how the profile was derived by
static analysis for other firmware builds.

## Credits

Settle immediates and the filter-bypass sites come from the published S620 work by
[catears124](https://github.com/catears124), MIT licensed. The drive-burst finding, the
timing model, the measurement tooling and the flasher are mine.

## Licence

Source-available, not open source. Read it and run it on your own hardware. Do not
redistribute or mirror it.

---

By **miku**. Support and reports in the Gaomon S620 thread in
[Tablet Firmware](https://discord.gg/eTu5U98h8T).
