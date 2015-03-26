# Introduction #

This system simplifies management of add-ons in an [OpenCollar](http://code.google.com/p/opencollar/). It lets the user easily install, remove or update cleanly an add-on in his/her collar. This system works similarly to the OpenCollar in-place updater, with a few modifications. It can be configured by the author using a Notecard.

# Notecard format #

The notecard must start with the name "InstallerConfig". It can be followed by any text, such as the add-on name and a version number.
Any line starting with a # will be ignored and can be used to add comments to the notecard.
Any line starting with a `[` and ending with a `]` will be read as a category title. The following lines will be read as part of this category, until the next category title.
The first non-empty line that isn't a comment must be a category title.
The category titles can be placed in any order.
The following paragraphs describes each category.
In several categories, each line describes an item, either in the collar or in the installer. If the line starts ends with a `*`, then every item whose name _starts_ with what precedes the `*` will be matched. If the line doesn't end with a `*`, then only the item whose name is exactly the line will be matched. As an example
```
My Script*
```
would match both `My Script` and `My Script - 1.23`, whereas
```
My Script
```
would only match `My Script`.
Currently only ending `*`'s are supported.
`|` is a special character used by the installer and should never be used in the notecard.

## `[`Main`]` ##

This category contains general information about the add-on. All the following lines must follow the `key=value` pattern. The following keys are recognized:
  * `Name`: add-on name
  * `Author`: add-on author name
  * `Version`: version number
  * `Help`: name of a notecard that can be given to the wearer when touching the installer, or when clicking on the "help" button before the installation process (optional)
  * `MinCollarVersion`: minimal version number of the collar necessary to use the installer. The installer itself needs an OpenCollar 3.4 or higher, so this parameter will be ignored if it is less than 3.4 (optional)

## `[`Detection`]` ##

Each line is the name of an item that the installer can look for into the collar. If _all_ the items are found, then the installer knows that the add-on (or a previous version) is installed, and will propose the "upgrade" button instead of "install". Only put in there the minimal list of what would necessary to detect the add-on. As an example just the script name could be enough.

## `[`Install`]` ##

Each line in that category give the name of an item to copy to the collar when installing or upgrading, or to delete when removing. If the item is a script, it will be started in the collar at the end of installation. If you have a script with a version number, it is important to use a `*` in the name instead of the version number. If for example an old version of the add-on had a script called `MyScript - 1.0` and the new version has a script called `MyScript - 1.1`, you must have a
```
MyScript - *
```
line in the notecard. This way:
  * when installing, the script `MyScript - 1.1` will be copied into the collar
  * when upgrading, either `MyScript - 1.0` or `MyScript - 1.1` will be removed and replaced my `MyScript - 1.1`
  * when removing, either `MyScript - 1.0` or `MyScript - 1.1` will be removed.

## `[`RemoveCleanUp`]` ##

This optional category describes additional steps that must be followed by the installer when removing the add-on, after having deleted all the items listed in `[Install]`. Each line follows a `key=value` pattern. The following keys are supported:
  * `item`: the value is the name of an item that must be removed from the collar. This can be used if an older version of the add-on had more items than the new one, and you need to remove an item that isn't in the `[Install]` category, or to delete an item that could have been generated through the normal use of the add-on. The value can end with a `*` to match multiple items.
  * `httpdb`: the value is the name of an entry that must be removed from the http database after everything has been removed from the collar. Use it to remove settings that were used by the add-on
  * `localsetting`: same thing with local settings
  * `script`: the value is the name of a script that will be uploaded and started in the collar after everything has been removed from the collar. The script can perform additional clean up, and should send a message before deleting itself. Only one script can be specified.
The clean up script needs to send the string "JMAddOnInstaller|done|script" on channel -7483214 when finished. The installer will wait until it receives this message before continuing. You can use the CleanUpDone() function from the example cleanup script.

## `[`UpgradeCleanUp`]` ##

The contents use the same definition than `[RemoveCleanUp]`, but is used when upgrading the system, after having deleted all the items listed in `[Install]` and before installing them.

# Examples #

Some examples can be found in subversion, in this folder: http://code.google.com/p/jeffmantelscripts/source/browse/#svn/trunk/addoninstaller/test. You will find:
  * [an example notecard](http://code.google.com/p/jeffmantelscripts/source/browse/trunk/addoninstaller/test/InstallerConfig-JMTestAddOn.txt)
  * [an example clean up script](http://code.google.com/p/jeffmantelscripts/source/browse/trunk/addoninstaller/test/JMTestAddOn-cleanscript.lslp)