# gosymphony
`gosymphony` is a library to allow devs to declare dependencies of tasks/functions, 
and `gosymphony` would run all tasks based on their dependencies with optimized concurrency.

This library significantly eases the flow-based/graph-based development, and achieves optimized concurrency automatically.

![](https://i.imgur.com/iGqac64.jpg)

## Install

```go
import "github.com/muqili/gosymphony"
```


## Quick Usage
![](https://i.imgur.com/PBrRWbE.png)



```go
package main

import (
    "context"
    "errors"
    "fmt"
    "github.com/muqili/gosymphony"
)

func main() {
    s := gosymphony.New()

    // Define Task f1, f2, f3
    // Task can be declare in any order
    s.Add("f1", nil, func(res map[string]*gosymphony.TaskState) (interface{}, error)){
        return "f1 result", nil
    }).Add("f2", nil, func(res map[string]*gosymphony.TaskState) (interface{}, error){
        return "f1 result", nil
    }).Add("f3", []string{"f1", "f2"}, func(res map[string]*gosymphony.TaskState) (interface{}, error) {
        return "f3 result", nil
    })
    // wait up to 1500ms
    res, err := s.Do(context.Background(), 1500)

    f2result,errf2 := res["f2"]
    f3result, errf3 := res["f3"]
 }
```

## Advanced Usage
![demo](https://i.imgur.com/F4ND6Ns.png)

Assuming the task dependency graph is the above one, 
the following code will resolve and run based on this


```go
package main

import (
    "context"
    "errors"
    "fmt"
    "github.com/muqili/gosymphony"
)

func main() {
    s := gosymphony.New()
    
    // Define Task f1, f2, f3, f4, f5, f6, f7
    // Task can be declare in any order
    s.Add("f2", []string{"f1", "f4"}, func(res map[string]*gosymphony.TaskState) (interface{}, error) {
        fmt.Println("==starting f2==")
        return "f2 result", nil
    }).Add("f3", []string{"f2", "f4"}, func(res map[string]*gosymphony.TaskState) (interface{}, error) {
        fmt.Println("==starting f3==")
        return fmt.Sprintf("%s|%s|%s", res["f2"].R, res["f4"].R, "f3 result"), nil
    }).Add("f4", []string{"f5"}, func(res map[string]*gosymphony.TaskState) (interface{}, error) {
        fmt.Println("==starting f4==")
        return "f4 result", nil
    }).Add("f5", nil, func(res map[string]*gosymphony.TaskState) (interface{}, error) {
        fmt.Println("==starting f5==")
        return "f5 result", nil
    }).Add("f6", []string{"f3"}, func(res map[string]*gosymphony.TaskState) (interface{}, error) {
        fmt.Println("==starting f6==")
        return "f6 result", nil
    }).Add("f7", []string{"f4"}, func(res map[string]*gosymphony.TaskState) (interface{}, error) {
        fmt.Println("==starting f7==")
        return "f7 result", nil
    }).Add("f1", nil, func(res map[string]*gosymphony.TaskState) (interface{}, error) {
        fmt.Println("==starting f1==")
        return "f1 result", nil
    })
    // wait up to 1500ms
    res, err := s.Do(context.Background(), 1500)
    

    f3r, _ := res["f3"]
    f6r, _ := res["f6"]
    fmt.Printf("err=%s", err) // err=nil
    fmt.Printf("f3=%s", f3r.R) // f3=f2 result|f4 result|f3 result
    fmt.Printf("f6=%s", f6r.R) // f6=f2 result|f4 result|f3 result|f6 result
 }
```
## Features
+ Automatic trigger dependent task
  + once all dependencies of a task are solved, the task would start running immediately
  
+ Dependency Safety check
  + Self Dependency
  + Non-existent dependency
  + Cyclic Dependency
  
+ Short-circuit
  + If any task returns error, will short-circuit all dependent tasks
  + In the above code, if `f3` returns error, f6 will not run (and return same error to all its dependencies if any)
  
 + Timeout
   + at `s.Do(context.Background(), 1500)`, `1500` defines the max running time in millisecond for, and will timeout by returning error
   if it exceeds this
   
Please see `symphony_test.go` for more use cases
