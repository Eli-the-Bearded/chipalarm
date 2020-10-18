chipalarm
=========

A GUI alarm tool for the now defunct Next Thing Co "pocketCHIP"
computer. Blog post about this project:

https://qaz.wtf/qz/blosxom/2020/10/17/alarm

About the Platform
------------------

https://en.wikipedia.org/wiki/CHIP_(computer)

The Chip was a $9 computer, the "Pocket" part was an attachment
to add a keyboard, touchscreen, battery pack, headers for GPIO
etc, and a small hacking space. Overall it cost about $60, which
was (and is) a steal compared to what a Raspberry Pi outfited as
a laptop / tablet / netbook / other portable computer costs. As
of 2020, you can still find new-old-stock on ebay for $50ish.
Unlike a Pi, Chip was low-res: no HDMI option; and has built-in
wifi and bluetooth and built-in storage. Now Pocketchip isn't
great, it's got a take-you-back-to-the-1980s pixel count
(480x272), a painfully awkward keyboard, no built-in speakers,
and poor audio volume from the audio jack. 

I modified mine adding a basic mono amplifier (PAM8230 from
Adafruit) and a speaker attached to that. I also connected one of
the GPIO pins to a standard 1/8th inch (2.5mm) audio jack, to use
with an external button.  There was ample room in the case for
the amp, speaker, and extra jack.  I did need to cut the plastic
a bit to get get the proper parts sticking out. 

Then I built a button out of a wooden box and Cherry D42 switch
with a lever that I pulled out of some device. The lever holds
the top of the box slightly open, pushing the lid down closes the
switch. Big easy to push button for sleepy alarm silencing.

The Alarm Program
-----------------

Prior to creating this, I used an old "feature" phone an an
alarm. It let me quickly turn on or off three different alarm
settings. I liked that, but wanted a way to set a week of those
at once and to have an occasional override.

This reads (or creates if it doesn't find one) a `.chipalarmrc`
file with three standard alarm times and one per-day-of-week
alarm time. While running, every thing on the screen is a button.
Click an alarm to toggle that, click the day of week to clear all
alarms on that day.  Push one GUI button over and over enough 
(I think it is set at five times) and the program quits.

The program saves state automatically if there have been any 
changes in what alarms are set. The wfites are delayed until the
next minute, trying to group them. Changing the times is done
by editing the rc file and either restarting the program or
sending it a `HUP` signal with `kill`.

```
vi .chipalarmrc
kill -HUP %1
```

Make changes to the alarm times and reread the file before
changing which alarms are set to avoid the race condition

When an alarm fires, it starts another program (once) and stops
caring. The other program is responsible for sounding the alarm
and dealing with the alarm off button.

The GPIO Program
----------------

This program has the hardware complicated side of things. It
plays a sound file (repeatedly up to 30 times or 60 seconds as
configured) and listens to a GPIO pin. Every 1/100th second the
GPIO pin is sampled. With ten high samples and ten low samples
-- to allow for a switch normally open or normally closed -- it
decides that's enough and kills the sound playback and quits.

I had a lot of trouble finding documentation (at first) for how
to use GPIO. Everything I encountered just described how the
Python library for the Raspberry Pi GPIO worked, not how GPIO
actually works in Linux. Thus I documented my code in some
detail. The general way it works is find the device directory in 
`/sys/class/gpio` for your driver card, write to some files there
to enable GPIO, then read from some files there to get state.
Not so difficult once you know the tricks (and the names).

As I recall, I had to add my user to a group that has write
access to the GPIO files. I don't recall the specifics for
the Chip, and on other systems those specifics would be
different anyway.

History
-------

I wrote the first version of this in mid-2016. I have RCS
files with the dates and changes, but I did not convert that
change history to git, as no one else cares.
