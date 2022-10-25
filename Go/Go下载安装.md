# Go下载安装

> https://go.dev/doc/install

# Download and install

Download and install Go quickly with the steps described here.

For other content on installing, you might be interested in:

- [Managing Go installations](https://go.dev/doc/manage-install) -- How to install multiple versions and uninstall.
- [Installing Go from source](https://go.dev/doc/install/source) -- How to check out the sources, build them on your own machine, and run them.

## 1. Go download.

Click the button below to download the Go installer.

[**Download Go for Windows**go1.19.2.windows-amd64.msi (135 MB)](https://go.dev/dl/go1.19.2.windows-amd64.msi)

Don't see your operating system here? Try one of the [other downloads](https://go.dev/dl/).

**Note:** By default, the `go` command downloads and authenticates modules using the Go module mirror and Go checksum database run by Google. [Learn more.](https://go.dev/dl)

## 2. Go install.

Select the tab for your computer's operating system below, then follow its installation instructions.

Linux Mac Windows

1. Open the MSI file you downloaded and follow the prompts to install Go.

   By default, the installer will install Go to `Program Files` or `Program Files (x86)`. You can change the location as needed. After installing, you will need to close and reopen any open command prompts so that changes to the environment made by the installer are reflected at the command prompt.

2. Verify that you've installed Go.

   1. In **Windows**, click the **Start** menu.

   2. In the menu's search box, type `cmd`, then press the **Enter** key.

   3. In the Command Prompt window that appears, type the following command:

      ```
      $ go version
      ```

   4. Confirm that the command prints the installed version of Go.

## 3. Go code.

You're set up! Visit the [Getting Started tutorial](https://go.dev/doc/tutorial/getting-started.html) to write some simple Go code. It takes about 10 minutes to complete.