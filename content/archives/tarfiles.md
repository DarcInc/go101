+++
title = "tar files"
+++
Tar files are a *nix standard originally used to write files into
a tape to create backups and archives.  It is rarely used for 
actual tape drives.  It is more often used to create files that 
are archives of directories or files.  Tar files are not compressed.
In many cases they are compressed after they are created using 
gzip (.gz) or bzip (.bz2).  Most users on Windows 
need to use a separate program to unpack tar files on Windows.
[Here](https://en.wikipedia.org/wiki/Tar_(computing)) is the
WikiPedia article on tar.  

### Creating a Tar file
Let's unpack [the example](https://golang.org/pkg/archive/tar/#example_) on the Go documentation site a little.  A tar writer implements 
the [io.Writer](https://golang.org/pkg/io/#Writer) interface.  By
itself, however, you can't use it to write data.  Tar is intended
to wrap another type supporting io.Writer, like a file or byte
buffer.  Calling [<code>NewWriter</code>](https://golang.org/pkg/archive/tar/#NewWriter) will return a 
tar writer that will use some underlying writer to actually
write data to a buffer or filesystem.

```Go
// Create a bytes buffer and pass that to the NewWriter 
// function.
buffer := new(bytes.Buffer)
// Now the tar writer will write data to the bytes buffer.
writer := tar.NewWriter(buffer)
```

### Writing Data
Writing data is a two step process.  The first step is to writer a 
"header" that explains what the subsequent bytes will contain.  The 
[Header](https://golang.org/pkg/archive/tar/#Header) struct 
has fields for attributes such as the file name, the file 
mode (the type of file an the file permissions), the file owner, 
the size, etc.  Writing a header does not require any data and 
sometimes it is useful to write a header that represents a directory
and will not be follwed by data.

```Go
// Create a header
myheader := &tar.Header{Name:"foo", Mode:0755}
tarWriter.WriteHeader(myheader)
```
### Opening a Tar Archive
Much like the writer implments io.Writer interface, the tar reader
implements the [io.Reader](https://golang.org/pkg/io/#Reader) interface.
The [<code>NewReader</code>](https://golang.org/pkg/archive/tar/#NewReader)
function takes something that implements an <code>io.Reader</code>
and wraps it.

```Go
file, _ := os.Open("myfile.tar")
reader := tar.NewReader(file)
```

### Reading from a Tar Archive
Reading from the tar file is also a two-step process.  The first part
is reading the header and then (if applicable) reading the bytes 
following the header.  Because the tar reader follows the io.Reader
interface, it can also be passed to the useful 
[<code>io.Copy</code>](https://golang.org/pkg/io/#Copy) function 
which will copy the bytes from the source reader to the destination
writer until the reader returns and end of file.

```Go
// First step is to get the header, which can contain the file size.
header, _ := reader.Next()
// Create a buffer large enough to hold the data.
buffer := make([]byte, header.Size)
// Read the data into the buffer.
reader.Read(buffer)
```

### Putting it All Together
The following program [tarfiles/main.go](https://github.com/DarcInc/go101examples/blob/master/archives/tarfiles/main.go) creates a tar archive and then reads data from
that archive.  In the header the <code>Name</code> and <code>Mode</code>
indicate the name of the file and permissions.  If you checkout the [example code](https://github.com/DarcInc/go101examples) and run the program
using <code>go run main.go</code>, you'll see two files created.  The first
file created is temp.tar.  If you use the *nix <code>tar -tf temp.tar</code>
command, you'll see it contains one file called "MyFile.txt".  The second 
file created is "MyFile.txt" when readArchive unpacks the archive.

```Go
package main

import (
	"archive/tar"
	"io"
	"os"
)

/*
 * This function creates an archive and writes a file
 * into the archive.
 */
func createArchive() {
	someData := []byte("The quick brown fox jumps over the lazy dog")

	file, _ := os.OpenFile("temp.tar", os.O_WRONLY|os.O_CREATE|os.O_TRUNC, 0755)
	defer file.Close()

	writer := tar.NewWriter(file)
	header := &tar.Header{Name: "MyFile.txt", Mode: 0644}
	writer.WriteHeader(header)
	writer.Write(someData)
}

/*
 * This function opens an archive for reading, reads the
 * header to get the file information, then copys the
 * data to a new file.
 */
func readArchive() {
	file, _ := os.Open("temp.tar")
	defer file.Close()

	reader := tar.NewReader(file)
	header, _ := reader.Next()

	outfile, _ := os.OpenFile(header.Name, os.O_WRONLY|os.O_CREATE|os.O_TRUNC, os.FileMode(header.Mode))
	defer outfile.Close()

	io.Copy(outfile, reader)
}

func main() {
	createArchive()
	readArchive()
}

```