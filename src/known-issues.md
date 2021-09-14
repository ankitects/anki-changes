# Known Issues

<!-- toc -->

## Hardware acceleration and startup issues on Windows/Linux

Hardware acceleration defaults to off on Windows and Linux. Enabling it
in the preferences screen and restarting Anki may make Anki’s interface
more responsive, but some users may experience missing menubars, blank
windows or crashes when it is enabled.

On Windows if you’re unable to get to Anki’s preferences screen and
restarting Anki a few times does not help, you may need to manually
adjust the graphics driver. You can do this by starting cmd.exe and
typing the following:

    echo auto > %APPDATA%\Anki2\gldriver

There are three settings you can try: 'auto', 'angle', and 'software'.
If you have problems both when hardware acceleration is turned on and
turned off in the preferences, it’s worth giving 'angle' a go.

On Linux, you can write either 'auto' or 'software' into
~/.local/share/Anki2/gldriver to adjust the driver. Please note that if
you’re using nouveau, it is known to be buggy and it only supports
software mode.

## Interface speed

Even if you enable hardware acceleration as mentioned above, you may find Anki
takes longer to start and show new windows than Anki 2.0.x - especially if your
computer is not the fastest. This is caused by the use of a newer web toolkit,
which brings new features and security improvements, but is unfortunately more
resource intensive. The web toolkit older Anki versions use has been abandoned
at this point, so it is no longer an option for future releases.

## Blank screens and eGPUs on Macs

If you experience blank screens when using an external graphics card on a Mac,
You can ctrl+click on the Anki app, click "Get Info", and enable the "prefer
eGPU" option.

When switching between monitors of different resolutions, you may
also run into problems that can be [worked around](https://forums.ankiweb.net/t/mac-known-issues-wording-suggestion/7331).

## Copy & paste problems on Windows

If you are experiencing problems with copying and pasting on Windows,
please check if you are running other programs on your computer that
monitor the clipboard, such as dictionary programs, clipboard managers
or clipping tools. The toolkit Anki uses can have trouble when such
programs are running.

## Wayland on Linux

From Anki 2.1.48, you can force Anki to use Wayland by defining ANKI_WAYLAND=1
before starting Anki. Wayland may give you better rendering across multiple
displays, but it is currently off by default, due to the following issues:

- On some distros, Windows are rendered without borders.
- Bringing windows to the front is not possible, so for example, clicking on Add
to reveal an existing Add Cards window will not work.

## Fcitx on Linux

The standard Anki build includes fcitx support, but it may not work on
all distributions. If you are unable to use fcitx, you may want to run
Anki from source, or switch to a different input method.

## Text size

If you find the text is the wrong size, there are two environmental
variables you can try:

- ANKI_NOHIGHDPI=1 will turn off some of Qt’s high dpi support

- ANKI_WEBSCALE=1 will alter the scale of Anki’s web views (like the
  deck list, study screen, etc), while leaving interface elements like
  the menu bar alone. Replace 1 with the desired scale, such as 1.5 or
  0.75.

On Windows you can add these to a batch file to make it easier to start
Anki. For example, create a file called startanki.bat on your desktop
with the following text:

    set ANKI_WEBSCALE=0.75
    start "Anki" "C:\Program Files\Anki\anki"

After saving, you can double click on the file to start Anki with that
setting.

## SSL errors

The following applies to Anki versions before 2.1.28. Newer versions
use the system keychain, and no longer need the workaround below.

Some work and school networks intercept your internet traffic, and this
can cause errors when syncing and downloading add-ons. You can prevent
the errors from occurring by setting the environmental variable
"ANKI_NOVERIFYSSL" to "1".

When you enable this option, you are telling Anki not to verify that it
is actually talking with AnkiWeb. This means that not only your work or
school, but also a bad actor on the local network may be able to
intercept your syncing traffic.

On Windows, you can create a .bat file as described in the Text size
section above.

On Mac, you can open Terminal.app, then start Anki with the following
command:

    ANKI_NOVERIFYSSL=1 open /Applications/Anki.app

On Linux, use the following in a terminal:

    ANKI_NOVERIFYSSL=1 anki

Once you’ve started Anki with SSL disabled, you can download the
following add-on to make the change permanent:

<https://ankiweb.net/shared/info/878367706>
