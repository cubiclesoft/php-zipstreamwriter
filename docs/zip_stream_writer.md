ZipStreamWriter Class:  'support/zip_stream_writer.php'
=======================================================

This class implements a streaming-capable ZIP file generator in pure PHP userland without extensions (although zlib is recommended to compress files).

Requires the DeflateStream and CRC32Stream classes to function.

32-bit PHP is supported but not recommended.  PHP uses signed integers, which means anything larger than 0x7FFFFFFF can become a negative value in 32-bit PHP.

Note that there are MANY constants in the ZipStreamWriter class not shown here in the documentation.

Example usage:

```php
<?php
	require_once "support/zip_stream_writer.php";

	$fp = fopen("test.zip", "wb");

	$zip = new ZipStreamWriter();
	$zip->Init();

	$lastmodified  = mktime(9, 41, 20, 12, 10, 2018);

	// Add an empty directory.
	$zip->AddDirectory("dir/", array("last_modified" => $lastmodified));

	// Add smaller files with a simple call.
	$zip->AddFileFromString("dir/ Hello.txt", "Hello there!", array());
	fwrite($fp, $zip->Read());

	// Finalize and close the ZIP file.
	$zip->Finalize();

	fwrite($fp, $zip->Read());
	fclose($fp);
```

See the main [ZipStreamWriter for PHP repository](https://github.com/cubiclesoft/php-zipstreamwriter) for additional usage.

ZipStreamWriter::IsDeflateSupported()
-------------------------------------

Access:  public static

Parameters:  None.

Returns:  A boolean containing whether or not zlib streaming filters for Deflate compression are supported.

This function returns whether or not Deflate compression will work.  If not, ZipStreamWriter will only store and return uncompressed data.

ZipStreamWriter::StartHTTPResponse($filename)
---------------------------------------------

Access:  public static

Parameters:

* $filename - A string containing the filename to recommend the user save the file as.

Returns:  Nothing.

This function adjust various PHP settings and outputs relevant HTTP headers to trigger a download in the user's web browser.

ZipStreamWriter::Init()
-----------------------

Access:  public

Parameters:  None.

Returns:  A boolean of true.

This function initializes the class.  The class can be reused by calling Init().

ZipStreamWriter::AddDirectory($path, $options = array())
--------------------------------------------------------

Access:  public

Parameters:

* $path - A string containing a path name.
* $options - A array of options compatible with `InitCentralDirHeader()` (Default is array()).

Returns:  A boolean of true if the operation was successful, false otherwise.

This function adds an empty directory to the ZIP file.  Most ZIP file reader tools require directory hierarchies to be completely spelled out.

The following $options are reserved:  64bit, crc32, uncompressed_size, compressed_size, bytes_left, filename, disk_start_num, header_offset.

ZipStreamWriter::AddFileFromString($filename, $data, $options = array(), $compresslevel = -1)
---------------------------------------------------------------------------------------------

Access:  public

Parameters:

* $filename - A string containing the UTF-8 filename to use.
* $data - A string containing the binary data to compress/store.
* $options - A array of options compatible with `InitCentralDirHeader()` (Default is array()).
* $compresslevel - An integer representing the compression level to use (Default is -1).

Returns:  A boolean of true if the operation was successful, false otherwise.

This function adds a file entry to the ZIP file and its contents all at one time.  Useful for adding smaller files (e.g. under 1MB).  It compresses the data in advance and if the compressed version is larger than the original (fairly rare and generally only happens when files are under 100 bytes in size), compression is disabled for the file and the data is stored uncompressed instead.

Note that any directory structure referenced by the $filename should be specified in one or more calls to `AddDirectory()` prior to calling this function.

The following $options are reserved:  64bit, crc32, uncompressed_size, compressed_size, bytes_left, filename, disk_start_num, header_offset.

ZipStreamWriter::InitFile($filename, $options = array(), $compresslevel = -1)
-----------------------------------------------------------------------------

Access:  public

Parameters:

* $filename - A string containing the UTF-8 filename to use.
* $options - A array of options compatible with `InitCentralDirHeader()` (Default is array()).
* $compresslevel - An integer representing the compression level to use (Default is -1).

Returns:  A boolean of true if the operation was successful, false otherwise.

This function adds a file entry to the ZIP file.  After this function, calls to `AppendFileData()` may be made to process and write the data contents.

Note that any directory structure referenced by the $filename should be specified in one or more calls to `AddDirectory()` prior to calling this function.

The following $options are reserved:  crc32, compressed_size, bytes_left, filename, disk_start_num, header_offset.

The following extra field is reserved:  Zip64.

ZipStreamWriter::AppendFileData($data)
--------------------------------------

Access:  public

Parameters:

* $data - A string containing the data to process and write.

Returns:  A boolean of true if the operation was successful, false otherwise.

This function adds another chunk of data to current file entry in the ZIP file.  The internal CRC32 stream instance is updated.  The data is processed through DeflateStream according to whether or not compression is enabled for the file.

ZipStreamWriter::CloseFile()
----------------------------

Access:  public

Parameters:  None.

Returns:  A boolean of true if the operation was successful, false otherwise.

This function closes a file in the ZIP file, writes an optional data descriptor for streaming files, and registers the file entry with the central directory.

Writing all bytes of data prior to calling this function when the size was declared to be known during the Init() call is required by this function.

ZipStreamWriter::Finalize($comment = "")
----------------------------------------

Access:  public

Parameters:

* $comment - A string containing a file comment to use (Default is "").

Returns:  A boolean of true if the operation was successful, false otherwise.

This function finalizes the whole ZIP file.  It writes out the Central Directory, Zip64 End of Central Directory record/locator, and End of Central Directory record.

Note that many ZIP file readers do not display ZIP file comments without clicking an Info button.  Some people have, over the years, placed EULA-style information into a ZIP file comment field.  Since most users will not see ZIP file comments and are not required to click on anything to contractually agree to them, the EULA in such files is not legally enforceable in most jurisdictions.

ZipStreamWriter::BytesAvailable()
---------------------------------

Access:  public

Parameters:  None.

Returns:  An integer containing the number of available bytes that can be read with a Read() call.

This function returns the size of the internal output buffer that will be returned during the next Read() call.  This function can be used to wait for the buffer to accumulate to a certain point before retrieving the output (e.g. 65KB) so that output buffer thrashing is kept to a minimum.

ZipStreamWriter::Read()
-----------------------

Access:  public

Parameters:  None.

Returns:  A string containing the current output buffer.

This function returns the available output and also clears the internal output.  This function can be called at any time.  However, to avoid output buffer thrashing and increase overall efficiency, both internally to the class and externally to PHP (e.g. fwrite(), echo, print), calling `BytesAvailable()` can be used to send the data to its final destination in larger chunks.

ZipStreamWriter::AppendZip64ExtraField(&$extrafields, $origsize)
----------------------------------------------------------------

Access:  public static

Parameters:

* $extrafields - An array to append the Zip64 extra field to.
* $origsize - An integer containing the original size.

Returns:  Nothing.

This internal static function appends a Zip64 extra field.  This particular function should not be used by applications.

ZipStreamWriter::AppendOS2ExtraField(&$extrafields, $os2extendedattrs)
----------------------------------------------------------------------

Access:  public static

Parameters:

* $extrafields - An array to append the OS/2 extra field to.
* $os2extendedattrs - A string containing OS/2 extended attributes.

Returns:  Nothing.

This static function appends a OS/2 extra field.  Only included because the ZIP file specification made the structure clear.  This function isn't actually useful since OS/2 is a fairly dead OS.

ZipStreamWriter::AppendNTFSExtraField(&$extrafields, $lastmodified, $lastaccessed, $created)
--------------------------------------------------------------------------------------------

Access:  public static

Parameters:

* $extrafields - An array to append the NTFS extra field to.
* $lastmodified - An integer containing a UNIX timestamp.
* $lastaccessed - An integer containing a UNIX timestamp.
* $created - An integer containing a UNIX timestamp.

Returns:  Nothing.

This static function appends a NTFS extra field.  The specification has only ever defined one "tag" for this field but could support additional tags such as symbolic links (reparse points) and alternate data streams.

ZipStreamWriter::AppendUNIXExtraField(&$extrafields, $lastaccessed, $lastmodified, $uid, $gid, $extradata = "")
---------------------------------------------------------------------------------------------------------------

Access:  public static

Parameters:

* $extrafields - An array to append the NTFS extra field to.
* $lastaccessed - An integer containing a UNIX timestamp.
* $lastmodified - An integer containing a UNIX timestamp.
* $uid - An integer containing a user/owner ID.
* $gid - An integer containing a group ID.
* $extradata - A string containing variable information (e.g. a symlink target).

Returns:  Nothing.

This static function appends a UNIX extra field.

Note that $lastmodified and $lastaccessed are flipped from `AppendNTFSExtraField()`.  These functions mirror the specification and the specification is inconsistent in its ordering of fields with relation to how cross-platform software might be developed to create ZIP files.

ZipStreamWriter::GenerateExtraFieldsStr($dir, $centraldir)
----------------------------------------------------------

Access:  protected static

Parameters:

* $dir - An array containing Central Directory information.
* $centraldir - A boolean indicating whether or not the central directory is being written.

Returns:  A string containing the generated extra fields data blob.

This internal static function converts a set of extra fields from an array to a string suitable for output.

ZipStreamWriter::MakeUnixExternalAttribute($mode, $unixtype = ZipStreamWriter::FILE_ATTRIBUTE_UNIX_DEVICE_REGULAR)
------------------------------------------------------------------------------------------------------------------

Access:  public static

Parameters:

* $mode - An integer containing a file mode (e.g. 0644).
* $unixtype - An integer containing a ZipStreamWriter UNIX file attribute constant (Default is ZipStreamWriter::FILE_ATTRIBUTE_UNIX_DEVICE_REGULAR).

Returns:  An integer containing a ZIP file compatible value for UNIX sources.

This static function calculates the external file attributes for the UNIX-specific portion of a file.  Prefer passing `unix_attrs` into various functions that accept `InitCentralDirHeader()` options arrays instead of calling this function directly.

ZipStreamWriter::InitCentralDirHeader($options = array(), $dostype = ZipStreamWriter::FILE_ATTRIBUTE_MSDOS_ARCHIVE)
-------------------------------------------------------------------------------------------------------------------

Access:  public static

Parameters:

* $options - An array containing options to use for a new Central Directory entry (Default is array()).
* $dostype - An integer containing a ZipStreamWriter MS-DOS file attribute constant (Default is ZipStreamWriter::FILE_ATTRIBUTE_MSDOS_ARCHIVE).

Returns:  An array containing merged and calculated options.

This static function creates a Central Directory entry with platform-specific values.  On Windows, it will tend to generate MS-DOS entries while other platforms will tend to generate UNIX entries.

The $options array accepts the following options:

* made_by_version - A double containing the ZIP version that generated the ZIP file (Default is 6.3).
* made_by_os - An integer containing a ZipStreamWriter MADE_BY_... constant (Default is ZipStreamWriter::MADE_BY_MSDOS or ZipStreamWriter::MADE_BY_UNIX depending on platform and inputs).
* version_required - A double containing the minimum ZIP version that is required to correctly process the entry (Default is 0 here, but variable depending on other functions).
* flags - An integer containing a bitfield of ZipStreamWriter flags (Default is 0).  There is little reason to ever specify this.
* compress_method - An integer containing either ZipStreamWriter::COMPRESS_METHOD_STORE or ZipStreamWriter::COMPRESS_METHOD_DEFLATE (Default is ZipStreamWriter::COMPRESS_METHOD_STORE).
* last_modified - An integer containing a UNIX timestamp (Default is 0).  Note that this will ultimately be converted to a MS-DOS FAT timestamp, which has a 2-second resolution.
* 64bit - A boolean that specifies whether or not to output Zip64 information for the entry (Default is false).  Not all ZIP readers support Zip64.
* crc32 - An integer containing the CRC-32 for an entry (Default is 0).  It is not possible to specify this.  The various class functions will automatically calculate this value.
* compressed_size - An integer containing the size of the compressed data (Default is 0).  It is not possible to specify this.  The various class functions will automatically calculate this value.
* uncompressed_size - An integer containing the size of the uncompressed data (Default is 0).  Specifying this will have various impacts such as enabling Zip64 when relevant.
* bytes_left - An internal integer containing the number of bytes of uncompressed data to accept (Default is 0).  It is not possible to directly specify this.
* filename - A string containing the path/filename to use for the entry (Default is "").  It is not possible to directly specify this.
* extra_fields - An array containing items to use for the ZIP entry's extra field (Default is array()).
* comment - A string containing a comment to use for the entry in the Central Directory (Default is "").  Some tools show this alongside the entry (e.g. 7-Zip).
* disk_start_num - An integer containing the starting disk of the entry (Default is 0).  It is not possible to directly specify this.
* internal_file_attrs - An integer containing internal ZIP file attributes (Default is 0).  These are reserved by PKWARE and almost always 0.
* external_file_attrs - An integer containing external (host) ZIP file attributes (Default is `$dostype [ | MakeUnixExternalAttribute($options["unix_attrs"])]`).
* header_offset - An integer containing the header offset of the local file header for the Central Directory entry (Default is 0).  It is not possible to directly specify this.
* unix_attrs - An optional integer containing UNIX file attributes.  When specified, it is usually a file mode declared in octal (e.g. 0644).

ZipStreamWriter::WriteLocalFileHeader()
---------------------------------------

Access:  protected

Parameters:  None.

Returns:  Nothing.

This internal function writes a local file header to the output buffer using the most current Central Directory entry information.

ZipStreamWriter::WriteDataDescriptor()
--------------------------------------

Access:  protected

Parameters:  None.

Returns:  Nothing.

This internal function writes a data descriptor to the output buffer if the current Central Directory entry flags have the ZipStreamWriter::FLAG_USE_DATA_DESCRIPTOR flag specified.

ZipStreamWriter::GetDOSDate($ts)
--------------------------------

Access:  public static

Parameters:

* $ts - An integer containin a UNIX timestamp.

Returns:  An short integer containing a MS-DOS date.

This static function returns a MS-DOS FAT compatible date.  The ZIP file format was originally designed for the MS-DOS FAT file system.

ZipStreamWriter::GetDOSTime($ts)
--------------------------------

Access:  public static

Parameters:

* $ts - An integer containin a UNIX timestamp.

Returns:  An short integer containing a MS-DOS date.

This static function returns a MS-DOS FAT compatible time.  The ZIP file format was originally designed for the MS-DOS FAT file system.  Note that MS-DOS FAT only has a 2 second resolution (0, 2, 4, 6, ..., 56, 58, 60).

ZipStreamWriter::SetUInt8(&$data, &$x, $val)
--------------------------------------------

Access:  public static

Parameters:

* $data - A string containing data to write to.
* $x - An integer containing the starting position of the UInt8.
* $val - An integer containing the value to write.

Returns:  Nothing.

This static function converts the value to a 1-byte, little-endian string and calls `SetBytes()`.

ZipStreamWriter::SetUInt16(&$data, &$x, $val)
---------------------------------------------

Access:  public static

Parameters:

* $data - A string containing data to write to.
* $x - An integer containing the starting position of the UInt16.
* $val - An integer containing the value to write.

Returns:  Nothing.

This static function converts the value to a 2-byte, little-endian string and calls `SetBytes()`.

ZipStreamWriter::SetUInt32(&$data, &$x, $val)
---------------------------------------------

Access:  public static

Parameters:

* $data - A string containing data to write to.
* $x - An integer containing the starting position of the UInt32.
* $val - An integer containing the value to write.

Returns:  Nothing.

This static function converts the value to a 4-byte, little-endian string and calls `SetBytes()`.

ZipStreamWriter::SetUInt64(&$data, &$x, $val)
---------------------------------------------

Access:  public static

Parameters:

* $data - A string containing data to write to.
* $x - An integer containing the starting position of the UInt64.
* $val - An integer (or float for 32-bit PHP) containing the value to write.

Returns:  Nothing.

This static function converts the value to an 8-byte, little-endian string and calls `SetBytes()`.

ZipStreamWriter::SetBytes(&$data, &$x, $val, $size)
---------------------------------------------------

Access:  public static

Parameters:

* $data - A string containing data to write to.
* $x - An integer containing the starting position of the UInt64.
* $val - A string containing the value to write.
* $size - An integer containing the number of bytes to write.

Returns:  Nothing.

This static function copies the value to the data inline.  If the source value length is less than the size, zeroes pad out the remainder.
