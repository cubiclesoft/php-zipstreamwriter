ZipStreamWriter for PHP
=======================

A fast, efficient streaming library for creating ZIP files on the fly in pure userland PHP without any external tools, PHP extensions, or physical disk storage requirements.  Follows version 6.3.7 of the PKWARE Zip specification.  Choose from a MIT or LGPL license.

[![Donate](https://cubiclesoft.com/res/donate-shield.png)](https://cubiclesoft.com/donate/) [![Discord](https://img.shields.io/discord/777282089980526602?label=chat&logo=discord)](https://cubiclesoft.com/product-support/github/)

If you use this project, don't forget to donate to support its development!

Features
--------

* Instantly start streaming a ZIP file from a PHP application to a user.
* Extremely low memory footprint.
* No external tools, PHP extensions (zlib is recommended though), or disk storage requirements.
* Easily add empty directories just like ZipArchive.
* Add files in their entirety or in chunks.
* Automatic, intelligent Deflate compression.  For whole files, if the uncompressed version is smaller than the compressed version, the uncompressed version is used.
* Support for NTFS timestamps and UNIX timestamps + uid/gid.
* Support for UNIX file attributes (e.g. 0644) + MS-DOS attributes.
* Automatic Zip64 large file support.
* Built using the PKWARE specification, a nice [ZIP artifact library](https://github.com/corkami/pocs/tree/master/zip), a hex editor, and sheer guts and determination.
* And much more.

Getting Started
---------------

Download/clone this repo, put the 'support' directory on a server, and then do something like this from PHP:

```php
<?php
	require_once "support/zip_stream_writer.php";

	// Adjust various PHP settings and output relevant HTTP headers to trigger a download.
	ZipStreamWriter::StartHTTPResponse("test.zip");

	$zip = new ZipStreamWriter();
	$zip->Init();

	$lastmodified  = mktime(9, 41, 20, 12, 10, 2018);
	$lastaccessed = $lastmodified + 60;
	$created = $lastmodified - 24 * 60 * 60;

	// Register some empty directories.
	$zip->AddDirectory("dir/", array("last_modified" => $lastmodified));
	$zip->AddDirectory("dir/dir2/", array("last_modified" => $lastmodified));


	// Add smaller files with a simple call.
	$zip->AddFileFromString("dir/ Hello.txt", "Hello there!", array("last_modified" => $lastmodified));

	// Keep RAM usage low by streaming data.  Just call Read() whenever to clear out the internal buffer.
	echo $zip->Read();


	// Completely customize options per file.
	// Some combinations can break some ZIP readers though (see documentation).
	$options = array(
		"last_modified" => $lastmodified,
		"64bit" => true,
		"unix_attrs" => 0644,
		"extra_fields" => array(),
		"comment" => "Best file ever."
	);

	// NTFS timestamps.
	ZipStreamWriter::AppendNTFSExtraField($options["extra_fields"], $lastmodified, $lastaccessed, $created);

	// UNIX timestamps + uid/gid.
	$uid = 1234;
	$gid = 1234;

	ZipStreamWriter::AppendUNIXExtraField($options["extra_fields"], $lastaccessed, $lastmodified, $uid, $gid);


	// Generate and compress larger files in chunks.
	$zip->OpenFile("dir/Hello2.txt", $options);

	$zip->AppendFileData("Hello ");
	if ($zip->BytesAvailable() > 65536)  echo $zip->Read();

	$zip->AppendFileData("there!");
	echo $zip->Read();

	$zip->CloseFile();
	echo $zip->Read();

	// Finalize and close the ZIP file.  Adding a comment is optional.
	$zip->Finalize("Woot!");

	// The last Read() call is required.  The rest are optional.
	echo $zip->Read();
```

See the [ZipStreamWriter::InitCentralDirHeader()](docs/zip_stream_writer.md#zipstreamwriterinitcentraldirheaderoptions--array-dostype--zipstreamwriterfile_attribute_msdos_archive) documentation for a complete list of options.

Documentation
-------------

* [ZipStreamWriter](docs/zip_stream_writer.md) - The only relevant class.
* [DeflateStream](https://github.com/cubiclesoft/ultimate-web-scraper/blob/master/docs/deflate_stream.md) - Handles Deflate stream compression.
* [CRC32Stream](https://github.com/cubiclesoft/ultimate-web-scraper/blob/master/docs/crc32_stream.md) - Handles CRC32 stream calculations.

Limitations
-----------

The following limitations are ordered from most important to least:

* Some tools that read ZIP files don't support Zip64 or streaming Zip64 extra fields.  Any files of a declared unknown uncompressed size (`uncompressed_size` = -1) or specified as over 2GB without a 64-bit declaration enables Zip64 for that file.
* Some tools that read ZIP files don't support Zip64 directory information.  The ZipStreamWriter class always creates a Zip64 End of Central Directory record and locator when more than 65,535 directories/files (total) are put into a single ZIP file.
* No encryption support, including password protection.  Some parts of the specification allude to proprietary, patented technology by PKWARE, Inc.
* Only the Store (0x0000, uncompressed) and Deflate (0x0008) compression methods are both supported and implemented.
* Deflate compression is only available if zlib stream filter support is compiled into PHP.  The class automatically falls back to the Store method if zlib stream filtering is not available or is broken.  Support for Deflate compression can be verified by calling `ZipStreamWriter::IsDeflateSupported()`.
* Incomplete disk spanning support.  The ZIP file format has the ability to span multiple disks.  Even though the ZipStreamWriter class has some code that implies support for disk spanning, the implementation is incomplete and not actually usable.  Not that this fact will actually matter for most people.
* Incomplete exotic "extra fields" support.  Only the most critical Zip64, NTFS, and UNIX extra fields are fully implemented.  The OS/2 extra field type is also implemented - because it was easy and well-documented, not because it is actually useful.  EXTRA_FIELD_WIN_NT_ACL (0x4453) support is distinctly missing.  Other extra fields can be added as binary strings.
* Limited symlink support may or may not work.  The ZIP file format was originally designed for MS-DOS and FAT16.  UNIX extra fields kind of support symbolic links but it's a tossup as to whether or not a reader will extract them correctly or just dump out a zero byte file to disk.  NTFS supports reparse points (aka symlinks and hard links), but it is up to a ZIP file reader to create reparse points and I don't know of any that do.  Symlink support is one of the major areas of the ZIP file specification that could use improvement.
* Alternate data stream support is non-existent in the ZIP file specification even though it hints at "future support" and is therefore also not supported here.
