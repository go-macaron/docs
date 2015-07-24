---
root: false
name: 数据绑定与验证
sort: 4
---

# 数据绑定与验证

中间件 binding 为 [Macaron 实例](../intro/core_concepts#macaron-%E5%AE%9E%E4%BE%8B) 提供了请求数据绑定与验证的功能。

- [GitHub](https://github.com/macaron-contrib/binding)
- [API 文档](https://gowalker.org/github.com/macaron-contrib/binding)

## 下载安装

	go get github.com/macaron-contrib/binding

## 使用示例

### 获取表单数据

假设您有一个联系人信息的表单，其中姓名和信息为必填字段，则我们可以使用如下结构来进行表示：

```go
type ContactForm struct {
	Name           string `form:"name" binding:"Required"`
	Email          string `form:"email"`
	Message        string `form:"message" binding:"Required"`
	MailingAddress string `form:"mailing_address"`
}
```

然后通过 Macaron 增加如下路由：

```go
m.Post("/contact/submit", binding.Bind(ContactForm{}), func(contact ContactForm) string {
    return fmt.Sprintf("Name: %s\nEmail: %s\nMessage: %s\nMailing Address: %v",
        contact.Name, contact.Email, contact.Message, contact.MailingAddress)
})
```

搞定！函数 [`binding.Bind`](https://gowalker.org/github.com/macaron-contrib/binding#Bind) 会帮助您完成对必选字段的数据验证。

默认情况下，如果在验证过程中发生任何错误（例如：必填字段的值为空），binding 中间件就会直接向客户端返回错误信息，提前终止请求的处理。如果您不希望 binding 中间件自动终止请求的处理，则可以使用 [`binding.BindIgnErr`](https://gowalker.org/github.com/macaron-contrib/binding#BindIgnErr) 函数来忽略对错误的自动处理。

**警告** 请不要使用类型为指针的嵌入结构，这会导致错误。请查看 [martini-contrib/binding issue 30](https://github.com/martini-contrib/binding/issues/30) 上的相关讨论获取完整信息。

#### 命名约定

默认情况下，`form` 标签的名称使用以下命名约定：

- `Name` -> `name`
- `UnitPrice` -> `unit_price`

也就是说，上面例子中的结构定义可以简化为如下代码：

```go
type ContactForm struct {
	Name           string `binding:"Required"`
	Email          string
	Message        string `binding:"Required"`
	MailingAddress string
}
```

超赞！有木有？

如果您想要自定义命名约定，可以通过 [`binding.SetNameMapper`](https://gowalker.org/github.com/macaron-contrib/binding#SetNameMapper) 函数来设置。该函数接受一个类型为 [`binding.NameMapper`](https://gowalker.org/github.com/macaron-contrib/binding#NameMapper) 的值作为参数。

### 获取 JSON 数据

将指定 `form` 标签的地方替换为 `json`，就可以完成对 JSON 数据的绑定。

**友情提示** 使用 [JSON-to-Go](http://mholt.github.io/json-to-go/) 网站工具可以帮助您更好更快地得根据 JSON 数据生成 Go 语言中对应的结构。

### 绑定到接口

如果您希望传递接口而不是一个具体的结构，则可以使用如下方法：

```go
m.Post("/contact/submit", binding.Bind(ContactForm{}, (*MyInterface)(nil)), func(contact MyInterface) {
	// ... 您接收到的值为一个接口
})
```

## 处理器说明

原则上，每个处理器之间是相互独立的，但在特定情况下，它们之间会相互调用。

### Bind

函数 [`binding.Bind`](https://gowalker.org/github.com/macaron-contrib/binding#Bind) 是一个便利性的高层封装，它能够自动识别表单类型并完成数据绑定与验证。

请求处理流程：

 1. 反序列化请求数据到结构
 2. 通过 [`binding.Validate`](https://gowalker.org/github.com/macaron-contrib/binding#Validate) 函数完成数据验证
 3. 如果您的结构实现了 [`binding.ErrorHandler`](https://gowalker.org/github.com/macaron-contrib/binding#ErrorHandler) 接口，则会调用相应的错误处理方法 `ErrorHandler.Error`；否则会使用默认的错误处理机制。

备注：

- 当使用默认的错误处理机制时，您的应用（队列后方的处理器）将根本不会意识到当前请求的存在。
- 头信息 `Content-Type` 是用于决定如何对请求数据进行反序列化的根本条件。

**重要安全提示** 请不要尝试绑定指向某个结构的指针，binding 中间件会直接 panic 并退出程序 [以防止可能发生的数据竞争](https://github.com/codegangsta/martini-contrib/pull/34#issuecomment-29683659)。

### Form

函数 [`binding.Form`](https://gowalker.org/github.com/macaron-contrib/binding#Form) 用于反序列化表单数据，可以是查询或 `form-urlencoded` 类型的请求。

请求处理流程：

 1. 反序列化请求数据到结构
 2. 通过 [`binding.Validate`](https://gowalker.org/github.com/macaron-contrib/binding#Validate) 函数完成数据验证

需要注意的是，该函数不具有默认错误处理机制。您可以通过获取类型为 [`binding.Errors`](https://gowalker.org/github.com/macaron-contrib/binding#Errors) 的参数来完成自定义错误处理。

### MultipartForm 和文件上传

类似 [`binding.Form`](https://gowalker.org/github.com/macaron-contrib/binding#Form)，函数 [`binding.MultipartForm`](https://gowalker.org/github.com/macaron-contrib/binding#MultipartForm) 同样是反序列化表单数据到结构。除此之外，它还能处理 `enctype="multipart/form-data"` 类型的 POST 请求。如果结构中包含类型为 [`*multipart.FileHeader`](http://gowalker.org/pkg/mime/multipart/#FileHeader)（或 `[]*multipart.FileHeader`）的字段，您可以直接从该字段读取客户端上传的文件。

请求处理流程：

 1. 反序列化请求数据到结构
 2. 通过 [`binding.Validate`](https://gowalker.org/github.com/macaron-contrib/binding#Validate) 函数完成数据验证

同样的，和函数 [`binding.Form`](https://gowalker.org/github.com/macaron-contrib/binding#Form) 一样，该函数不具有默认错误处理机制，但您可以通过获取类型为 [`binding.Errors`](https://gowalker.org/github.com/macaron-contrib/binding#Errors) 的参数来完成自定义错误处理。

#### 使用示例

```go
type UploadForm struct {
	Title      string                `form:"title"`
	TextUpload *multipart.FileHeader `form:"txtUpload"`
}

func main() {
	m := macaron.Classic()
	m.Post("/", binding.MultipartForm(UploadForm{}), uploadHandler(uf UploadForm) string {
		file, err := uf.TextUpload.Open()
		// ... 您可以在这里读取上传的文件内容
	})
	m.Run()
}
```

### Json

函数 [`binding.Json`](https://gowalker.org/github.com/macaron-contrib/binding#Json) 反序列化 JSON 数据。

请求处理流程：

 1. 反序列化请求数据到结构
 2. 通过 [`binding.Validate`](https://gowalker.org/github.com/macaron-contrib/binding#Validate) 函数完成数据验证

与函数 [`binding.Form`](https://gowalker.org/github.com/macaron-contrib/binding#Form)，该函数不具有默认错误处理机制，但您可以通过获取类型为 [`binding.Errors`](https://gowalker.org/github.com/macaron-contrib/binding#Errors) 的参数来完成自定义错误处理。


### Validate

函数 [`binding.Validate`](https://gowalker.org/github.com/macaron-contrib/binding#Validate) 接受一个结构并对它进行基本数据验证。如果该结构实现了 [`binding.Validator`](https://gowalker.org/github.com/macaron-contrib/binding#Validator) 接口，则会调用 `Validator.Validate()` 方法完成后续的数据验证。

#### 验证规则

目前有一些内置的验证规则，通过格式为 `binding:"<Name>"` 的标签使用。

|名称|说明|
|----|----|
|`OmitEmpty`|值为空时忽略后续验证|
|`Required`|必须为相同类型的非零值|
|`AlphaDash`|必须为半角英文字母、阿拉伯数字或 `-_`|
|`AlphaDashDot`|必须为半角英文字母、阿拉伯数字、`-_` 或 `.`|
|`Size(int)`|固定长度|
|`MinSize(int)`|最小长度|
|`MaxSize(int)`|最大长度|
|`Range(int,int)`|取值范围（包含边界值）|
|`Email`|必须为邮箱地址|
|`Url`|必须为 HTTP/HTTPS URL 地址|
|`In(a,b,c,...)`|必须为数组的一个元素|
|`NotIn(a,b,c,...)`|必须不是数组的元素|
|`Include(string)`|必须包含|
|`Exclude(string)`|必须不包含|
|`Default(string)`|当字段为零值时设置默认值（当使用接口绑定时不能设置该规则）|

当需要使用多条规则时： `binding:"Required;MinSize(10)"`。

## 自定义操作

### 自定义验证

如果您想要进行自定义的附加验证操作，您的结构可以通过实现接口 [`binding.Validator`](https://gowalker.org/github.com/macaron-contrib/binding#Validator) 来完成：

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

现在，任何包含信息 "Go needs generics" 的联系人表单都会报错。

### 自定义验证规则

当您觉得内置的验证规则不够时，可以通过函数 [`binding.AddRule`](https://gowalker.org/github.com/macaron-contrib/binding#AddRule) 来增加自定义验证规则。该函数接受一个类型为 [`binding.Rule`](https://gowalker.org/github.com/macaron-contrib/binding#Rule) 的参数。

假设您需要验证字段的最小值：

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

自定义规则的应用发生在内置规则之后。

### 自定义错误处理

如果您即不想使用默认的错误处理机制，又希望 binding 中间件自动化地调用您的自定义错误处理，则可以通过实现接口 [`binding.ErrorHandler`](https://gowalker.org/github.com/macaron-contrib/binding#ErrorHandler) 来完成：

```go
func (cf ContactForm) Error(ctx *macaron.Context, errs binding.Errors) {
	// 自定义错误处理过程
}
```

该操作发生在自定义验证规则被应用之后。
