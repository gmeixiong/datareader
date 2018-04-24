datareader : read SAS and Stata files in Go
=========================

__datareader__ is a pure [Go](https://golang.org) (Golang) package
that can read binary SAS format (SAS7BDAT) and Stata format (dta) data
files into native Go data structures.  For non-Go users, there are two
command line utilities that convert SAS and Stata files into text file
formats.

The Stata reader is based on the Stata documentation for the [dta file
format](http://www.stata.com/help.cgi?dta) and supports dta versions
115, 117, and 118.

There is no official documentation for SAS binary format files.  The
code here is translated from the Python
[sas7bdat](https://pypi.python.org/pypi/sas7bdat) package, which in
turn is based on an [R
package](https://github.com/BioStatMatt/sas7bdat).  Also see
[here](https://cran.r-project.org/web/packages/sas7bdat/vignettes/sas7bdat.pdf)
for more information about the SAS7BDAT file structure.

This package also provides a simple column-oriented data container
called a `Series`.  Both the SAS reader and Stata reader return the
data as an array of `Series` objects, corresponding to the columns of
the data file.  These can in turn be converted to other formats as
needed.

Both the Stata and SAS reader support streaming access to the data
(i.e. reading the file by chunks of consecutive records).

## SAS

Here is an example of how the SAS reader can be used in a Go program
(error handling omitted for brevity):

```
import (
        "datareader"
        "os"
)

// Create a SAS7BDAT object
f, _ := os.Open("filename.sas7bdat")
sas, _ := datareader.NewSAS7BDATReader(f)

// Read the first 10000 records (rows)
ds, _ := sas.Read(10000)

// If column 0 contains numeric data
// x is a []float64 containing the dta
// m is a []bool containing missingness indicators
x, m, _ := ds[0].AsFloat64Slice()

// If column 1 contains text data
// x is a []string containing the dta
// m is a []bool containing missingness indicators
x, m, _ := ds[1].AsStringSlice()
```

## Stata

Here is an example of how the Stata reader can be used in a Go program
(again with no error handling):

```
import (
        "datareader"
        "os"
)

// Create a StataReader object
f,_ := os.Open("filename.dta")
stata, _ := datareader.NewStataReader(f)

// Read the first 10000 records (rows)
ds, _ := stata.Read(10000)
```

## CSV

The package includes a CSV reader with type inference for the column data types.

```
import (
        "datareader"
)

f, _ := os.Open("filename.csv")
rt := datareader.NewCSVReader(f)
rt.HasHeader = true
dt, _ := rt.Read(-1)
// obtain data from dt as in the SAS example above
```

## Commands

Two command-line utilities use the datareader package to allow
conversion of SAS and Stata datasets to other formats without using
Go.  Run the Makefile to compile these commands.  The executables will
be copied into your GOBIN directory.

The `stattocsv` command converts a SAS7BDAT or Stata dta file to a csv
file, it can be used as follows:

```
> stattocsv file.sas7bdat > file.csv
> stattocsv file.dta > file.csv
```

The `columnize` command takes the data from either a SAS7BDAT or a
Stata dta file, and writes the data from each column into a separate
file.  Numeric data can be stored in either binary (native 8 byte
floats) or text format (binary is considerably faster).

```
> columnize -in=file.sas7bdat -out=cols -mode=binary
> columnize -in=file.dta -out=cols -mode=text
```

## Testing

Automated testing is implemented against the Stata files used to test
the pandas Stata reader (for versions 115+):

https://github.com/pydata/pandas/tree/master/pandas/io/tests/data

A CSV data file for testing is generated by the `gendat.go` script.
There are scripts `make.sas` and `make.stata` in the test directory
that generate SAS and Stata files for testing.  SAS and Stata software
are required to run these scripts.  The generated files are provided
in the `test_files/data` directory, so `go test` can be run without
having access to SAS or Stata.

The `columnize_test.go` and `stattocsv_test.go` scripts test the
commands against stored output.

## Feedback

Please file an issue if you encounter a file that is not properly
handled.  If possible, share the file that causes the problem.