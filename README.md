# Apple Pascal Time and OS Global Variables

This is a wildly anachronistic project to give Apple Pascal running on an Apple
II series computer the ability to read a hardware Real Time Clock (RTC) and
update the current date as recorded by the Pascal Operating System.

## Background

If you're reading this, chances are you already know the background. But let me
relate history just in case someone stumbles on this unwittingly.

Apple Pascal, based on UCSD Pascal, for the Apple II series of machines, was
published in 1979. While UCSD had support for time, there was no RTC support.
The only time support was access to a free running system clock to measure
execution time. Apple Pascal didn't even have that since Apple II hardware had
no clock hardware other then the 14 MHz system oscillator.

Add-on clock hardware was later provided by third party vendors. Mostly this was
in the form of add-on cards, such as the popular Thunderclock Plus that was
introduced in 1980. One remarkable solution was the No-Slot Clock installed in
parallel with one of the standard system ROM chips. When the IIgs was
introduced, it included its own built in clock hardware that shared hardware
registers with the battery-back settings RAM that were accessed by memory-mapped
I/O.

Which is all fine and good, but Apple Pascal, from the original 1.0 in 1979 to
1.3 released in 1984, never incorporated any standard support for reading
clocks. Apple Pascal *would* update file directory entries with a last modified
date stamp. But only if the user remembered to update the system date every day,
which I was very bad at.

The ProDOS line of operating systems did have clock support. The original ProDOS
support was basic. It also set the creation and/or modification
time and date when a file was saved. But if there was a supported system clock
available, the values would be read from that, automatically as necessary.

Nostalgia means I've been using Apple Pascal on Applewin and GSport. I still
have a bad memory and I wanted clock support. Applewin emulates a No-Slot Clock.
GSport emulates the IIgs clock.

## Design Choices

I originally did something fancy. First I thought I might like applications that
were portable without recompilation across the two emulators. So I came up with
a solution that was based on having a user driver, like the Apple II Mouse
driver for Apple Pascal (available as part of the MouseText Tool Kit). It proved
to be unwieldy, so I changed it to a simple external assembly procedure that
could be linked into a host program. There might be some use for a user driver
version, so I've kept a copy of the code.

The other thing that changed as I went was access to the Pascal system global
variables. I was fairly certain that system programs could access them, but the
Apple documentation for the `$U-` compiler switch is very spartan: 

> Compilation at the system level will produce meaningful results only if the
> program was written with knowledge of the operating-system structure. Do not
> attempt system-level compilation unless you have this knowledge.

However the P-machine internals were documented, so I came up with an external
assembly procedure that read the system registers, and walked the procedure call
dynamic links back to the head of the call stack. It was very interesting, and
took a long time to get right. Then I discovered a book that explained how to
use the `$U-` compiler option (*Advanced UCSD Pascal Programming Techniques* by
Willner and Demchack, Prentice-Hall, 1985). Doing things this way only took an
hour or so to implement.

## Source Catalogue

 * **`STDMACRO.TEXT`** -- A collection of useful assembly language macros from
   appendix 3D of the Apple Pascal 1.3 manual.
 * **`SYSSTUFF.TEXT`** -- A unit for reading the Pascal system global variables.
   Relies on an assembly routine in `SYSTU.ASM.TEXT`.
 * **`SYSSTU.ASM.TEXT`** -- An assembly routine for locating the global
   variables of the Pascal runtime system. Operates by walking the dynamic stack
   frames up to the top looking for the procedure at lexical level -1.
   Positively fascinating. But using the `$U-` compiler option is a much better
   solution. See `STARTUP.TEXT`.
 * **`USERDRIVER.TEXT`** -- The sample device driver, more or less, from appendix A
   of the Apple Pascal 1.3 Device Support Tools documentation.
 * **`CLKUNIT.TEXT`** -- A unit driver that implements reading the IIgs clock as a
   `UNITREAD` call.
 * **`CLKUNIT.DATA`** -- `SYSTEM.ATTACH` data for `CLKUNIT.TEXT`.
 * **`CLOCKSTUFF.TEXT`** -- A Pascal unit that provides the necessary type
   definitions for calling the clock unit, and provides a skeleton routine that
   hides the `UNITREAD` call. 
 * **`READTIME.TEXT`** -- A sample application that uses `CLOCKSTUFF.TEXT`.
 * **`OLDSTART.TEXT`** -- The original version of a system startup application
   that would read the system RTC, and update the Pascal system date
   automatically. Uses `SYSSTUFF.TEXT` and `CLOCKSTUFF.TEXT`.
 * **`IIGSCLK.TEXT`** -- An Apple IIgs built-in clock reading routine intended
   to be linked directly into a program. Works on a real IIgs ROM 03 and in
   GSport with ROM 03.
 * **`NOSLOTCLK.TEXT`** -- A No Slot Clock reading routine. Does a search for
   the clock installation location the same way that version 1.4 of the ProDOS 8
   driver does. Works on a real enhanced Apple //e with a Manilla Gear No-Slot
   clock inserted under the CF ROM. It also works just fine inside an Applewin
   emulator.
 * **`CLOCK.TEXT`** -- A unit that defines the Pascal interface to call either
   driver.
 * **`CALLRDTIME.TEXT`** -- A simple host program that links to either
   `IIGSCLK.TEXT` or `NOSLOTCLK.TEXT`, calls the ReadClock procedure and
   displays the current time and date.
 * **`GLOBALS.TEXT`** -- `const`, `type` and `var` declarations for the Pascal
   system global variables. Originally based on the released code of UCSD Pascal
   I.5. Updated to match changes for Apple Pascal 1.3. For example, the core
   unit table is extended from 12 to 20 units.
 * **`STARTUP.TEXT`** -- A startup program that will link to either
   `IIGSCLK.TEXT` or `NOSLOTCLK.TEXT` and set the system date to the current
   date. Compiled as a system program to access the system global variables.
   This technique is explained in *Advanced UCSD Pascal Programming Techniques*
   by Willner and Demchack (Prentice-Hall, 1985). It's described in section
   5.0.12. The date is also written back to the system boot disk.

## Building

Building applications is a manual sequence of steps. Exec files would be able to
automate some of this, but they would be opaque (figuring out file and build
dependencies from an Exec file is hard) and unreliable (they are just blind
sequences of canned user input that make no allowances for failure). Here is a
rough guide.

Note, no file references (uses or includes) have a volume qualifier. All
required files will need to be on the default volume defined by the Filer Prefix
command.

The following instructions skip interim steps that are helpful such as getting
and saving the system work files.

### System Stuff

Depends on `STDMACRO.TEXT`.

  1. Assemble `SYSSTU.ASM.TEXT`.
  2. Compile `SYSSTUFF.TEXT`.
  3. Link in `SYSSTU.ASM.CODE`.

### Clock User Driver

Does not depend on anything else.

  1. Assemble `CLKUNIT.TEXT`.
  2. Add to a library called `ATTACH.DRIVERS`.
  3. Copy `ATTACH.DRIVERS`, `SYSTEM.ATTACH` to the system boot disk.
  4. Either copy `CLKUNIT.DATA` to the system disk as `ATTACH.DATA`, or use
     `ADMERGE.CODE` to add it to an existing `ATTACH.DATA`.

### Clock Unit

Does not directly depend on anything else. It's usage relies on the clock user
driver being installed, and they must agree on the clock data structure.

  1. Compile `CLOCKSTUFF.TEXT`.

### Reading Time with the User Driver

Depends on Clock Unit. Depends on the Clock user driver being installed.

  1. Compile `READTIME.TEXT`.
  2. Link in `CLOCKSTUFF.CODE`.

### The Original Startup

Depends on System Stuff and Clock Unit. Depends on the Clock user driver being
installed.

  1. Compile `OLDSTART.TEXT`.
  2. Link in `SYSSTUFF.CODE`, `SYSSTU.ASM.CODE`, and `CLOCKSTUFF.CODE`.
  3. Copy to the system disk as `SYSTEM.STARTUP`.

### IIgs Built-In Clock External Procedure

Does not depend on anything else.

  1. Assemble `IIGSCLK.TEXT`.

### No-Slot Clock External Procedure

Does not depend on anything else.

  1. Assemble `NOSLOTCLK.TEXT`.

### Clock Unit

Does not depend on anything else.

  1. Compile `CLOCK.TEXT`.

### Reading Time with the New External Procedures

Depends on the clock unit, and on either the IIgs external procedure or the
No-Slot Clock external procedure.

  1. Compile `CALLRDTIME.TEXT`.
  2. Link with `CLOCK.CODE`, and either `IIGSCLK.CODE` or `NOSLOTCLK.CODE`.

### The New Startup

Depends on either the IIgs external procedure or the No-Slot Clock external
procedure. Depends on `GLOBALS.TEXT`.

  1. Compile `STARTUP.TEXT`.
  2. Link with either `IIGSCLK.CODE` or `NOSLOTCLK.CODE`.
  3. Copy to the system disk as `SYSTEM.STARTUP`.

## Usage

This only describes the current model of clock driver, implemented in
`IIGSCLK.TEXT` and `NOSLOTCLK.TEXT`, and called by `CALLRDTIME.TEXT` and
`STARTUP.TEXT`. The interface into both drivers is summarised by the following
declarations.

```pascal
type
    daterec = packed record
        month: 0..12;
        day  : 0..31;
        year : 0..100
    end {daterec};
    timerec = packed record
        hour   : 0..23;
        minute : 0..59;
        second : 0..59;
    end {timerec};
    clockrec = record
        date: daterec;
        time: timerec
    end {clockrec};

function InitClock:boolean;
external;

procedure ReadClock(var now:clockrec);
external;

procedure ClockName(var name:string);
external;
```
`InitClock` must be called and return `true` before `ReadClock` and `ClockName`
will return meaningfull values. `InitClock` can be safely called multiple times.
`ReadClock` and `ClockName` can be safely called if `InitClock` has not been
called or has returned `false`, but will return a zeroed out clock record and a
name indicating that there is no clock.

The declarations above can be copied into your application inline, which is then
compiled and linked to either clock driver, or you can use the clock unit and
then link to both it and one of the clock drivers.

## Build Catalogue

 * **`START.IIGS.CODE`** -- `STARTUP.TEXT` compiled and linked with `IIGSCLK.CODE`.
   Ready to be transferred as `SYSTEM.STARTUP` to a IIgs startup disk.
 * **`START.NSC.CODE`** -- `STARTUP.TEXT` compiled and linked with `NOSLOTCLK.CODE`.
   Ready to be transferred as `SYSTEM.STARTUP` to a startup disk for a system
   with a NoSlotClock installed. Works on AppleWin.
 * **`CLOCK.CODE`** -- The Clock interface unit compiled and ready to be linked
   with your application and a driver.
 * **`NOSLOTCLK.CODE`** The No-Slot Clock driver assembled and ready to be
   linked with your application.
 * **`IIGSCLK.CODE`** The IIgs built-in clock driver assembled and ready to be
   linked with your application.
