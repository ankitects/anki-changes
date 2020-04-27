# Changes

## Changes in 2.1.24

Released 2020-04-28, build 359b9f5c.

:warning: After using this version, if you wish to open your collection with an
earlier Anki release, please go to the File>Switch Profile menu item, and click
on "Downgrade & Quit". If you skip this step, you may get an error message when
opening your collection in an older Anki version, and you will need to return to
this version, downgrade, then try again.

Searching:

- You can use `w:something` to search on word boundaries, eg:
  - `w:dog`  
    search for "dog" on a word boundary - will match "dog", but not "doggy"
    or "underdog".
  - `w:dog*`  
    will match "dog" and "doggy", but not "underdog".
  - `w:*dog`  
    will match "dog" and "underdog", but not "doggy".
- You can now use `re:something` to search via regular expression, eg:
  - `"re:(some|another).*thing"`  
    find notes that have "some" or "another" on them, followed by 0 or more characters, and then "thing"
  - `re:\d{3}`  
    find notes that have 3 digits in a row
  - When searching by regex, unicode case folding is used, so searching for `re:über` will
    show a card that has "Über" on it.
- `nc:something` (short for "no combining (characters)") can be used to search while
  ignoring accents, eg `nc:uber` will match both "über" and "Über". This behaves the same way as the "Ignore Accents" add-on, but is about 16x faster.
- You can now sort on the deck, card template, note type and tags columns.
- You can now use wildcards when limiting the search to a field, eg `field*:something`.
- You can now use wildcards when searching for a card template or note type by name.
- `rated:x` searches are now capped to a year instead of a month.
- You can now escaped double-quotes in a search - eg `"foo\"bar"`
- Single-quote searches are no longer supported.
- Because the searching code has been rewritten, add-ons that modify the search
  code will need to be updated to support 2.1.24. It is no longer possible to
  override the Finder class - add-ons will need to use the new hooks in the
  browser screen to either rewrite the search text, or perform their own lookups
  instead. The Advanced Browser add-on has already been updated, and can be used
  as an example of how to accomplish things in 2.1.24.
- Non-wildcard searches now do full unicode case folding (eg 'tag:masse' matches 'Maße').
- Wildcard searches do simple unicode case folding.
- The tag list in the Browse screen now uses unicode case folding.

macOS dark mode handling:

- Anki now solely relies on the night mode setting in the preferences to decide
  whether to show in light or dark mode. Some users wanted to run Anki in light
  mode while keeping the rest of their system dark, and there were various
  display problems when dark mode was changed after Anki started that couldn't
  be easily worked around.
- Users who only use dark mode, and preferred the native look of widgets in dark
  mode, can achieve the previous appearance by running the following command in
  the terminal:

  ```
  defaults write net.ankiweb.dtop NSRequiresAquaSystemAppearance -bool no
  ```

  And the following in the debug console:

  ```
  mw.pm.meta["dark_mode_widgets"] = True
  ```

Database changes (mainly of interest to add-on developers):

- Anki now uses Rust's sqlite libraries instead of Python's.
- The 'db' object on the collection retains most of the same API as before, minimizing the amount of immediate code changes that are required.
- Custom sql functions are no longer supported, and named DB arguments (eg "where id = :id") are deprecated.
- The old database code remains in db.py, and add-ons can continue to use it for accessing
  their own databases.
- The database is now behind a mutex, and can be safely accessed from a background thread.
- Various screens like the database check have been updated to run on a background thread,
  so they no longer lock up the UI while they're running.
- The database progress handler has been removed. Anki previous had sqlite call back into
  Python periodically during long-running DB operations so it could drain the UI queue,
  but this would vary in choppyness depending on the type of DB operation being performed,
  and it was the cause of some crashes in the past. Add-ons that perform long-running operations
  should instead use mw.taskman.run_in_background() or their own threading solution moving forward.

Other changes:

- A tweak which should fix some broken add-ons from preventing the collection from being loaded.
- Add socks support to media sync.
- Allow dragging fields to change their position (thanks to BlueGreenMagick).
- Allow selecting add-on config help text (thanks to ijgnd).
- Allow the type answer arrow to be styled (thanks to Evandro).
- Anki will now wait for a media sync to complete or be aborted before closing the collection.
- Build improvements (thanks to Evandro).
- Changed the way cloze deletions in RTL fields are handled, which should address some corner cases.
- Clean up the previewing code (thanks to Arthur). Add-ons that modify the preview screen will need updating.
- Don't force a full sync when DB check finds cards with a high due number.
- Don't show a popup when a network error occurs while syncing media.
- Fixed a case where decks could be sorted incorrectly (thanks to Arthur).
- Fixed a useless log file being created when exporting.
- Fixed add-ons with multiple branches not updating properly.
- Fixed an error that could occur when performing an operation in the browse screen then immediately closing it.
- Fixed Anki closing after a full sync on collection load.
- Fixed current card sometimes not being centered when searching.
- Fixed deck_browser_did_render hook.
- Fixed editor buttons not being highlighted (thanks to Simone).
- Fixed interface getting stuck when a corrupt collection was encountered.
- Fixed media sync waiting forever when connection dropped.
- Fixed progress dialogs failing to appear in a timely manner.
- Fixed tag searches in custom study (thanks to zjosua).
- Fixed the collection_did_load add-on hook.
- Fixed the wrong language shown in the preferences screen for some languages.
- GitHub now checks Windows and Mac builds as well (thanks to Evandro).
- Handle renamed cloze fields when previewing (thanks to BlueGreenMagick).
- Ignore .DS_Store files in the media trash folder.
- Improved invalid HTML/JS error messages (thanks to Evandro).
- Improved the handling of deck deletions (thanks to Arthur).
- Improvements to debug console (thanks to BlueGreenMagick).
- Left-align tags in the browser.
- Media syncs no longer take time to abort.
- More hooks (thanks to Arthur).
- Moved the scheduling options in the preferences to a separate tab, so options fit on the screen even on devices with small screens.
- Prepare for uploading releases to PyPI (thanks to Evandro).
- The media check will now fix file references in fields that broke because a filename was shortened as part of a sync.
- Updated config handling. While there should be no immediate breakages, if you're an add-on author and
  store lists or dicts in Anki's config, please see 676f4e74a.

## Changes in 2.1.23

Released 2020-03-19, build de9543ff.

A macOS-only build that fixes a problem syncing media files
with non-Latin filenames added by previous Anki versions on macOS.

Please see 2.1.22 below for the bulk of the changes.

## Changes in 2.1.22

Released 2020-03-18, build 0ecc189a.

Media syncing improvements:

- Media syncing now happens in the background, so you can continue using
  Anki while the media sync completes.
- Aside from syncing at open and close, Anki will sync any media changes
  every 15-20 minutes.
- You can click on the sync button while the spinner is active to monitor
  progress.
- Long filenames and problematic characters should be handled smoothly now,
  instead of causing syncing errors.
- Anki should no longer sometimes forget to download files when a media
  sync fails due to network errors.
- When media files are added within Anki, Anki now marks them
  in the database immediately, which can make things faster for people with
  slower disks if they are not modifying the media folder externally.

Media check improvements:

- The Check Media function now shows progress, and can be interrupted.
- There is now a separate button to generate missing LaTeX.
- If LaTeX fails to build, the problem card will be revealed in the browse
  screen.
- When Anki finds files that are too long or would cause errors on some
  operating systems, it will automatically rename them and update your notes
  to point to the new filename.

Both media sync and the media check now place deleted files in a media.trash
folder inside your profile, instead of placing the files in the system trash.
You can use the Check Media function to either empty the trash, or restore
the deleted files back to your media folder.

Other changes:

- You can now export .apkg files with the V2 scheduler enabled.
- Add "New #" prefix to the due column for new cards.
- Fixed audio getting stuck on Windows.
- Clear the audio queue when moving between cards with autoplay off.
- Fixed play icons not appearing in browser preview when autoplay off.
- Show next learning card due time, and count for today.
- Restored grey styling of zeros in the deck list that got lost in the night mode changes.
- Improvements to the readability of the scheduling code (thanks to Arthur)
- Add-on hook improvements, thanks to Glutanimate and Arthur.
- Fixed fields containing a filename with non-Latin text from being corrupted when editing HTML (thanks to Evandro).
- Support for validating add-on config schemas (thanks to Arthur).
- Removed the 'too many decks' message in the deck list screen.
- Fix for issue playing audio from flash drive.
- Fixed Anki getting stuck when importing an invalid file.
- More type hints in the code (thanks to Alan).
- Improvements to the build process and building on Windows (thanks to Evandro).
- Support '/' separator in add-on web paths on Windows (thanks to BlueGreenMagick)
- Fix tags that are in the wrong encoding as part of the DB check.
- Hide the default deck in more cases (thanks to Arthur).
- Updates to the translation infrastructure, including tweaks to the way
  the answer buttons and the review history screen show intervals.

## Changes in 2.1.21

Released 2020-03-09, build f1734a47.

- Fixed error messages when playing audio.
- Fixed legacy add-on filters not working (reading generation in Japanese Support, etc).
- The alternate Mac build works properly when macOS is in dark mode now,
  and can be used if you prefer light Anki in macOS dark mode.
- Prevent UI scale from being decreased below 100%, which caused display problems.
- Fixed Anki failing to start on some Windows 7 machines that were missing TTS support.
- Display a more useful message when mpv/mplayer not installed.
- Don't allow exporting into Anki folder.
- Fixed display of AnkiMobile drawings in night mode.
- Fixed interrupting of current audio when autoplay is turned off.
- Night mode defaults to dark grey instead of black card background.
- Fixed {{Deck}} showing filtered deck instead of original deck.
- Fixed an error that could occur with very small learning steps.
- Fixed a negative version number being shown when add-ons incompatible.
- Fixed some invalid HTML in the review screen (thanks to BlueGreenMagic)
- Added back missing fcntl module.

## Changes in 2.1.20

Released 2020-02-14, build 47a1bf8b.

Template changes:

The way Anki combines your card templates and fields has been updated.
Many users will not notice a difference, but if you encounter error
messages inside the review screen and opening and closing the Cards
screen from the editing area does not resolve the issue, please see
[this support
page](https://anki.tenderapp.com/kb/problems/card-template-has-a-problem).

Add-ons that alter the way cards are shown may [need to be
updated](https://anki.tenderapp.com/discussions/beta-testing/1706-anki-2120-updates-to-card-template-rendering).

Audio changes:

- Text to speech is now [supported in card
  templates](https://apps.ankiweb.net/docs/manual.html#text-to-speech).

- Audio buttons are now shown on the card, and can be turned off in
  the preferences. They will show for both regular audio and text to
  speech.

- You can [customize the size and
  colour](https://apps.ankiweb.net/docs/manual.html#audio-replay-buttons).

- Added shortcut keys in the review screen to pause and jump
  forward/backward 5 seconds.

- Anki now starts a new copy of mplayer for each audio file on
  Windows, which avoids the need to create temporary files.

- Added an option in the preferences to not interrupt the currently
  playing audio when answering.

- Fix multiple spaces in filenames from getting truncated when pasting
  sound files.

Night mode:

- The night mode option in the preferences screen now turns the
  interface dark as well.

- On macOS, when the system is in dark mode, Anki will switch to night
  mode automatically.

- Invert LaTeX in night mode (thanks to zjosua).

- Some of the colours in areas like the graphs could be improved -
  pull requests with included screenshots of the changes would be
  appreciated.

Add-on changes:

- Anki will now check for add-on updates automatically once a day.

- Disabled add-ons are now included in the check as well.

- Add-on authors can specify the minimum and maximum Anki version they
  support, and add-ons will be automatically disabled when running on
  an unsupported Anki version.

- Add-on authors can now upload different add-on versions for
  different Anki versions, and Anki will download the correct one.

- A new hook system for add-ons - please see
  [here](https://anki.tenderapp.com/discussions/beta-testing/1704-anki-2120-updates-to-the-hook-system).

- For add-on authors, some more examples using the new hook system are
  available on the following page, including ported versions of the
  clickable tags and additional card fields add-ons:
  <https://github.com/ankitects/anki-addons/tree/master/demos>

Other changes:

- Added the ability to export selected notes from the Browse screen
  (thanks to Arthur).

- Updated to a newer toolkit.

- Emptying a filtered deck in the V2 scheduler no longer unsuspends
  suspended cards inside it.

- Fix incorrect delay being logged when Hard is used on the first
  learning step in the V2 scheduler.

- The editor no longer modifies percent-escaped text outside of image
  tags.

- Fix an extra linebreak being left in a field when an image is
  attached to an empty field.

- Tweaks to the 'tag updated notes' feature (thanks to Erez)

- Fix cards being sorted in wrong order when added after the note was
  created (thanks to Arthur)

- Disabled elastic scrolling in webviews to work around a Qt bug.

- Don’t filter em/strong tags when pasting.

- Fix error when double-clicking the open profile button.

- Constrain image width in editor to the field width.

# Changes in 2.1.19

Released 2020-01-16, build 3c8690ae.

- Fix formatting and images getting lost when creating cloze
  deletions.

- Added an option to the preferences screen to strip formatting by
  default.

- Fix the preview button shortcut key not working.

# Changes in 2.1.18

Released 2020-01-14, build fb9d59fe.

- Fixed Anki failing to start for some users updating from Anki 2.0.

- Fixed the alternate Windows build failing to start on Windows 8.

# Changes in 2.1.17

Released 2020-01-11, build c69ccb50.

Linux version repackaged on 2020-01-12 to fix a 'make install' issue.

- Improved the performance of the browse screen’s sidebar for users
  with many decks/tags.

- Add-ons that modify the sidebar will break when you update, and will
  need to be updated by the add-on author.

- Add-ons that alter the audio handling may need to be updated, as
  code has moved from anki.sound to aqt.sound.

- Changing large note types is significantly faster.

- Added an option in the preferences screen to adjust the user
  interface size.

- You can now double-click on an .ankiaddon file to install it (thanks
  to Glutanimate).

- Updated GUI libraries for the standard installs and the alternate
  Windows install.

- The minimum Python version is now 3.7, and the packaged versions
  ship with Python 3.8.

- The alternate Linux build has been dropped - you will need to be on
  a Linux distro from 2016+ with systemd support to use the packaged
  version.

- Source tarballs are available from the releases tab of the GitHub
  repo.

- Added an option to tag updated notes when importing (thanks to
  Erez).

- Automatically remove ':' from field names when opening the card
  templates screen, as it conflicts with the template syntax.

- Fix a bug in the handling of MathJax+Cloze (thanks to Michal).

- Fixed a regression in the way duplicate deck names were handled.

- Remove help button from some Window titles.

# Changes in 2.1.16

Released 2019-12-12, build 4bc33e2f.

Due to some minor issues that were found, the website was not updated to
point to this release.

- Pasting now includes formatting by default.

- Preserve foreground/background color when pasting.

- Preserve bold/italic/underline when pasting from Google Docs.

- When pasting with the shift key, bold/italics/underline is also
  stripped.

- Ensure learning cards in filtered decks with 'order due' show in
  template order.

- Remove the 'experimental' label from the new scheduler.

- You can now import and export decks with scheduling enabled in the
  new scheduler.

- Hide empty Default deck in deck picker (thanks to Arthur).

- Add an extra day to the interval when using Easy on a relearning
  card.

- Preserve surrounding styling when making cloze deletions.

- Draw preview screen more quickly.

- Fix race condition in preview screen (thanks to Håkon).

- Use --exact with dvsvgm to prevent truncated subscript/superscript
  in LaTeX.

- Newly created cards could be given the wrong due number (thanks to
  Arthur).

- Sticky fields were ignored when closing the add card window (thanks
  to Arthur).

- Adding a note type forced a full sync (thanks to Arthur).

- Remove shortcut keys from translations (thanks to Arthur).

- Documentation changes for translators (thanks to Arthur).

- Case not being preserved when changing a deck’s parent (thanks to
  Arthur).

- Hide default deck in other screens when empty (thanks to Arthur).

- Fix qtwebengineprocesses not being cleaned up when stats window
  closed.

- Allow smaller window when editing current card.

- Support pasting multiple URLs at once.

- Add ability to force software rendering on old Macs (thanks to Mike)

- A fix for case insensitive field name handling in find&replace
  (thanks to lovac42)

- Fix non-integer intervals being imported from Mnemosyne (thanks to
  Blauelf)

- Clear undo queue when changing scheduler (thanks to lovac42)

- Default to not closing add window (thanks to Aidan)

- Sort new cards separately when sorting by ease (thanks to Arthur)

- Fix a bug in the V2 scheduler.

- Properly handle backslashes in the replacement section of
  Find&Replace.

- Various small fixes.

# Changes in 2.1.15

Released 2019-08-22, build 442df9d6.

- The V2 scheduler now fully randomizes review cards due on a given
  day.

- Fix add-ons errors on Windows when profile path was short.

- Fix flag changes in Browse screen not syncing.

- Cleanup recording wav file when recording canceled.

- Fix the window icon on Wayland (thanks to Wilco).

- Add a progress bar to media deletion.

- Other minor changes.

# Changes in 2.1.14

Released 2019-06-27, build 7b93e985.

- Fix a bug in the V2 scheduler that would cause partially learnt
  cards removed from filtered decks to revert to an earlier state.

- Fix a bug in the handling of relearning cards when switching back to
  the V1 scheduler.

- Fix lost space when pasting indented text.

- Limit image height relative to window height, not document height.

- Fix deck list being re-rendered unnecessarily.

- Remove the 'unable to connect to local port' message.

# Changes in 2.1.13

Released 2019-05-20, build 3ba55990.

- Fix formatting getting lost when copying&pasting between fields on
  macOS.

- Fix some issues that cause the main window to get stuck.

- Fix an empty deck list sometimes appearing when restoring from a
  backup.

- Fix Anki hanging after an error occurs during startup.

- Fix error caused by profile with trailing space on Windows.

- Fix error message when syncing with an unconfirmed email address.

- Use jsonschema for add-on manifests (thanks to Erez).

- Warn in DB check when high due numbers are encountered.

- Improve error messages on full disk and failed add-on deletion.

- Fix relearning cards being given learning step count in V2
  scheduler.

- Fix preview window failing to appear when 'show both sides' enabled.

- Removing trailing BR tag when pasting into an empty field.

- Don’t throw an error when non-Latin text in the Javascript console
  can’t be shown.

- Double click on add-ons to edit their configuration (thanks to
  lovac42).

- Fix the window icon in a few screens (thanks to John).

- Don’t highlight the deck selection button in the add screen on
  Windows.

- Improve the default 'type in the answer' note type.

# Changes in 2.1.12

Released 2019-04-23, build eef86bf3.

- Fix an issue that could prevent profile renaming/deletion on
  Windows.

- Fix fields appearing under editor buttons.

- Fix memory leak in card layout screen.

- Fix some issues with previewing in the Browse screen.

- Fix card counts not updating when a review is undone.

- Fix an error that could occur on startup on some Windows installs.

- The Mac build now uses the new hardened runtime on Mojave.

- Change focus outline colour on Windows.

- Fix an error caused by missing note types.

- A possible workaround for the audio player getting stuck on Macs.

- Display the installed version in the Windows uninstall screen.

- Fix an issue checking for add-on updates (thanks to Glutanimate).

- Disable add-on config button when not appropriate (thanks to
  Glutanimate).

- Tweaks to the 'deck age' graph binning (thanks to Jian).

- Add-ons hosted on AnkiWeb can now define conflicts in the manifest
  file.

- Switch to mplayer on the alternate OS X build, as mpv was not
  working on some older machines.

- Make sure mpv doesn’t attempt to load scripts from default location.

- Other minor fixes.

# Changes in 2.1.11

Released 2019-03-11, build 3cf770c7.

- Change Undo shortcut back to Ctrl+Alt+Z/Cmd+Opt+Z in Browse screen,
  to prevent accidentally undoing non-text changes when editing
  fields.

- Revert a previous card template optimization that could cause an
  error.

- Suppress a spurious error message that could occur when editing.

# Changes in 2.1.10

Released 2019-03-07, build 22d6feed.

- Add option to strip html in export.

- Avoid nbsp for single spaces when pasting text.

- Fix preview screen flashing when moving between cards.

- Improvements to the add-ons screen (thanks to Glutanimate).

- Support .ankiaddon bundles (thanks to Glutanimate).

- Improve subpixel antialiasing on some machines (thanks to
  Glutanimate).

- Improve Japanese interface font on Windows 10, and make it possible
  for translators to change the font for other languages that need it
  as well.

- Fix inability to start if problem occurs on first run.

- Allow decreasing daily limits in custom study.

- Add a button to copy debug info to about screen (thanks to
  Glutanimate).

- Fix problem running from source on Windows (thanks to dlon).

- Allow add-ons to serve files from mediasrv (thanks to Glutanimate).

- More user-friendly error messages for some network errors.

# Changes in 2.1.9

Released 2019-02-20, build ae67c976.

- Update standard build to latest toolkit version.

- Hardware acceleration defaults to off again on Windows/Linux, due to
  the issues it was causing some users. If you were not experiencing
  any issues, turning hardware acceleration back on in the preferences
  screen is recommended.

- Various statistics fixes for the V2 scheduler, including an
  automatic remapping of button 2/3 in the review history when moving
  back and forth between scheduler versions so the "answer buttons"
  graph displays correctly.

- Fix BR tags being included in empty fields (thanks to David and
  zjosua)

- Optimize card template repositioning (thanks to Arthur)

- Fix a crash when copying/cutting with an empty selection (thanks to
  David)

- Avoid screen flash when undoing reviews.

- Make sure info/warning dialogs appear on top.

- Fixed an issue with just-typed text not being saved when using the
  mouse to save/add a card.

- Added support for \\{{CardFlag}}, which is either empty, or in the
  format "flagN" where N is 1-4.

- Fix bulk flag changes in Browse screen not syncing.

- Fix advanced menu in editor not showing shortcut keys.

- When UI fails to load after resuming computer from sync, show a
  tooltip and automatically refresh.

- Clean up old mplayer instances after a crash so that profile
  renaming works.

- Fix add-on list not refreshing when toggling enabled in latest
  toolkit.

- Fix cursor jumping on first click in "Edit Current" area on Windows.

- Preserve whitespace when pasting plain text.

- Prevent errors caused by a timer firing after collection is
  unloaded.

- Ensure a full sync is forced when restoring from a backup.

- Ensure full window is on screen when displaying windows on a changed
  screen layout.

- Improvements to the add-ons, debug console, and error screens
  (thanks to Glutanimate)

- Ensure \\{{Deck}} shows the correct deck when adding (thanks to
  Arthur)

- Ensure windows don’t get shown off-screen.

- Remember add-on window size and position.

# Changes in 2.1.8

Released 2019-01-02, build 71e0c880.

- Fix startup on Windows 8.

- Fix field content appearing under editor buttons.

- Better handle an error when recording.

- Fix improper handling of collections with deck errors.

- Fix duplicate deck names being created due to text encoding.

- Fix gtk2 theme and fcitx module not being included.

- Detect nouveau graphics drivers and automatically switch to software
  rendering.

# Changes in 2.1.7

Released 2018-12-17.

- Fix "QPushButton has been deleted" error messages after a problem
  occurs changing note types.

- Fix errors during "Check Database" that are just a byproduct of a
  previous operation that failed.

- Fix problems searching for some non-Latin text in decks/note type
  names.

- Ensure cgi and uuid modules are available to add-ons.

- Improvements to the Windows installer.

- Automatically restart mpv if it stops responding.

- Don’t convert non-Latin characters in add-on configuration to
  difficult-to-read escape codes.

- Add a bottom border to the menubar on Windows 10.

# Changes in 2.1.6

Released 2018-12-10.

Downloads are now split into a standard and alternate version.

**The standard version**:

- Is built with the latest toolkit, which fixes various issues.

- Changes the undo shortcut back to Ctrl+Z or Command+Z like Anki 2.0.

- Includes a separate anki-console.exe executable in the Windows build
  that may be useful for add-on authors.

- Includes support for the fcitx input method and a gtk2 theme in the
  Linux build.

You can switch between the standard and alternate 2.1.6 without
problems. If you move back to a previous Anki 2.1 release, your sync
login details and window positions will be lost, and will need to be set
again.

**The standard version has updated requirements**:

- The Windows build only supports 64 bit Windows 7 or later, and will
  not run on a 32 bit install.

- The Mac build requires macOS 10.12 or later.

- The Linux build requires a Linux distribution from approximately
  2016 or later.

The alternate version is similar to previous Anki 2.1 releases, and is
built with an older toolkit. It will run on some older systems that the
standard version will not, but as it is not as up to date, it may be
missing bugfixes and security improvements that the standard version
includes.

**The alternate version will run on**:

- Windows 7 32 bit or 64 bit installs

- Mac 10.10 or later.

- A Linux distro from around 2014 or later.

**In addition to the toolkit upgrade, there have been some changes in
Anki itself**:

- Improvements to the Browse screen and flagging:

  - Search text is normalized, which fixes problems searching for
    unicode characters with multiple possible encodings.

  - The selection is now partially transparent, allowing you to see
    the underyling colours of the rows.

  - The screen doesn’t scroll when performing actions that don’t
    change the selection count.

  - Flags now toggle on and off, and the separate clear flag action
    has been removed.

  - The second flag is now orange instead of purple.

  - Find&replace now only shows fields relevant to the notes you’ve
    selected, and is case insensitive.

  - Fix card list not updating when editing HTML.

- Importing apkgs is now more verbose about how notes have been
  handled.

- Prevent errors caused by the user adding a field reference to itself
  on a field.

- Better handle issues with the deck list, such as decks that are
  missing a parent deck.

- Anki should now be able to function even if a system proxy is
  configured for localhost connections.

- Fix font size being copied when pasting between Anki fields with
  bold text.

- Pasting a link with shift held down now creates a clickable link.

- Fixed an issue with the bulk remove tags option where tags with
  similar names could be removed as well.

- Fixed an error that occurred with very long filenames on Windows.

- Fixed an issue running latex commands on some Linux installs.

- The Browse screen’s sidebar now defaults to on.

- Fixed a race condition that could cause two copies of Anki to open.

- When adding media to cards, Anki now will automatically rename the
  filenames if they’re too long.

- The experimental scheduler now regularly checks if new learning
  cards have become due.

- Handle invalid add-on config (thanks to Arthur).

- Enforce template ordering when card templates are reordered after
  card creation (thanks to Arthur).

- Don’t change deck when Esc pressed in deck chooser (thanks to
  David).

- Fix a problem on initial startup when English not the default
  language.

- Fix busy cursor showing in import results screen.

- Fix content overlap when add-ons have added many editor buttons.

- Don’t change current note when reopening editor from review screen
  (thanks to Arthur).

- A fix for running on Python 3.7 (thanks to Alexey).

- Restore the tooltip for the Fields and Cards buttons in editor.

- A possible fix for 'database is locked' errors on Windows.

# Changes in 2.1.5

Released 2018-10-01.

- Use selected answer button instead of default when enter/space
  pressed.

- Change undo shortcut in browse screen to avoid conflict with editing
  functionality.

- Ignore standard mpv config file location in favour of Anki data
  folder.

- Fix importing of .apkg files when interface in Dutch.

- Fix translations not working on Linux after 'make install'.

- Support newlines in type:cloze, and treat them as spaces.

- Add browser.rowChanged hook for add-on authors.

- Possible fix for some 'database is locked' errors.

- Fix errors on startup when deck given an invalid name.

- Fix sorting not working when field contains only a media reference.

- Fix 'access denied' error not being caught properly.

- Fix exporting of v2 colpkg when interface in non-English language.

- Fix conditional replacement not ignoring HTML formatting.

- Fix question fade time being forced when hardware acceleration on.

- Add a small margin between buttons during review.

- V2 scheduler now respects maximum interval even if it will lead to
  all buttons giving the same interval.

- Tweak margins in overview and answer button areas.

- Ignore UI events that are received after collection has been closed.

- Don’t try to import .anki(2) files as text.

- Added support for Lojban (thanks to giqtaqisi)

# Changes in 2.1.4

Released 2018-09-05.

- Fix deck list getting stuck when creating filtered deck.

- Prevent local cards being overwritten when accidentally downloading
  empty AnkiWeb collection.

- Favour mark/flag colour over suspended colour in browse screen.

- Fix new day calculation in experimental scheduler.

- Linux theme tweaks (thanks to Glutanimate).

- Disable view page button for locally added add-ons (thanks to
  upday7).

# Changes in 2.1.3

Released 2018-08-30.

- Hardware acceleration can now be toggled in the preferences screen
  on Windows/Linux.

- Disable question fade-in during review when hardware acceleration is
  off.

- Fix some add-ons leaving a blank space in the main window when Anki
  restarted.

- Fix file associations on macOS.

- Fix some unwanted text being included when pasting.

- Fix shortcut keys like space from repeatedly triggering when held
  down.

# Changes in 2.1.2

Released 2018-08-20.

- Add missing .apkg and .colpkg file associations.

- Improve handling of images inlined in fields.

# Changes in 2.1.1

Released 2018-08-09.

- Fixed exporting of .apkg files with regular scheduler.

- Work around Anki failing to start on some Windows machines.

- Fix dialogs failing to show in tabs when using macOS’s full screen
  mode.

- Extract embedded images when pasting HTML.

- Fix images copied from Finder not pasting properly.

- When the sort field is set to RTL, display in RTL order in the
  browser.

- Update toolkit version on Windows.

# Changes in 2.1.0

Released 2018-08-06.

The first stable release of Anki 2.1.

Changes from the previous release candidate:

- Don’t unmaximize when reshowing browse screen.

- Add \*.webm to attach media file selector.

- Add shortcut key for MathJax mhchem support.

# Changes in Anki 2.1

## Translations

- [日本語](http://rs.luminousspice.com/changeinanki2/)

## At a glance

- Anki 2.1 uses the same scheduling, syncing and file format as Anki
  2.0.x, so you can upgrade and downgrade at will.

- It’s built with recent support libraries (Python 3.6, Qt 5.9/5.12),
  bringing fixes for crashes, better handling of high resolution
  displays, non-Latin text, and the latest web standards.

- It requires a modern system - Windows 7+, OSX 10.10+, or a Linux
  distro from around 2014+.

- Add-ons will need to be updated to work with 2.1.

## Add-ons

Add-ons need to be updated to work with Anki 2.1. [Some
add-ons](https://ankiweb.net/shared/addons/2.1) have already been
updated; others have not been ported yet.

When you install 2.1, it will create a separate folder for add-ons, and
not import any existing ones you have installed.

If you’re an add-on author, you can read more about the required changes
on <https://apps.ankiweb.net/docs/addons.html>

## Other changes

Anki 2.1 brings numerous bugfixes, and some quality of life
improvements, such as:

- Built in MathJax support

- A "restore backup" option in the profiles screen

- SVG rendering support for LaTeX

- Improved add-on configuration, management and updating

- Night mode for reviewing

- Improved pasting, with less unnecessary formatting included, and
  better handling of media links. You can hold the shift key when
  pasting to allow more formatting to be included.

The Browse screen has been simplified:

- Various shortcuts that were previously in the sidebar are now
  available via the Filter button. If you find yourself frequently
  accessing certain items, you can save the search to have them appear
  in the side bar.

- The top bar with common shortcuts has been removed. You can access
  all actions quickly by right clicking or ctrl+clicking in the card
  list instead, and there is an add-on available for adding back the
  top bar if you still prefer it.

A full list of changes made during the alpha and beta period are
documented [here](https://apps.ankiweb.net/docs/beta.html).

## Experimental scheduler

Anki 2.1 contains an optional, [experimental
scheduler](https://anki.tenderapp.com/kb/anki-ecosystem/experiment-scheduling-changes-in-anki-21).
The experimental scheduler is not compatible with older Anki versions.
