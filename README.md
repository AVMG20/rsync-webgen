# **Rsync Command Generator**

This utility is designed to help you generate the **`rsync`** command based on your requirements.

## **Usage**

First, select whether your source or destination is remote:

- **Remote Source**: Use this if you want to pull data from a remote server to your local machine.
- **Remote Destination**: Use this if you want to push data from your local machine to a remote server.

Next, enter the following details:

- **Source Server/Destination Server**: Enter the username and remote host information (e.g., **`username@remote_host`**).
- **Source Path/Destination Path**: Enter the path to the directory on the source/destination (e.g., **`/var/www/example/`**).

There are a number of options you can configure for your **`rsync`** command:

- **Dry Run**: Show what would have been transferred without actually transferring any files.
- **Archive Mode**: This mode is equivalent to **`rlptgoD`**. It preserves symbolic links, file permissions, user & group ownership, and timestamps. It also recurses into directories and copies files.
- **Compress**: Compress file data during the transfer, reducing the amount of data sent over the network.
- **Show Progress**: Show progress during file transfer. This can be useful for tracking the status of large file transfers.
- **Partial**: Keep partially transferred files. This is useful if a transfer is interrupted, as it allows the transfer to resume from where it left off rather than starting from scratch.
- **Verbose**: Increase verbosity. This option makes **`rsync`** provide more detailed information about the file transfer process.
- **Human Readable**: Output numbers in a human-readable format. This affects the output of the file sizes in the transfer summary.
- **Remove Source**: Remove synchronized files (non-dirs) from the source after transfer.
- **Delete**: Delete extraneous files from destination dirs. This will make the destination directory match the source directory exactly.
- **Update**: Skip files that are newer on the receiver.

In addition, you can also set a bandwidth limit (in KB/s), include/exclude specific files, set a file size limit, and specify the SSH port.

Once you have entered all the necessary details and selected your options, the utility will generate the corresponding **`rsync`** command. You can then copy this command to your clipboard by clicking on it.

Please be aware that this utility is meant to assist you in generating **`rsync`** commands. Always double-check the command before running it, especially if you are working with sensitive data or production servers.

## **Disclaimer**

The author(s) and contributor(s) are not responsible for any data loss or other damages that may occur as a result of using this utility. Always ensure you have a backup and understand the command you are running.

## Developing

Once you've created a project and installed dependencies with `npm install` (or `pnpm install` or `yarn`), start a development server:

```bash
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

## Building

To create a production version of your app:

```bash
npm run build
```

You can preview the production build with `npm run preview`.

> To deploy your app, you may need to install an [adapter](https://kit.svelte.dev/docs/adapters) for your target environment.
