---
title: "Elevating Your Go Code: Struct Tags and Sophisticated Field-Level Access Control"
date: 2025-03-01T15:03:11+08:00
draft: false
---

In Go, structures primarily serve to define complex data types. Struct tags, strings appended to struct fields, offer a method for storing metadata about these fields. However, these tags don't directly impact the program's logic during runtime.

This article delves into the concept of struct tags in Go, exploring what they are, their practical uses, and concludes with a real-world example to deepen our understanding of struct tags.

## What is a Struct Tag?

Consider the struct `Person` type with defined tags:

```go
type Person struct {
    Name string `json:"name"`
    Age  int    `json:"age"`
}
```

In this example, `json:"name"` and `json:"age"` are examples of struct tags. Their usage is intuitive; they are defined as key-value pairs enclosed in backticks, following the struct field.

## Their Purpose?

What's the utility of these tags, and why are they necessary?

Taking this example, suppose you are employing the `Person` struct within the `"encoding/json"` library. It dictates to Go how to transform field names during the JSON serialization and deserialization processes.

Let's demonstrate this with its application in `"encoding/json"`.

```go
p := Person{Name: "John", Age: 30}
jsonData, err := json.Marshal(p)
if err != nil {
    log.Println(err)
}
fmt.Println(string(jsonData)) 
```

Output: 
```json
{"name":"John","age":30}
```

Notice how the JSON output keys are `name` and `age`, not `Name` and `Age`.

In comparison to other languages, Go's struct tags bear a resemblance to Java annotations or C# attributes, but they are more streamlined and primarily accessed at runtime through reflection.

This design is a testament to Go's ethos: simplicity, directness, and efficacy, albeit with a trade-off in functionality.

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-26-struct-tag-uses-in-golang-02-v2.png)

## Common usecases

Struct tags in Go find frequent usage in:

### JSON/XML Serialization and Deserialization

As earlier illustrated, tags can dictate how struct fields are converted to/from JSON or XML in libraries like `encoding/json` or `encoding/xml`.

### Database Operations

In ORM (Object-Relational Mapping) frameworks, tags can specify database table column names, types, and other attributes.

For instance, using Gorm:

```go
type User struct {
    gorm.Model
    Name   string `gorm:"type:varchar(100);unique_index"`
    Age    int    `gorm:"index:age"`
    Active bool   `gorm:"default:true"`
}
```

### Data Validation

In certain libraries, tags are utilized for data validation, such as verifying the validity of an email address in a field.

An example with [govalidator](github.com/asaskevich/govalidator):

```go
type User struct {
    Email string `valid:"email"`
    Age   int    `valid:"range(18|99)"`
}
```

Here, the `valid` tag sets rules for field validation.

### Customizing Tag Behavior

Customizing tag behavior requires understanding Go's reflection mechanism.

Example:

```go
type Person struct {
    Name string `mytag:"MyName"`
}

t := reflect.TypeOf(Person{})
field, _ := t.FieldByName("Name")
fmt.Println(field.Tag.Get("mytag")) // Output: MyName
```

## Example: Struct Field Access Control

Consider a scenario of a struct access control system based on roles like `admin`, `user`, or request sources.

Let's define a struct type:

```go
type UserProfile struct {
    Username    string `access:"user"`  // Accessible to all users
    Email       string `access:"user"`  // Accessible to all users
    PhoneNumber string `access:"admin"` // Only for admins
    Address     string `access:"admin"` // Only for admins
}
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-26-struct-tag-uses-in-golang-04-v1.png)

In this structure, PhoneNumber and Address are sensitive fields, visible only to users with the admin role. Conversely, Username and Email are accessible to all users.

The subsequent task is to develop a function that dictates the visibility of fields in UserProfile, guided by the specified access tags.

Let's assume the function is named FilterFieldsByRole:

```go
func FilterFieldsByRole(profile UserProfile, role string) map[string]string {
    result := make(map[string]string)
    val := reflect.ValueOf(profile)
    typ := val.Type()

    for i := 0; i < val.NumField(); i++ {
        field := typ.Field(i)
        accessTag := field.Tag.Get("access")
        if accessTag == "user" || accessTag == role {
            fieldName := strings.ToLower(field.Name) 
            fieldValue := val.Field(i).String() 
            result[fieldName] = fieldValue
        }
    }
    return result
}
```

Usage:

```go
func main() {
    profile := UserProfile{
        Username:    "johndoe",
        Email:       "johndoe@example.com",
        PhoneNumber: "123-456-7890",
        Address:     "123 Elm St",
    }

    userInfo := FilterFieldsByRole(profile, "user")
    fmt.Println(userInfo)

    adminInfo := FilterFieldsByRole(profile, "admin")
    fmt.Println(adminInfo)
}
```

Output:

```bash
map[username:johndoe email:johndoe@example.com]
map[username:johndoe email:johndoe@example.com phonenumber:123-456-7890 address:123 Elm St]
```

This example showcases role-based access control with custom struct tags.

Undoubtedly, this code is more lucid and easier to maintain, boasting considerable flexibility and scalability. Furthermore, it facilitates the effortless addition of more roles.

## Conclusion

In summary, this article presented the fundamentals of struct tags in Go, their functionality, and diverse applications. The final example demonstrated how struct tags can make our code more adaptable and potent.

Blog post: [Using Struct Tags in Go: Field-Level Access Control](https://en.poloxue.com/posts/2024-01-26-struct-tag-uses-in-golang/)
