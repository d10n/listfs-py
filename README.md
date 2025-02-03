# listfs-py

* `listfs` mounts file listings as a filesystem.
* Useful with `find` and `ncdu`.
* Inspect file and directory origin with `cat` or `getfattr -d`.
* Supported listing type summary (more detail below):
  * basic lines of paths
  * `find -ls`
  * `tar -c -vv`
  * tabular text: `mtime bytes path`
  * jsonl: `["mtime", bytes, "path"]`
  * jsonl: `{"t":"mtime", "m":"mode", "s":bytes, "p":"path"}`

## Installation

* `uv tool install listfs`

* If you get an installation error, make sure that the build dependencies are installed:
* Fedora: `sudo dnf install python python3-devel fuse3 fuse3-devel pkg-config gcc`
* Debian: `sudo apt install python3 python3-dev fuse3 libfuse3-dev pkg-config gcc`

## Usage
```text
listfs. Mount file listings as a FUSE filesystem.
Usage:
  listfs [-o MOUNT_OPTIONS] [--foreground]
         ([--skip-components=N] [--prefix-dir=DIR] LISTING)...
         MOUNTPOINT
  listfs -h | --help | --help-formats | --version

Arguments:
  LISTING              Path to a listing file.
                       Must be in a supported format.
                       May be compressed with zstd, gzip, bzip2, lz4, or xz.
                       See --help-formats for supported formats.
  MOUNTPOINT           Directory where the filesystem will be mounted.

Options:
  -o --options         FUSE mount options.
  --foreground         Do not fork to background.
  --skip-components=N  Skip the first N components of the path.
                       Affects the next LISTING argument.
  --prefix-dir=DIR     Prefix the path with DIR.
                       Affects the next LISTING argument.
  -h --help            Show this message and exit.
  --help-formats       Show supported listing format detail and exit.
  --version            Show the version and exit.
```

## Example usage

### Running listfs
```shell
listfs /media/backups/index1.jsonl /mnt/listfs
listfs index.jsonl archive.log listing.txt.zst /mnt/listfs
listfs --skip-components=2 list1.log --skip-components=2 list2.log /mnt
listfs --prefix-dir /b1 index1.jsonl --prefix-dir /b2 index2.jsonl /mnt
```

### Inspecting the mounted filesystem
```shell
getfattr -d /mnt/Documents
getfattr -d /mnt/Documents/myfile.txt
ls -la /mnt/Documents
cat /mnt/Documents/myfile.txt
du -hsc /mnt/Documents/*
```

## Supported listing formats

### Basic path listing
One path per line, parsed literally.
```text
/Documents/
/Documents/linux_iso_links.txt
```

### `find -ls` listing
Listing of files as created by `find -ls`
```text
   41       4 drwxr-sr-x  1 user   100        1376 Feb 11  2024 ISOs/recovery
20055 2740608 -rwxr-xr-x  1 user  user  2806382592 Dec 15 19:20 ISOs/recovery/Fedora-KDE-Live-x86_64-41-1.4.iso
```
Note: if the file was recent enough as of the time of the listing that it does not indicate the year last modified, the year is assumed based on the timestamp of the listing file.

### `tar` listing
Listing of files as created by `tar -c -vv --index-file=index.log`
(`--full-time` optional)
```text
drwxr-xr-x  user/user         0 2022-03-16 12:24 code/cpython/Modules/_ctypes/
-rw-r--r--  user/user       240 2022-03-16 12:24 code/cpython/Modules/_ctypes/_ctypes.c
```
```text
drwxr-xr-x root/root         0 2024-03-02 13:48:35.477158221 Documents/
-rw-r--r-- root/root    262144 2024-09-24 02:18:49.122856322 Documents/tux.svg
```

### tabular text
whitespace-separated fields (each field is required):
* timestamp, in seconds with optional decimal
* size (bytes)
* path (assumed to be a directory if ending with "/", otherwise assumed to be a file)
```text
1681475278.2696418010   24334   /media/data/models/LLM Papers.html
1681475278.2529749660   186175  /media/data/models/LLM Papers_files/bootstrap.min.css
```

### jsonl (array)
`jsonl`; one JSON array per line (each field is required):
* timestamp string, in seconds with optional decimal
* size (bytes)
* path (assumed to be a directory if ending with "/", otherwise assumed to be a file)
```text
["1684828592.0000000000", 14316, "/Nextcloud/Templates/Elegant.odp"]
```

### jsonl (object)
`jsonl`; one JSON object per line (only `"p"` is required):
* `"t"`: timestamp string, in seconds with optional decimal
* `"y"`: type; `"d"` directory, `"f"` file, `"l"` symlink
* `"i"`: original inode
* `"e"`: `1` if empty directory, `0` otherwise
* `"m"`: octal file mode string (e.g. `"0644"`)
* `"s"`: size (bytes)
* `"k"`: size (1KiB blocks)
* `"p"`: path
```text
{"t":"1722859284.2913177150","y":"d","i":5120416,"e":0,"m":"0755","s":4096,"k":4,"p":"/Documents/README.md"}
{"p":"/Documents/README.txt"}
```

## Notes

Note: GNU find and GNU tar produce different output than macOS / BSD find and tar.
* GNU find and tar quote/escape unusual characters in the path, while bsdtar prints the literal path
* GNU find and tar are more advanced, so you probably installed the GNU tools on macOS anyway, so listfs only supports GNU find- and tar-generated output.

## License
[GPL-3.0-or-later](./LICENSE)

## Links
* <https://github.com/d10n/listfs-py/>
* <https://pypi.org/project/listfs/>
