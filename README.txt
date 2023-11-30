=== SUMMARY ===

You deleted your only copy of an OpenPGP private key file. You wonder if the key
material can be recovered via forensic techniques such as file carving.
( https://en.wikipedia.org/wiki/File_carving ) Happily you're in luck! Dodge
(named after the key obsessed demon in Locke & Key show) may be able to help!

=== VERSION ===

1.0.0

=== USAGE ===

Basic usage:
dodge <image_file_path> <recovered_object_folder_path>

Advanced usage for those wanting to control the size of the carved out objects:
dodge <image_file_path> <recovered_object_folder_path> <byte_count_to_carve_out>

Here the image file path may be a raw device file or some disk image acquired
by some other means. If the disk image was compressed somehow before you got it,
then it must be decompressed before feeding it to Dodge. (Or other hungry demon
echoes.)

The recovery process may take some time so enjoy a nice cup of tea while Dodge
does its work. Afterwards hopefully there should be some files in the folder
you specified to hold recovered objects.

To work with and validate the recovered artifacts you'll need GnuPG which you
can get at https://gnupg.org/ if it's not already present on your system. You're
also welcome to try importing them with OpenPGP implementations as well but
their parsers may be less robust and may fail to import otherwise usable data.
If in doubt try several OpenPGP implementations before concluding the recovered
objects are unusable. Do not give up without a fight.

Worst comes to worst try editing out extra bytes after any
'-----END PGP PRIVATE KEY BLOCK-----' bytes in the recovered files.

Other than maybe GnuPG during the validation phase Dodge has no dependencies on
account of having been written in portable POSIX sh and should work out of the
box on any UNIX like system.

=== IMPORTANT NOTE FOR THE MORE TECHNICAL USERS WHO CAN ACT ON IT ===

If the deleted key is on a Solid State Drive and the drive supports the TRIM
command pray it hasn't run since the deletion. If continuous TRIM is enabled
you are likely screwed. If periodic TRIM is enabled disable it immediately
before it has a chance to run again. Keep TRIM disabled until after a rigorous,
systematic, and hopefully successful recovery attempt has been made.

=== LICENSE ===

This is a Freedom Respecting Technology. Think Open Source for everyone, not
just for well off people with reliable Internet access. Learn more at:
https://makesourcenotcode.github.io/freedom_respecting_technology.html

It's (admittedly small) Open Knowledge Set consists of:
* the main program source/executable file dodge
  licensed under GNU GPL v3 ( https://www.gnu.org/licenses/gpl-3.0.txt )
* the documentation / Help Information Set consisting of this file
  licensed under GNU FDL v1.3 ( https://www.gnu.org/licenses/fdl-1.3.txt )

=== MOTIVATION AND DEVELOPMENT NOTES ===

This tool should have been written in 2018 when I dealt with exactly this
situation. It took four days and the help of two others for me to recover the
data. This is what I wish was available to me at the time.

The approach we used was different than what's used here and in many ways
less robust in terms of the number of things that could have gone wrong. Though
the details of that story aren't part of Dodge's official documentation curious
people can read about it at:
https://makesourcenotcode.github.io/LispNYCSlides.pdf

That story is Thoroughly Entertaining. But for those short on time pages 67-75
contain the critical details you need to try that approach as well. Page 80 of
the slides is the approach I believe in retrospect would have been best.

Namely find all offsets for the '----BEGIN PGP PRIVATE KEY BLOCK-----' bytes in
the image, read some chunk of data from each such offset storing it in a
temporary file, try importing that temporary file with GnuPG which shouldn't be
bothered by any junk bytes at the end. Or worst case scenario GnuPG should
import your key before crashing.

Dodge was tested both on the Fedora Linux laptop I use day to day as well as a
FreeBSD virtual machine running in VirtualBox. Smaller synthetic images
embedding key material between random bytes were also used.

=== LIMITATIONS AND DIRECTIONS FOR FUTURE WORK ===

Patches or pull requests addressing these will be gratefully received.

1:

This tool only works in the cases of an ASCII armored key file that was deleted.
Private keys exported in the binary format are not yet supported but may be in
future releases.

2:

This tool also may not work for private keys that were generated but not
exported into separate key files that later got deleted somehow. This is because
there's no telling how the OpenPGP implementation you're using stores they key
material internally and that may look quite different than the standardized
textual or binary structures found in exported keys. Reverse engineering this
stuff for popular OpenPGP implementations and using it in future versions may
prove a fun project.

3:

Dodge also assumes that the deleted key material is contiguous on disk which is
often but not always true. Perhaps a much more advanced tool could handle such
scenarios.

=== OTHER TOOLS TO LOOK INTO IF THIS ONE FAILS ===

1:

Scalpel: https://github.com/sleuthkit/scalpel

It doesn't seem to have any kin of support for OpenPGP data formats. However it
may be possible to feed a line to scalpel.conf approximately like:
NONE y 65536 "----BEGIN PGP PRIVATE KEY BLOCK-----" "-----END PGP PRIVATE KEY BLOCK-----" REVERSE

You may need to hex encode the header and footer bits above as it's unclear from
my cursory inspection how the config file parser handles spaces in each column.

2:

The Sleuth Kit: https://github.com/sleuthkit/sleuthkit

If the deleted key was by some miracle on an Ext2 filesystem you can follow the
procedure defined in section 11.3.1 of the fantastic book at:
https://www.linuxleo.com/Docs/LinuxLeo_4.97.pdf

If you're using Ext3+ you're out of luck as they're not friendly to deleted file
recovery based on filesystem metadata.

3:

TestDisk: https://www.cgsecurity.org/wiki/TestDisk

Apparently it has undelete functionality for some older filesystems.

4:

PhotoRec: https://www.cgsecurity.org/wiki/PhotoRec

A file carver which seems to have at least some support for carving out ASCII
armored private keys. Sadly it didn't work against the synthetic test images I
fed it but your luck may be better.

=== CHANGELOG ===

v1.0.0: initial implementation
