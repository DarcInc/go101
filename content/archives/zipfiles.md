+++
title = "zip files"
+++

Zip files are a more recent archive format dating from the late
1980's.  Unlike tar, zip files are supported on Windows.  Zip is
often used to distribute archives of files on the internet 
because it is supported in most *nix distributions and on Windows.
Unlike tar files, zip files are a compressed format.  For more
inoformation on zip files, [here](https://en.wikipedia.org/wiki/Zip_(file_format)) is the WikiPedia article.

### Creating a Zip File
Creating a new Zip involves wrapping an [<code>io.Writer</code>](https://golang.org/pkg/io/#Writer)
using the [<code>zip.NewWriter</code>](https://golang.org/pkg/archive/zip/#NewWriter) function.
It is not sufficient to just close the underlying io.Writer.  The zip.Writer must also be closed
to write out the Zip file's contents.

```Go
// Create a new writer (in this case a file) to be used by the zip.
zipFile, _ := os.OpenFile("lorem.zip", os.O_WRONLY|os.O_TRUNC|os.O_CREATE, 0644)
defer zipFile.Close()

// Wrap the writer in a zip writer
zipWriter := zip.NewWriter(zipFile)
defer zipWriter.Close()
```

### Writing Data to a Zip File
Writing data requires creating an instance of io.Writer from the zip.Writer.  
Optionally, a header can be created to add file metadata that should be used when
unpacking the zip archive.  Unlike the header for the tar archive,
the [<code>zip.FileHeader</code>](https://golang.org/pkg/archive/zip/#FileHeader) 
requires transforming values to integers.  (Convenience functions for
the file mode and modified date and time).

The [<code>zip.Writer#Create</code>](https://golang.org/pkg/archive/zip/#Writer.Create)
will create a header for you.  It returns an instance of io.Writer which can
be used to write the data.  

```Go
dataWriter, _ := zipWriter.Create("lorem.txt")
io.Copy(dataWriter, inputFile)
```

### Opening a Zip File
Much like the zip.Writer relies on the underlying io.Writer, the 
[<code>zip.Reader</code>](https://golang.org/pkg/archive/zip/#Reader) relies on an
underlying [<code>io.Reader</code>](https://golang.org/pkg/io/#Reader).  The 
function [<code>zip.NewReader</code>](https://golang.org/pkg/archive/zip/#NewReader)
encapsulates the io.Reader into a zip.Reader.  The file size is imporant and 
it is not sufficient to pass in a zero because the Zip file's table of contents
is located at the end of the file.

```Go
// In this case a filesystem file is used, but the source could be a byte 
// stream or other reader, as long as the size is known.
zipFile, _ := os.Open("lorem.zip")
defer zipFile.Close()

// Need to obtain the file size for the zip reader.  The os.Stat function can 
// be used for filesystem files.  
fileinfo, _ := os.Stat("lorem.zip")

// Create a new zip.Reader with the underlying io.Reader and file size.
zipReader, _ := zip.NewReader(zipFile, fileinfo.Size()
```

### Reading From a Zip File
The zip.Reader contains a slice with all the table of contents from the Zip
file.  The files are in [zip.File](https://golang.org/pkg/archive/zip/#File)
entries which contain the header.  The zip.File entry can be opened for
reading with the [<code>zip.File#Open</code>](https://golang.org/pkg/archive/zip/#File.Open)
method.  The method returns an 
[<code>io.ReadCloser</code>](https://golang.org/pkg/io/#ReadCloser) which needs to be closed.
Iterating over all the file entries is fairly straightforward

https://golang.org/pkg/archive/zip/#File

```Go
for _, z := range zipReader.File {
    file, _ := z.Open()
    defer file.Close()

    io.Copy(os.Stdout, file)
}
```

### Putting it All Together
The program [zipfiles/main.go](https://github.com/DarcInc/go101examples/blob/master/archives/zipfiles/main.go)
creates a zip file from a text file in the same directory change to the 
archives/zipfiles directory and <code>go run main.go</code>  The program will
create a zip file and display the difference in size between the 
original text file and the zip file.  The zip file created is a standard zip
file.  

```Go
package main

import (
	"archive/zip"
	"fmt"
	"io"
	"os"
)

func createZip() {
	zipFile, _ := os.OpenFile("lorem.zip", os.O_WRONLY|os.O_TRUNC|os.O_CREATE, 0644)
	defer zipFile.Close()

	zipWriter := zip.NewWriter(zipFile)
	defer zipWriter.Close()

	inputFile, _ := os.Open("lorem.txt")
	defer inputFile.Close()

	dataWriter, _ := zipWriter.Create("lorem.txt")
	io.Copy(dataWriter, inputFile)

	zipWriter.Flush()
}

func unpackZip() {
	zipFile, _ := os.Open("lorem.zip")
	defer zipFile.Close()

	fileinfo, _ := os.Stat("lorem.zip")
	zipReader, _ := zip.NewReader(zipFile, fileinfo.Size())

	for _, z := range zipReader.File {
		file, _ := z.Open()
		defer file.Close()

		io.Copy(os.Stdout, file)
	}
}

func fileSizes() {
	origInfo, _ := os.Stat("lorem.txt")
	zipInfo, _ := os.Stat("lorem.zip")

	fmt.Println()
	fmt.Println()
	fmt.Printf("Original %d bytes but zipped %d bytes\n", origInfo.Size(), zipInfo.Size())
}

func main() {
	createZip()
	unpackZip()
	fileSizes()
}
```