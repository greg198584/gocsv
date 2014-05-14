Go CSV
=====

The GoCSV package aims to provide easy serialization and deserialization functions to use CSV in Golang

API and techniques inspired from http://labix.org/mgo

[![GoDoc](https://godoc.org/github.com/JonathanPicques/gocsv?status.png)](https://godoc.org/github.com/JonathanPicques/gocsv)

Full example
=====

Consider the following CSV file

```csv

client_id,client_name,client_age
1,Jose,42
2,Daniel,26
3,Vincent,32

```

Easy binding in Go!
---

```go

package main

import (
	"fmt"
	"gocsv"
	"os"
)

type Client struct { // Our example struct, you can use "-" to ignore a field
	Id      string `csv:"id"`
	Name    string `csv:"name"`
	Age     string `csv:"age"`
	NotUsed string `csv:"-"`
}

func main() {
	clientsFile, err := os.OpenFile("clients.csv", os.O_RDWR|os.O_CREATE, os.ModePerm)
	if err != nil {
		panic(err)
	}
	defer clientsFile.Close()

	clients := []*Client{}

	if err := gocsv.UnmarshalFile(clientsFile, &clients); err != nil { // Load clients from file
		panic(err)
	}
	for _, client := range clients {
		fmt.Println("Hello", client.Name)
	}

	if _, err := clientsFile.Seek(0, 0); err != nil { // Go to the start of the file
		panic(err)
	}

	clients = append(clients, &Client{Id: "12", Name: "John", Age: "21"}) // Add clients
	clients = append(clients, &Client{Id: "13", Name: "Fred"})
	clients = append(clients, &Client{Id: "14", Name: "James", Age: "32"})
	clients = append(clients, &Client{Id: "15", Name: "Danny"})
	csvContent, err := gocsv.MarshalString(&clients) // Get all clients as CSV string
	//err = gocsv.MarshalFile(&clients, clientsFile) // Use this to save the CSV back to the file
	if err != nil {
		panic(err)
	}
	fmt.Println(csvContent) // Display all clients as CSV string

}

```

Customizable Converters
---

```go

type DateTime struct {
	time.Time
}

// Convert the internal date as CSV string
func (date *DateTime) MarshalCSV() (string, error) {
	return date.Time.Format("20060201"), nil
}

// Convert the CSV string as internal date
func (date *DateTime) UnmarshalCSV(csv string) (err error) {
	date.Time, err = time.Parse("20060201", csv)
	if err != nil {
		return err
	}
	return nil
}

type Client struct { // Our example struct with a custom type (DateTime)
	Id       string   `csv:"id"`
	Name     string   `csv:"name"`
	Employed DateTime `csv:"employed"`
}

```

Customizable CSV Reader / Writer
---

```go

func main() {
	...
	
	gocsv.SetCSVReader(func(in io.Reader) *csv.Reader {
    	//return csv.NewReader(in)
    	return gocsv.LazyCSVReader(in) // Allows use of quotes in CSV
    })

    ...

    gocsv.UnmarshalFile(file, &clients)

    ...

    gocsv.SetCSVWriter(func(out io.Writer) *csv.Writer {
    	return csv.NewWriter(out)
    })

    ...

    gocsv.MarshalFile(&clients, file)

	...
}

```