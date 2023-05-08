##REST API developement using golang

##### To create a REST API using Go (Golang) and MySQL, you'll need to follow a few steps:

1. Set up the Go environment
2. Install necessary dependencies
3. Run Mysql in docker and create a MySQL database and table
4. Create the Go code with handlers and route and model [here we took books as model]
5. Run the code
6. Get results using postman

- Step 1: Install go in your system

	1. Install Go from the official website: https://golang.org/dl/
	2. Set up the GOPATH environment variable.

- Step 2: Install necessary dependencies:

	1. go get github.com/go-sql-driver/mysql
	2. go get github.com/gorilla/mux

	 - Or at last

	run `go get .`

- Step 3: Create Mysql Instance and create database with table books
	
	1. `docker run -d -p 8083:3306 --name mysql_container -e MYSQL_ROOT_PASSWORD=root mysql:latest`
	2. Access mysql in workbench
	3. create database `create database learning;`
	4. create table ```CREATE TABLE books (
  id INT AUTO_INCREMENT PRIMARY KEY,
  title VARCHAR(255) NOT NULL,
  author VARCHAR(255) NOT NULL
);```

- Step 4: Create a go code with handler, routes and models

	1. GO Imports

		``` go
		// Your Go code here
		import (
			"database/sql"
			"encoding/json"
			"fmt"
			"log"
			"net/http"

			_ "github.com/go-sql-driver/mysql"
			"github.com/gorilla/mux"
		)
		```
	2. Book Model

	```go 
	type Book struct {
	ID     string `json:"id"`
	Title  string `json:"title"`
	Author string `json:"author"`
	}
	```

	3. Initialize DM connection

	``` go
		// Initialize MySQL connection
	func initDB() {
		var err error
		db, err = sql.Open("mysql", "root:root@tcp(127.0.0.1:8083)/learning")
		if err != nil {
			log.Fatal(err)
		}

		err = db.Ping()
		if err != nil {
			log.Fatal(err)
		}

		log.Println("Connected to MySQL database")
	}

	```

	4. Get all books

	``` go
		func getBooks(w http.ResponseWriter, r *http.Request) {
		var books []Book
		rows, err := db.Query("SELECT id, title, author FROM books")
		if err != nil {
			log.Fatal(err)
		}

		defer rows.Close()

		for rows.Next() {
			var book Book
			err := rows.Scan(&book.ID, &book.Title, &book.Author)
			if err != nil {
				log.Fatal(err)
			}
			books = append(books, book)
		}

		json.NewEncoder(w).Encode(books)
	}


	```

	5. Get a single book by ID

	``` go
		func getBook(w http.ResponseWriter, r *http.Request) {
		params := mux.Vars(r)
		var book Book
		row := db.QueryRow("SELECT id, title, author FROM books WHERE id = ?", params["id"])
		err := row.Scan(&book.ID, &book.Title, &book.Author)
		if err != nil {
			log.Fatal(err)
		}

		json.NewEncoder(w).Encode(book)
	}

	```

	6. Create a new book

	``` go
		func createBook(w http.ResponseWriter, r *http.Request) {
		var book Book
		json.NewDecoder(r.Body).Decode(&book)

		result, err := db.Exec("INSERT INTO books(title, author) VALUES ( ?, ?)", book.Title, book.Author)
		if err != nil {
			log.Fatal(err)
		}

		// Get the newly inserted book ID
		bookID, _ := result.LastInsertId()

		fmt.Println(result)
		// Set the book ID in the response
		book.ID = string(bookID)

		json.NewEncoder(w).Encode(book)
	}


	```

	7. Update a book

	``` go
		func updateBook(w http.ResponseWriter, r *http.Request) {
		params := mux.Vars(r)
		var book Book
		json.NewDecoder(r.Body).Decode(&book)

		_, err := db.Exec("UPDATE books SET title = ?, author = ? WHERE id = ?", book.Title, book.Author, params["id"])
		if err != nil {
			log.Fatal(err)
		}

		json.NewEncoder(w).Encode(book)
	}

	```

	8. Delete a book

	``` go
		func deleteBook(w http.ResponseWriter, r *http.Request) {
		params := mux.Vars(r)

		_, err := db.Exec("DELETE FROM books WHERE id = ?", params["id"])
		if err != nil {
			log.Fatal(err)
		}

		w.WriteHeader(http.StatusNoContent)
	}

	```

	9. main.go

	``` go

	var db *sql.DB

	func main() {
		// Initialize router
		router := mux.NewRouter()

		// Initialize MySQL connection
		initDB()

		// API endpoints
		router.HandleFunc("/books", getBooks).Methods("GET")
		router.HandleFunc("/books/{id}", getBook).Methods("GET")
		router.HandleFunc("/books", createBook).Methods("POST")
		router.HandleFunc("/books/{id}", updateBook).Methods("PUT")
		router.HandleFunc("/books/{id}", deleteBook).Methods("DELETE")

		// Start server
		log.Fatal(http.ListenAndServe(":8000", router))
	}

	```
	- Step 5 Run the code from directory

	``` go 
	go run .
	```

	- Step 6: Get results using postman

		```
		docker pull quay.io/goswagger/swagger
		```

