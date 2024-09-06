**reflect**

1. 反射是用来检测存储在接口变量内部(值value；类型concrete type) pair对的一种机制。它提供了两种类型（或者说两个方法）让我们可以很容易的访问接口变量内容，分别是reflect.ValueOf() 和 reflect.TypeOf()


reflect.ValueOf() 获取反射值对象
reflect.TypeOf() 获取反射类型对象


```go
package main

import (
	"fmt"
	"reflect"
	"runtime"
)



type Demo struct {
	Id    int64 `json:"id"`
	Name  string
	Other struct {
		A float64
		B bool
		C interface{}
	}
}

func (d Demo) Print() {
	fmt.Println("Print")
}

func (d Demo) Print2(name string) string {
	fmt.Println("Print2 input param: ", name)
	return fmt.Sprintf("%s", name)
}

type Demo2 struct {
	Name string `json:"name" xml:"name" xorm:"not null comment('name') VARCHAR(255)"`
	age  int    `json:"age" xml:"age"`
}

func F(str string) string {
	return str
}

func main() {
	a := Demo{
		Id:   18,
		Name: "abc",
		Other: struct {
			A float64
			B bool
			C interface{}
		}{A: 1.2, B: false, C: func() {}},
	}

	f := a.Print2
	reflectDemo(f)

	reflectDemo2()

	return
}

func reflectDemo(i interface{}) {
	v := reflect.ValueOf(i)
	t := reflect.TypeOf(i)
	switch t.Kind() {
	case reflect.Struct:
		for i := 0; i < t.NumField(); i++ {
			fieldT := t.Field(i)
			fieldV := v.Field(i)
			if fieldV.Kind() == reflect.Struct {
				reflectDemo(fieldV.Interface())
			} else {
				fmt.Printf("%s: %v = %v\n", fieldT.Name, fieldT.Type, fieldV.Interface())
			}
		}
		methodNum := v.NumMethod()
		if methodNum > 0 {
			fmt.Println(t.Name(), "Struct method")
		}

		for i := 0; i < methodNum; i++ {
			method := t.Method(i)
			fmt.Printf("- %s(", method.Name)

			// 打印方法参数
			for j := 1; j < method.Type.NumIn(); j++ {
				paramType := method.Type.In(j)
				fmt.Printf("%s", paramType)
				if j < method.Type.NumIn()-1 {
					fmt.Printf(", ")
				}
			}

			// 打印方法返回值
			if method.Type.NumOut() > 0 {
				fmt.Printf(") (")
				for j := 0; j < method.Type.NumOut(); j++ {
					returnType := method.Type.Out(j)
					fmt.Printf("%s", returnType)
					if j < method.Type.NumOut()-1 {
						fmt.Printf(", ")
					}
				}
				fmt.Printf(")\n")
			} else {
				fmt.Printf(")\n")
			}
		}

		// 调用结构体方法
		if methodNum > 0 {
			fmt.Println("Print2 method exec...")
			f := v.MethodByName("Print2")
			retVals := f.Call([]reflect.Value{reflect.ValueOf("aha")})
			result := make([]interface{}, 0)
			for _, val := range retVals {
				result = append(result, val.Interface())
			}

			fmt.Println("method result:", result)

		}
	case reflect.Slice, reflect.Array, reflect.String:
		fmt.Printf("%c", '[')
		for i := 0; i < v.Len(); i++ {
			elem := v.Index(i)
			fmt.Printf("%v ", elem.Interface())
		}
		fmt.Printf("%c\n", ']')
	case reflect.Map:
		for _, k := range v.MapKeys() {
			field := v.MapIndex(k)
			fmt.Printf("%v => %v\n", k.Interface(), field.Interface())
		}
	case reflect.Func:
		// 获取函数的程序计数器地址
		functionPC := v.Pointer()
		// 使用runtime.FuncForPC获取函数对象
		function := runtime.FuncForPC(functionPC)
		fmt.Println("func_name:", function.Name())

		inParam := make([]string, t.NumIn())
		for i := 0; i < t.NumIn(); i++ {
			inParam[i] = t.In(i).Name()
		}
		fmt.Println("input:", inParam)

		outValue := make([]string, t.NumOut())
		for i := 0; i < t.NumOut(); i++ {
			outValue[i] = t.Out(i).Name()
		}
		fmt.Println("output:", outValue)
	default:
		fmt.Println("other type:", t.Kind())
	}
}

func reflectDemo2() {
	// 修改值
	u := &Demo2{"dj", 18}
	uv := reflect.ValueOf(u).Elem()
	name := uv.Field(0)
	fmt.Println(name.CanAddr(), name.CanSet())
	age := uv.Field(1)
	fmt.Println(age.CanAddr(), age.CanSet())

	name.SetString("jd")
	fmt.Println(u)

	// 获取标签
	u2 := Demo2{"dj2", 182}
	t := reflect.TypeOf(u2)
	fmt.Println("name tag: ", t.Field(0).Tag)
	fmt.Println("name json tag:", t.Field(0).Tag.Get("json"))
}

```

