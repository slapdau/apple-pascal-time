# Contributing

Thank you for thinking about contributing. Anything you can do to make the time
support for UCSD Pascal better on the humble Apple II, both real and emulated,
will make the world just that tiny bit more awesome.

There is value in reading the guidelines. Firstly, I can't fix defects if I
don't understand what went wrong. The easier you make it for me, the more likely
I am to do something. Secondly, if you want to contribute code yourself doing so
in a way that makes it easy fore me to understand and apply will increase the
odds that I will.

Things that I would greatly appreciate:

- Drivers for more clocks. I may add more myself, but that will be contingent on
  me getting clock hardware that I can develop and test against. The big
  name in Apple Clock hardware that is missing support is Timemaster H.O.
- Support for setting time.

## Guidelines

Content is encouraged, but a well thought out defect report is welcome. That
requires the following:

- A clear description of how current behavior deviates from desired or expected
  (note that is often a matter of opinion and someone else may disagree with
  you).
- Instructions to reproduce. It's hard to give guidance on how detailed the
  instructions have to be. Be prepared to negotiate.
- Test data or assets as required.

Before contributing code, please review the LICENSE and NOTICE files. To keep
things simple, contributions must be made with an Apache License, Version 2.0.
Contributed code must be correctly attributed. This may be by combining the
code contributed with an update to the NOTICE file.

If you are contributing code, then do so using the mechanisms provided by Git.
It makes it easier to see, review and apply the changes. It's also a good way
for me to keep attribution correct.

I know all about the bizarre structure of UCSD text files. However files in git
source control are in standard format for the host system. A disk image with the
source in Apple Pascal TEXT files is provided as part of each release, but that
is only as a convenience.

That said, I've haven't actually worked out a good way to get text files from
the host OS into Apple Pascal so I'm mostly updating the project by exporting
files from disk that I have used Apple Pascal to work on.

No reformatting of code to suit your preferred line breaks, indentation, imports
organization, etc. This is mostly because changes like this mess with source
control blame annotations.

While your own contributions are not going to be exhaustively checked against a
312 point code style guide, do roughly follow the style of the files you are
modifying so that your own contribution looks like it belongs there.

That's about it in terms of formal guidelines. This project is currently very
much a one man show in the quiet hours after dinner. There is no social media.
There is no project website. Code review consists of a quick once over to check
there aren't too many "Eugh!" moments.
