# graphb: A GraphQL query builder
__Focus on the query, not string manipulation.__

As Go developers, when there is a GraphQL server, we often build our query string like this:
```go
// Define this in some file x.go
const queryTemplate = `
    "query": "
        query an_operation_name { 
            a_field_name (
                an_argument_name_if_any: \"%s\"
            ) {
                a_sub_field,
                another_sub_field
            }
        }"`
    
// DO this in some other file y.go
func buildQuery(value string) string {
    return fmt.Sprintf(queryTemplate, value)
}
```
This approach is verbose, inflexible and error prone.

It is verbose and inflexible because every time you want to query a different structure, you need to rewrite another template and another string interpolation function. Not to mention when you have argument types such lists or enums or when you want to use fragments, directives and other syntax of GraphQL.

It is error prone because there is no way to check the correctness of your syntax until very late, when the query string is actually send to the server.

It also wastes bytes because all the extra spaces and new lines.

Because of these problems, when it comes to GraphQL client in Go, a developer spends much of her time to fight the string manipulation war (it's very easy to get string manipulation wrong and very time consuming to debug), not the query itself(aka the business logic).

This library solves all these problems for you so that you focus on your business logic.

## Example
All code are well documented. See [example](example) dir for more examples.

#### Query a mentor from Mentors API
```go
q, err := graphb.NewQuery(
    graphb.TypeQuery,
    graphb.OfName("mcFetchMentor"),
    graphb.OfField(
        "mentor",
        graphb.OfArguments(
            graphb.ArgumentString("uid", uid),
        ),
        graphb.OfFields(
            "uid", "paypal_email", "country", "languages", "bio", "educational_background", "intro_msg", "github_url",
            "linkedin_url", "avatar_url", "application", "blocked", "blocked_nds", "created_at", "updated_at",
        ),
    ),
)
jsonString, err := q.JsonBody()
```
jsonString evaluates to
```json
{"query":"query mcFetchMentor{mentor(uid:\"123\"){uid,paypal_email,country,languages,bio,educational_background,intro_msg,github_url,linkedin_url,avatar_url,application,blocked,blocked_nds,created_at,updated_at,},}"}
```
#### Query all courses from Classroom Content
```go
q := Query{
    OperationType: "query",
    OperationName: "test_graphb",
    Fields: []*Field{
        {
            Name:      "courses",
            Alias:     "Lets_Have_An_Alias",
            Arguments: nil,
            Fields:    Fields("id", "key"),
        },
    },
}
str, err := q.JsonBody()
fmt.Println(str)
```
prints
```json
{"query":"query test_graphb{Lets_Have_An_Alias:courses{id,key,},}"}
```
#### It's also easy to construct nested query
```go
q, err := NewQuery(
    "query",
    OfName("another_test"),
    OfField(
        "users",
        OfFields("id", "username"),
        OfField(
            "threads",
            OfFields("title", "created_at"),
        ),
    ),
)
s, err := q.JsonBody()
```
You get
```json
{"query":"query another_test{users{id,username,threads{title,created_at,},},}"}
```
#### Method Chaining
You can also see a method chaining API example in [example/example_test.go](example/example_test.go)

### Words from the author
As you can see, one can use `NewQuery`, `MakeQuery` or the struct literal to construct a query.

I recommend `NewQuery` because it provides more initialization time checking.

One may also find `MakeQuery`'s method chaining API more friendly to work with because it provides IDE code suggestions, which means you can literally build a GraphQL query through code suggestions.

Using a struct literal does not prevent you from constructing an invalid query. Of course, all errors will be caught once your construct the string.

__The library catches cycles.__ That is, if you have a `Field` whose sub Fields can reach the `Field` itself, the library reports an error.

I hesitate to make all fields private and only allow constructing a query through `NewQuery` and `MakeQuery`. I also don't know if `MakeQuery` is better than `NewQuery` or the contrary. __Please use it and give feedback__.

## Error Handling
All `graphb` errors are wrapped by [pkg/errors](https://github.com/pkg/errors).

## Test
`graphb` uses [testify/assert](https://github.com/stretchr/testify/#assert-package).
```bash
go test
```

## Todos
The library does not currently support:
1. Directive
2. Variables
3. Fragments

I do not know how useful would them be for a user of this library. Since the library builds the string for you, you sort of get the functionality of Variable and Fragment for free. You can just reuse a Field or the values of Fields and Arguments as normal Go code. Directive might be the most useful one for this library.
