---
root: false
name: Data Binding
sort: 4
---

# Data Binding and Validation

Middlware binding provides request data binding and validation for Macaron [Instances](../intro/core_concepts#instances).

- [GitHub](https://github.com/macaron-contrib/binding)
- [API Reference](https://gowalker.org/github.com/macaron-contrib/binding)

## Installation

	go get github.com/macaron-contrib/binding

## Usage

### Getting form data from a request

Suppose you have a contact form on your site where at least name and message are required. We'll need a struct to receive the data:

```go
type ContactForm struct {
	Name           string `form:"name" binding:"Required"`
	Email          string `form:"email"`
	Message        string `form:"message" binding:"Required"`
	MailingAddress string `form:"mailing_address"`
}
```

Then we simply add our route in Macaron:

```go
m.Post("/contact/submit", binding.Bind(ContactForm{}), func(contact ContactForm) string {
    return fmt.Sprintf("Name: %s\nEmail: %s\nMessage: %s\nMailing Address: %v",
        contact.Name, contact.Email, contact.Message, contact.MailingAddress)
})
```

That's it! The [`binding.Bind`](https://gowalker.org/github.com/macaron-contrib/binding#Bind) function takes care of validating required fields.

By default, if there are any errors (like a required field is empty), binding middleware will return an error to the client and your app won't even see the request. To prevent this behavior, you can use [`binding.BindIgnErr`](https://gowalker.org/github.com/macaron-contrib/binding#BindIgnErr) instead.

**Caveat** Don't try to bind to embedded struct pointers; it won't work. See [martini-contrib/binding issue 30](https://github.com/martini-contrib/binding/issues/30) if you want to help with this.)

#### Naming Convention

By default, there is one naming convention for form tag name, which are:

- `Name` -> `name`
- `UnitPrice` -> `unit_price`

For example, previous example can be simplified with following code:

```go
type ContactForm struct {
	Name           string `binding:"Required"`
	Email          string
	Message        string `binding:"Required"`
	MailingAddress string
}
```

Clean and neat, isn't it?

If you want to custom your app naming convention, you can use [`binding.SetNameMapper`](https://gowalker.org/github.com/macaron-contrib/binding#SetNameMapper) function, which accepts a function that is type of [`binding.NameMapper`](https://gowalker.org/github.com/macaron-contrib/binding#NameMapper).

### Getting JSON data from a request

To get data from JSON payloads, simply use the `json:` struct tags instead of `form:`.

**Pro Tip** Use [JSON-to-Go](http://mholt.github.io/json-to-go/) to correctly convert JSON to a Go type definition. It's useful if you're new to this or the structure is large/complex.

### Binding to interfaces

If you'd like to bind the data to an interface rather than to a concrete struct, you can specify the interface and use it like this:

```go
m.Post("/contact/submit", binding.Bind(ContactForm{}, (*MyInterface)(nil)), func(contact MyInterface) {
	// ... your struct became an interface!
})
```

## Description of Handlers

Each of these middleware handlers are independent and optional, though be aware that some handlers invoke other ones.

### Bind

[`binding.Bind`](https://gowalker.org/github.com/macaron-contrib/binding#Bind) is a convenient wrapper over the other handlers in this package. It does the following boilerplate for you:

 1. Deserializes request data into a struct
 2. Performs validation with [`binding.Validate`](https://gowalker.org/github.com/macaron-contrib/binding#Validate)
 3. If your struct doesn't implement [`binding.ErrorHandler`](https://gowalker.org/github.com/macaron-contrib/binding#ErrorHandler), then default error handling will be applied. Otherwise, calls `ErrorHandler.Error` method to perform custom error handling.

Notes:

- Your application (the final handler) will not even see the request if there are any errors when default error handling is applied.
- Header `Content-Type` will be used to know how to deserialize the requests.

**Important Safety Tip** Don't attempt to bind a pointer to a struct. This will cause a panic [to prevent a race condition](https://github.com/codegangsta/martini-contrib/pull/34#issuecomment-29683659) where every request would be pointing to the same struct.

### Form

[`binding.Form`](https://gowalker.org/github.com/macaron-contrib/binding#Form) deserializes form data from the request, whether in the query string or as a `form-urlencoded` payload. It only does these things:

 1. Deserializes request data into a struct
 2. Performs validation with [`binding.Validate`](https://gowalker.org/github.com/macaron-contrib/binding#Validate)

Note that it does not handle errors. You may receive a [`binding.Errors`](https://gowalker.org/github.com/macaron-contrib/binding#Errors) into your own handler if you want to handle errors.

### MultipartForm and File Uploads

Like [`binding.Form`](https://gowalker.org/github.com/macaron-contrib/binding#Form), [`binding.MultipartForm`](https://gowalker.org/github.com/macaron-contrib/binding#MultipartForm) deserializes form data from a request into the struct you pass in. Additionally, this will deserialize a POST request that has a form of `enctype="multipart/form-data"`. If the bound struct contains a field of type [`*multipart.FileHeader`](http://gowalker.org/pkg/mime/multipart/#FileHeader) (or `[]*multipart.FileHeader`), you also can read any uploaded files that were part of the form.

This handler does the following:

 1. Deserializes request data into a struct
 2. Performs validation with [`binding.Validate`](https://gowalker.org/github.com/macaron-contrib/binding#Validate)

Again, like [`binding.Form`](https://gowalker.org/github.com/macaron-contrib/binding#Form), no error handling is performed, but you can get the errors in your handler by receiving a [`binding.Errors`](https://gowalker.org/github.com/macaron-contrib/binding#Errors) type.

#### Example

```go
type UploadForm struct {
	Title      string                `form:"title"`
	TextUpload *multipart.FileHeader `form:"txtUpload"`
}

func main() {
	m := macaron.Classic()
	m.Post("/", binding.MultipartForm(UploadForm{}), uploadHandler(uf UploadForm) string {
		file, err := uf.TextUpload.Open()
		// ... you can now read the uploaded file
	})
	m.Run()
}
```

### Json

[`binding.Json`](https://gowalker.org/github.com/macaron-contrib/binding#Json) deserializes JSON data in the payload of the request. It does the following things:

 1. Deserializes request data into a struct
 2. Performs validation with [`binding.Validate`](https://gowalker.org/github.com/macaron-contrib/binding#Validate)

Similar to [`binding.Form`](https://gowalker.org/github.com/macaron-contrib/binding#Form), no error handling is performed, but you can get the errors and handle them yourself.

### Validate

[`binding.Validate`](https://gowalker.org/github.com/macaron-contrib/binding#Validate) receives a populated struct and checks it for errors with basic rules. It will execute the `Validator.Validate()` method on the struct, if it is a [`binding.Validator`](https://gowalker.org/github.com/macaron-contrib/binding#Validator).

#### Validation Rules

There are some builtin validation rules. To use them, the tag format is `binding:"<Name>"`.

|Name|Note|
|----|----|
|`OmitEmpty`|Omit rest of validations if value is empty|
|`Required`|Must be non-zero value|
|`AlphaDash`|Must be alpha characters or numerics or `-_`|
|`AlphaDashDot`|Must be alpha characters or numerics, `-_` or `.`|
|`Size(int)`|Fixed length|
|`MinSize(int)`|Minimum length|
|`MaxSize(int)`|Maximum length|
|`Range(int,int)`|Value range(inclusive)|
|`Email`|Must be E-mail address|
|`Url`|Must be HTTP/HTTPS URL address|
|`In(a,b,c,...)`|Must be one of element in array|
|`NotIn(a,b,c,...)`|Must not be one of element in array|
|`Include(string)`|Must contain|
|`Exclude(string)`|Must not contain|
|`Default(string)`|Set default value when field is zero-value(cannot use this when bind with interface wrapper)|

To combine multiple rules: `binding:"Required;MinSize(10)"`.

## Customize Operations

### Custom Validation

If you want additional validation beyond just checking required fields, your struct can implement the [`binding.Validator`](https://gowalker.org/github.com/macaron-contrib/binding#Validator) interface like so:

```go
func (cf ContactForm) Validate(ctx *macaron.Context, errs binding.Errors) binding.Errors {
	if strings.Contains(cf.Message, "Go needs generics") {
		errs = append(errors, binding.Error{
			FieldNames:     []string{"message"},
			Classification: "ComplaintError",
			Message:        "Go has generics. They're called interfaces.",
		})
	}
	return errs
}
```

Now, any contact form submissions with "Go needs generics" in the message will return an error explaining your folly.

### Custom Validation Rules

If you need to more validation rules that are applied automatically for you, you can add custom rules by function [`binding.AddRule`](https://gowalker.org/github.com/macaron-contrib/binding#AddRule), it accepts type [`binding.Rule`](https://gowalker.org/github.com/macaron-contrib/binding#Rule) as argument.

Suppose you want to limit minimum value:

```go
binding.AddRule(&binding.Rule{
	IsMatch: func(rule string) bool {
		return strings.HasPrefix(rule, "Min(")
	},
	IsValid: func(errs Errors, name string, v interface{}) bool {
		num, ok := v.(int)
		if !ok {
			return false
		}
		min, _ := strconv.Atoi(rule[4 : len(rule)-1])
		if num < min {
			errs.Add([]string{name}, "MinimumValue", "Value is too small")
			return false
		}
		return true
	},
})
```

Custom validation rules are applied after builtin rules.

### Custom Error Handler

If you want to avoid default error handle process but still want binding middleware calls handle function for you, your struct can implement the [`binding.ErrorHandler`](https://gowalker.org/github.com/macaron-contrib/binding#ErrorHandler) interface like so:

```go
func (cf ContactForm) Error(ctx *macaron.Context, errs binding.Errors) {
	// Custom process to handle error.
}
```

This operation happens after your custom validation.
