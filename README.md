# volume-backup for Raspberry PI

An utility to backup and restore [docker volumes](https://docs.docker.com/engine/reference/commandline/volume/).

It's fork of `loomchild/volume-backup` image.

**Note**: Make sure no container is using the volume before backup or restore, otherwise your data might be damaged. See [Miscellaneous](#miscellaneous) for instructions.

## Backup

### Backup to standard output

This avoids mounting a second backup volume and allows to redirect it to a file, network, etc.

Syntax:

```bash
docker run -v [volume-name]:/volume --rm croogie/volume-backup backup - > [archive-name]
```

For example:

```bash
docker run -v some_volume:/volume --rm croogie/volume-backup backup - > some_archive.tar.bz2
```

will archive volume named `some_volume` to `some_archive.tar.bz2` archive file.

**WARNING**: This method should not be used under PowerShell on Windows as no usable backup will be generated.

### Backup to a file

Syntax:

```bash
docker run -v [volume-name]:/volume -v [output-dir]:/backup --rm croogie/volume-backup backup [archive-name]
```

For example:

```bash
docker run -v some_volume:/volume -v /tmp:/backup --rm croogie/volume-backup backup some_archive
```

will archive volume named `some_volume` to `/tmp/some_archive.tar.bz2` archive file.

## Restore

**WARNING**: This operation will delete all contents of the volume

### Restore from standard input

This avoids mounting a second backup volume.

**Note**: Don't forget the `-i` switch for interactive operation.

Syntax:

```bash
cat [archive-name] | docker run -i -v [volume-name]:/volume --rm croogie/volume-backup restore -
```

For example:

```bash
cat some_archive.tar.bz2 | docker run -i -v some_volume:/volume --rm croogie/volume-backup restore -
```

will clean and restore volume named `some_volume` from `some_archive.tar.bz2` archive file.

### Restore from a file

Syntax:

```bash
docker run -v [volume-name]:/volume -v [output-dir]:/backup --rm croogie/volume-backup restore [archive-name]
```

For example:

```bash
docker run -v some_volume:/volume -v /tmp:/backup --rm croogie/volume-backup restore some_archive
```

will clean and restore volume named `some_volume` from `/tmp/some_archive.tar.bz2` archive file.

### Copy volume between hosts

One good example of how you can use the output to stdout would be directly migrating the volume to a new host

Syntax:

```bash
docker run -v [volume-name]:/volume --rm croogie/volume-backup backup - |\
    ssh [receiver] docker run -i -v [volume-name]:/volume --rm croogie/volume-backup restore -
```

**Note**: In case there are no traffic limitations between the hosts you can trade CPU time for bandwidth by turning off compression as shown in the example below.

For example:

```bash
docker run -v some_volume:/volume --rm croogie/volume-backup backup -c none - |\
    ssh user@new.machine docker run -i -v some_volume:/volume --rm croogie/volume-backup restore -c none -
```

## Miscellaneous

1. Upgrade / update volume-backup

   ```bash
   docker pull croogie/volume-backup
   ```

1. Find all containers using a volume (to stop them before backing-up)

   ```bash
   docker ps -a --filter volume=[volume-name]
   ```

1. Exclude some files from the backup and send the archive to stdout

   ```bash
   docker run -v [volume-name]:/volume --rm croogie/volume-backup backup -e [excluded-glob] - > [archive-name]
   ```

1. Use different compression algorithm for better performance
   ```bash
   docker run -v [volume-name]:/volume --rm croogie/volume-backup backup -c gz - > [archive-name]
   ```
1. Show simple progress indicator using verbose `-v` flag (works both for backup and restore)
   ```bash
   docker run -v [volume-name]:/volume --rm croogie/volume-backup backup -v > [archive-name]
   ```
