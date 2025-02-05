# sqlutil
[![Build Status](https://github.com/allisson/sqlutil/workflows/Release/badge.svg)](https://github.com/allisson/sqlutil/actions)
[![Go Report Card](https://goreportcard.com/badge/github.com/allisson/sqlutil)](https://goreportcard.com/report/github.com/allisson/sqlutil)
[![go.dev reference](https://img.shields.io/badge/go.dev-reference-007d9c?logo=go&logoColor=white&style=flat-square)](https://pkg.go.dev/github.com/allisson/sqlutil)

A collection of helpers to deal with database.

Example:

```golang
package main

import (
	"context"
	"database/sql"
	"fmt"
	"log"

	"github.com/allisson/sqlutil"
	_ "github.com/lib/pq"
)

type Player struct {
	ID   string `db:"id"`
	Name string `db:"name" fieldtag:"update"`
	Age  int    `db:"age" fieldtag:"update"`
}

func main() {
	// Connect to database
	db, err := sql.Open("postgres", "postgres://user:pass@localhost/sqlutil?sslmode=disable")
	if err != nil {
		log.Fatal(err)
	}
	err = db.Ping()
	if err != nil {
		log.Fatal(err)
	}
	defer db.Close()

	// Create table
	_, err = db.Exec(`
		CREATE TABLE IF NOT EXISTS players(
			id VARCHAR PRIMARY KEY,
			name VARCHAR NOT NULL,
			age INTEGER NOT NULL
		)
	`)
	if err != nil {
		log.Fatal(err)
	}

	// Insert players
	r9 := Player{
		ID:   "R9",
		Name: "Ronaldo Fenômeno",
		Age:  44,
	}
	r10 := Player{
		ID:   "R10",
		Name: "Ronaldinho Gaúcho",
		Age:  41,
	}
	flavour := sqlutil.PostgreSQLFlavor
	tag := "" // empty tag will use all fields from struct
	ctx := context.Background()
	if err := sqlutil.Insert(ctx, db, sqlutil.PostgreSQLFlavor, tag, "players", &r9); err != nil {
		log.Fatal(err)
	}
	if err := sqlutil.Insert(ctx, db, sqlutil.PostgreSQLFlavor, tag, "players", &r10); err != nil {
		log.Fatal(err)
	}

	// Get player
	findOptions := sqlutil.NewFindOptions(flavour).WithFilter("id", "R10")
	if err := sqlutil.Get(ctx, db, "players", findOptions, &r10); err != nil {
		log.Fatal(err)
	}
	fmt.Printf("%#v\n", r10)

	// Select players
	players := []*Player{}
	findAllOptions := sqlutil.NewFindAllOptions(flavour).WithLimit(10).WithOffset(0).WithOrderBy("name asc")
	if err := sqlutil.Select(ctx, db, "players", findAllOptions, &players); err != nil {
		log.Fatal(err)
	}
	for _, p := range players {
		fmt.Printf("%#v\n", p)
	}

	// Update player
	tag = "update" // will use fields with fieldtag:"update"
	r10.Name = "Ronaldinho Bruxo"
	if err := sqlutil.Update(ctx, db, sqlutil.PostgreSQLFlavor, tag, "players", r10.ID, &r10); err != nil {
		log.Fatal(err)
	}

	// Delete player
	if err := sqlutil.Delete(ctx, db, sqlutil.PostgreSQLFlavor, "players", r9.ID); err != nil {
		log.Fatal(err)
	}
}
```
