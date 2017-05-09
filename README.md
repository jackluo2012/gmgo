# gmgo
Convenient wrapper around Go's MongoDB driver Mgo

## Usage:

```go
package main

import (
	"fmt"
	"github.com/narup/gmgo"
	"log"
)

var userDB gmgo.Db

####################
type User struct {
    Name string `json:"name" bson:"name"`
    Email string `json:"email" bson:"email"`
}

// Each of your data model that needs to be persisted should implment DBObject interface
func (user User) CollectionName() string {
    return "user"
}

####################

func saveNewUser() {
   session := userDB.Session()
   defer session.Close()
   
   user := &User{Name:'Puran', Email:'puran@xyz.com'}
   user.Id = bson.NewObjectId()
   userId, err := session.Save(user)
   if err != nil {
	log.Fatalf("Error saving user : %s.\n", err)
   }

   fmt.Printf("User id %s", userId)
}

func findUser(userId string) *User {
    session := userDB.Session()
    defer session.Close()
   
    user := new(User)
    if err := session.FindByID(userId, user); err != nil {
        return nil
    }
    return user
}

//Find all users
func findAllUsers() {
    session := userDB.Session()
    defer session.Close()

    users, err := session.FindAll(gmgo.Q{}, new(User)) //Note user pointer is passed to identify the collection type etc.
    if err != nil {
    	fmt.Printf("Error fetching users %s", err)
    } else {
	for _, user := range users {
	   fmt.Println(user.Name)
        }
    }
}

func setupUserDB() {
    if userDbErr := gmgo.Setup(gmgo.DbConfig{"localhost:27017", "userdb", "", ""}); userDbErr != nil {
        
    //如果 你要带上验证的一定用下面的
    //if userDbErr := gmgo.DbConfig{HostURL:"localhost:27017",Hosts:[]string{"localhost"}, DBName: "mdb", UserName: "user", Password: "pass", Mode: 1}; userDbErr != nil {
    		log.Fatalf("Database connection error : %s.\n", userDbErr)
    		return
    }

    newDb, userDbErr := gmgo.New("userdb")
    if userDbErr != nil {
        log.Fatalf("Db connection error : %s.\n", err)
    }
    userDB = newDb
}

func main() {
    //setup Mongo database connection. You can setup multiple db connections
    setupUserDB()
    user := findUser("56596608e4b07ceddcfad96e")
    if user != nil {
    	fmt.Printf("User name:%s\n", user.Name)
    } else {
	fmt.Printf("Couldn't find user")
    }
	
    findAllUsers()
}

```
