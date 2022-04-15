# Go-Web

## http包

### 创建一个http服务示例



```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"strings"
)

func sayhelloName(w http.ResponseWriter, r *http.Request) {
	err := r.ParseForm()
	if err != nil {
		return
	} //解析参数，默认是不会解析的
	fmt.Println(r.Form) //这些信息是输出到服务器端的打印信息
	fmt.Println("path", r.URL.Path)
	fmt.Println("scheme", r.URL.Scheme)
	fmt.Println(r.Form["url_long"])
	for k, v := range r.Form {
		fmt.Println("key:", k)
		fmt.Println("val:", strings.Join(v, ""))
	}
	_, err = fmt.Fprintf(w, "Hello astaxie!")
	if err != nil {
		return
	} //这个写入到w的是输出到客户端的
}

func main() {
	http.HandleFunc("/abc", sayhelloName)    //设置访问的路由
	err := http.ListenAndServe(":9090", nil) //设置监听的端口
	if err != nil {
		log.Fatal("ListenAndServe: ", err)
	}
}

```

### 自定义路由器

`ListenAndServe`的第二个参数就是用以配置外部路由器的，它是一个Handler接口，即外部路由器只要实现了Handler接口就可以

```go
package main

import (
	"fmt"
	"net/http"
)

type MyMux struct {
}

func (p *MyMux) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	if r.URL.Path == "/" {
		sayhelloName(w, r)
		return
	}
	http.NotFound(w, r)
	return
}

func sayhelloName(w http.ResponseWriter, r *http.Request) {
	fmt.Fprintf(w, "Hello myroute!")
}

func main() {
	mux := &MyMux{}
	http.ListenAndServe(":9090", mux)
}
```

### 多重路由http.NewServeMux

```go
package main

import (
	"fmt"
	"math/rand"
	"net/http"
)

func main() {
	newMux := http.NewServeMux()

	newMux.HandleFunc("/randomFloat", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintln(w, rand.Float64())
	})

	newMux.HandleFunc("/randomInt", func(w http.ResponseWriter, r *http.Request) {
		fmt.Fprintln(w, rand.Intn(100))
	})
	http.ListenAndServe(":8000", newMux)
}

```

### 多重路由：使用httprouter

```go
package main

import (
	"io"
	"log"
	"net/http"
	"os/exec"

	"github.com/julienschmidt/httprouter"
)

func getCommandOutput(command string, arguments ...string) string {
	//执行命令行输入输出
	out, _ := exec.Command(command, arguments...).Output()
	return string(out)
}

func goVersion(w http.ResponseWriter, r *http.Request, params httprouter.Params) {
    //go的路径
	response := getCommandOutput("E:\\GoLand\\SDK\\go1.18rc1\\bin\\go.exe", "version")
	io.WriteString(w, response)
	return
}

//func getFileContent(w http.ResponseWriter, r *http.Request, params httprouter.Params) {
//	fmt.Fprintf(w, getCommandOutput("/bin/cat", params.ByName("name")))
//}

func main() {
	router := httprouter.New()
	router.GET("/api/v1/go-version", goVersion)
	//router.GET("/api/v1/show-file/:name", getFileContent)
	log.Fatal(http.ListenAndServe(":8000", router))
}

```

#### 文件功能ServeFiles

```go
package main

import (
	"log"
	"net/http"

	"github.com/julienschmidt/httprouter"
)

func main() {
	router := httprouter.New()
	// Mapping to methods is possible with HttpRouter
	router.ServeFiles("/static/*filepath", http.Dir("E:\\All Files\\Programmaing\\Go\\Hands-On-Restful-Web-services-with-Go-master\\chapter2\\httprouterExample"))
	log.Fatal(http.ListenAndServe(":8000", router))
}

```



### 多重路由：gorilla/mux

#### 路径匹配

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"time"

	"github.com/gorilla/mux"
)

func ArticleHandler(w http.ResponseWriter, r *http.Request) {
	vars := mux.Vars(r)
	w.WriteHeader(http.StatusOK)
	fmt.Fprintf(w, "Category is: %v\n", vars["category"])
	fmt.Fprintf(w, "ID is: %v\n", vars["id"])
}

func main() {
	r := mux.NewRouter()
	r.HandleFunc("/articles/{category}/{id:[0-9]+}", ArticleHandler)
	srv := &http.Server{
		Handler:      r,
		Addr:         "127.0.0.1:8000",
		WriteTimeout: 15 * time.Second,
		ReadTimeout:  15 * time.Second,
	}
	log.Fatal(srv.ListenAndServe())
}

```



#### 查询匹配

```go
package main

import (
	"fmt"
	"log"
	"net/http"
	"time"

	"github.com/gorilla/mux"
)

// QueryHandler handles the given query parameters
func QueryHandler(w http.ResponseWriter, r *http.Request) {
	queryParams := r.URL.Query()
	w.WriteHeader(http.StatusOK)
	fmt.Fprintf(w, "Got parameter id:%s!\n", queryParams["id"][0])
	fmt.Fprintf(w, "Got parameter category:%s!", queryParams["category"][0])
}

func main() {
	r := mux.NewRouter()
	r.HandleFunc("/articles", QueryHandler)
    //存入对应的query键和值
	r.Queries("id", "category")
	srv := &http.Server{
		Handler:      r,
		Addr:         "127.0.0.1:8000",
		WriteTimeout: 15 * time.Second,
		ReadTimeout:  15 * time.Second,
	}
	log.Fatal(srv.ListenAndServe())
}
```

#### 反向匹配（Reverse mapping）

> 只需要对handlefunc命名即可，然后根据名字传入参数即可

```go
r.HandlerFunc("/articles/{category}/{id:[0-9]+}", ArticleHandler).Name("articleRoute")

url, err := r.Get("articleRoute").URL("category", "books", "id", "123")
fmt.Printf(url.Path) // prints /articles/books/123
```

#### 路径前缀

一个文件服务器前缀`static`

```go
r.PathPrefix("/static/").Handler(http.StripPrefix("/static/",http.FileServer(http.Dir("/tmp/static"))))
```



#### 匹配编码后的路径参数

可以同时匹配http://localhost:8000/books/2和http://localhost:8000/books%2F2

```go
r.UseEncodedPath()
r.NewRoute().Path("/category/id")
```



## 中间件

中间件在请求被处理之前和处理后被返回之前使用

![image-20220405161348698](C:\Users\Administrator\Desktop\WEB\笔记\Golang\img\image-20220405161348698.png)

### 纯http自定义单个中间件

```go
package main

import (
	"fmt"
	"net/http"
)

func middleware(handler http.Handler) http.Handler {
	//返回一个处理函数
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		fmt.Println("Executing middleware before request phase!")
		// Pass control back to the handler
		handler.ServeHTTP(w, r)
		fmt.Println("Executing middleware after response phase!")
	})
}

func handle(w http.ResponseWriter, r *http.Request) {
	// Business logic goes here
	fmt.Println("Executing mainHandler...")
	w.Write([]byte("OK"))
}

func main() {
	// HandlerFunc returns a HTTP Handler
	originalHandler := http.HandlerFunc(handle)
	//使用上面定义的middleware函数包装
	//和之前handlefunc的区别是处理函数必须是http.HandlerFunc（这两个多了一个r）
	http.Handle("/", middleware(originalHandler))
	http.ListenAndServe(":8000", nil)
}

```

### 多层（嵌套）中间件

```go
package main

import (
	"encoding/json"
	"log"
	"net/http"
	"strconv"
	"time"
)

type city struct {
	Name string
	Area uint64
}

func filterContentType(handler http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		log.Println("Currently in the check content type middleware")
		// Filtering requests by MIME type
		if r.Header.Get("Content-type") != "application/json" {
			w.WriteHeader(http.StatusUnsupportedMediaType)
			w.Write([]byte("415 - Unsupported Media Type. Please send JSON"))
			return
		}
		handler.ServeHTTP(w, r)
	})
}

func setServerTimeCookie(handler http.Handler) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
		// Setting cookie to every API response
		cookie := http.Cookie{Name: "ServerTimeUTC", Value: strconv.FormatInt(time.Now().Unix(), 10)}
		http.SetCookie(w, &cookie)
		log.Println("Currently in the set server time middleware")
		handler.ServeHTTP(w, r)
	})
}

func handle(w http.ResponseWriter, r *http.Request) {
	// Check if method is POST
	if r.Method == "POST" {
		var tempCity city
		decoder := json.NewDecoder(r.Body)
		err := decoder.Decode(&tempCity)
		if err != nil {
			panic(err)
		}
		defer r.Body.Close()
		// Your resource creation logic goes here. For now it is plain print to console
		log.Printf("Got %s city with area of %d sq miles!\n", tempCity.Name, tempCity.Area)
		// Tell everything is fine
		w.WriteHeader(http.StatusOK)
		w.Write([]byte("201 - Created"))
	} else {
		// Say method not allowed
		w.WriteHeader(http.StatusMethodNotAllowed)
		w.Write([]byte("405 - Method Not Allowed"))
	}
}

func main() {
	originalHandler := http.HandlerFunc(handle)
	http.Handle("/city", filterContentType(setServerTimeCookie(originalHandler)))
	http.ListenAndServe(":8000", nil)
}

```



### 使用alice进行嵌套

`github.com/justinas/alice`

```go
func main() {
	originalHandler := http.HandlerFunc(handle)
	chain := alice.New(filterContentType, setServerTimeCookie).Then(originalHandler)
	http.Handle("/city", chain)
	http.ListenAndServe(":8000", nil)
}
```

### 使用Gorilla的日志中间件

`github.com/gorilla/handlers`

```go
package main

import (
	"log"
	"net/http"
	"os"

	"github.com/gorilla/handlers"
	"github.com/gorilla/mux"
)

func handle(w http.ResponseWriter, r *http.Request) {
	log.Println("Processing request!")
	w.Write([]byte("OK"))
	log.Println("Finished processing request")
}

func main() {
	r := mux.NewRouter()
	r.HandleFunc("/", handle)
    //第一个参数是输出位置，第二个是router，最后传入loggedRouter
	loggedRouter := handlers.LoggingHandler(os.Stdout, r)
	http.ListenAndServe(":8000", loggedRouter)
}

```

## RPC

概念：RPC是远程过程调用（Remote Procedure Call）的缩写形式，他能使独立运行的程序互相调用。

### 基础使用

> 服务端

```go
package main

import (
	"log"
	"net"
	"net/http"
	"net/rpc"
	"time"
)

type Args struct{}

type TimeServer int64

func (t *TimeServer) GiveServerTime(args *Args, reply *int64) error {
	// Fill reply pointer to send the data back
	*reply = time.Now().Unix()
	return nil
}

func main() {
	timeserver := new(TimeServer)
	// 将timeserver实例的方法公开给server
	rpc.Register(timeserver)
	// 将timeserver实例的方法注册到server上
	rpc.HandleHTTP()
	// Listen for requests on port 1234
	l, e := net.Listen("tcp", ":1234")
	if e != nil {
		log.Fatal("listen error:", e)
	}
	http.Serve(l, nil)
}

```



> 客户端

```go
package main

import (
	"log"
	"net/rpc"
)

type Args struct {
}

func main() {
	var reply int64
	args := Args{}
	client, err := rpc.DialHTTP("tcp", "localhost"+":1234")
	if err != nil {
		log.Fatal("dialing:", err)
	}
	err = client.Call("TimeServer.GiveServerTime", args, &reply)
	if err != nil {
		log.Fatal("arith error:", err)
	}
	log.Printf("%d", reply)
}
```



### jsonRPC

```go
package main

import (
	jsonparse "encoding/json"
	"io/ioutil"
	"log"
	"net/http"
	"os"

	"path/filepath"

	"github.com/gorilla/mux"
	"github.com/gorilla/rpc"
	"github.com/gorilla/rpc/json"
)

// Args holds arguments passed to JSON RPC service
type Args struct {
	ID string
}

// Book struct holds Book JSON structure
type Book struct {
	ID     string `json:"id,omitempty"`
	Name   string `json:"name,omitempty"`
	Author string `json:"author,omitempty"`
}

type JSONServer struct{}

// GiveBookDetail is RPC implementation
func (t *JSONServer) GiveBookDetail(r *http.Request, args *Args, reply *Book) error {
	var books []Book
	// Read JSON file and load data
	absPath, _ := filepath.Abs("./books.json")
	raw, readerr := ioutil.ReadFile(absPath)
	if readerr != nil {
		log.Println("error:", readerr)
		os.Exit(1)
	}
	// 将json字符串转为books对象数组
	marshalerr := jsonparse.Unmarshal(raw, &books)
	if marshalerr != nil {
		log.Println("error:", marshalerr)
		os.Exit(1)
	}
	// Iterate over each book to find the given book
	for _, book := range books {
		if book.ID == args.ID {
			// If book found, fill reply with it
			*reply = book
			break
		}
	}
	return nil
}

func main() {
	// Create a new RPC server
	s := rpc.NewServer()
	// Register the type of data requested as JSON
	s.RegisterCodec(json.NewCodec(), "application/json")
	// Register the service by creating a new JSON server
	s.RegisterService(new(JSONServer), "")
	r := mux.NewRouter()
	r.Handle("/rpc", s)
	http.ListenAndServe(":1234", r)
}

```



## 框架

### restful

`go get github.com/emicklei/go-restful`

#### 基础使用

```go
package main

import (
	"fmt"
	"github.com/emicklei/go-restful"
	"io"
	"net/http"
	"time"
)

func main() {
	// Create a web service
	webservice := new(restful.WebService)
	// Create a route and attach it to handler in the service
	webservice.Route(webservice.GET("/ping").To(pingTime))
	// Add the service to application
	restful.Add(webservice)
	http.ListenAndServe(":8000", nil)
}

func pingTime(req *restful.Request, resp *restful.Response) {
	//将第二个参数写入第一个参数
	io.WriteString(resp, fmt.Sprintf("%s", time.Now()))
}

```

#### 建立API

```go
package main

import (
	"database/sql"
	"encoding/json"
	"log"
	"net/http"
	"time"

	"YourProjectName/Hands-On-Restful-Web-services-with-Go-master/chapter4/dbutils"
	"github.com/emicklei/go-restful"
	_ "github.com/mattn/go-sqlite3"
)

// DB Driver visible to whole program
var DB *sql.DB

// TrainResource is the model for holding rail information
type TrainResource struct {
	ID              int
	DriverName      string
	OperatingStatus bool
}

// StationResource holds information about locations
type StationResource struct {
	ID          int
	Name        string
	OpeningTime time.Time
	ClosingTime time.Time
}

// ScheduleResource links both trains and stations
type ScheduleResource struct {
	ID          int
	TrainID     int
	StationID   int
	ArrivalTime time.Time
}

// Register adds paths and routes to container
func (t *TrainResource) Register(container *restful.Container) {
	ws := new(restful.WebService)
	//指定接收和发送的数据类型均为JSON
	ws.Path("/v1/trains").Consumes(restful.MIME_JSON).Produces(restful.MIME_JSON) // you can specify this per route as well

	ws.Route(ws.GET("/{train-id}").To(t.getTrain))
	ws.Route(ws.POST("").To(t.createTrain))
	ws.Route(ws.DELETE("/{train-id}").To(t.removeTrain))

	container.Add(ws)
}

// GET http://localhost:8000/v1/trains/1
func (t TrainResource) getTrain(request *restful.Request, response *restful.Response) {
	id := request.PathParameter("train-id")
	err := DB.QueryRow("select ID, DRIVER_NAME, OPERATING_STATUS FROM train where id=?", id).Scan(&t.ID, &t.DriverName, &t.OperatingStatus)
	if err != nil {
		log.Println(err)
		response.AddHeader("Content-Type", "text/plain")
		response.WriteErrorString(http.StatusNotFound, "Train could not be found.")
	} else {
		//将一个类结构以指定的输出方式（router里，这里是JSON）写入response
		//这个是直接输出200
		response.WriteEntity(t)
	}
}

// POST http://localhost:8000/v1/trains
func (t TrainResource) createTrain(request *restful.Request, response *restful.Response) {
	log.Println(request.Request.Body)
	//解码请求获取参数放入结构体
	decoder := json.NewDecoder(request.Request.Body)
	var b TrainResource
	err := decoder.Decode(&b)
	log.Println(b.DriverName, b.OperatingStatus)

	// Error handling is obvious here. So omitting...
	statement, _ := DB.Prepare("insert into train (DRIVER_NAME, OPERATING_STATUS) values (?, ?)")
	result, err := statement.Exec(b.DriverName, b.OperatingStatus)
	if err == nil {
		newID, _ := result.LastInsertId()
		b.ID = int(newID)
		//这个是按照第一个参数输出，并返回值
		response.WriteHeaderAndEntity(http.StatusCreated, b)
	} else {
		response.AddHeader("Content-Type", "text/plain")
		response.WriteErrorString(http.StatusInternalServerError, err.Error())
	}
}

// DELETE http://localhost:8000/v1/trains/1
func (t TrainResource) removeTrain(request *restful.Request, response *restful.Response) {
	id := request.PathParameter("train-id")
	statement, _ := DB.Prepare("delete from train where id=?")
	_, err := statement.Exec(id)
	if err == nil {
		response.WriteHeader(http.StatusOK)
	} else {
		response.AddHeader("Content-Type", "text/plain")
		response.WriteErrorString(http.StatusInternalServerError, err.Error())
	}
}

func main() {
	var err error
	DB, err = sql.Open("sqlite3", "./railapi.db")
	if err != nil {
		log.Println("Driver creation failed!")
	}
	dbutils.Initialize(DB)
	wsContainer := restful.NewContainer()
	//restful.CurlyRouter{}开启使用{id}作为路径参数的功能
	wsContainer.Router(restful.CurlyRouter{})
	t := TrainResource{}
	t.Register(wsContainer)

	log.Printf("start listening on localhost:8000")
	server := &http.Server{Addr: ":8000", Handler: wsContainer}
	log.Fatal(server.ListenAndServe())
}

```

### Gin

安装：`github.com/gin-gonic/gin`

生产模式：将环境变量设为`GIN_MODE=release`

#### 基本使用

```go
package main

import (
	"time"

	"github.com/gin-gonic/gin"
)

func main() {
	//返回一个gin引擎实例
	r := gin.Default()
	//参数为路径和处理函数handler，handler接收上下文参数
	r.GET("/pingTime", func(c *gin.Context) {
		// JSON将里面的内容序列化为json对象并包裹在response里
		c.JSON(200, gin.H{
			"serverTime": time.Now().UTC(),
		})
	})
	// 可以不填，默认为8080端口
	r.Run(":8000") // Listen and serve on 0.0.0.0:8080
}

```

#### 建立API

```go
package main

import (
	"database/sql"
	"log"
	"net/http"

	"YourProjectName/Hands-On-Restful-Web-services-with-Go-master/chapter4/dbutils"
	"github.com/gin-gonic/gin"
	_ "github.com/mattn/go-sqlite3"
)

// DB Driver visible to whole program
var DB *sql.DB

// StationResource holds information about locations
type StationResource struct {
	ID          int    `json:"id"`
	Name        string `json:"name"`
	OpeningTime string `json:"opening_time"`
	ClosingTime string `json:"closing_time"`
}

// GetStation returns the station detail
func GetStation(c *gin.Context) {
	var station StationResource
	//获取get请求的参数
	id := c.Param("station_id")
	//CAST(CLOSING_TIME as CHAR)将时间转为字符串
	err := DB.QueryRow("select ID, NAME, CAST(OPENING_TIME as CHAR), CAST(CLOSING_TIME as CHAR) from station where id=?", id).Scan(&station.ID, &station.Name, &station.OpeningTime, &station.ClosingTime)
	if err != nil {
		log.Println(err)
		c.JSON(500, gin.H{
			"error": err.Error(),
		})
	} else {
		c.JSON(200, gin.H{
			"result": station,
		})
	}
}

// CreateStation handles the POST
func CreateStation(c *gin.Context) {
	var station StationResource
	// 将请求体的数据解析为station
	if err := c.BindJSON(&station); err == nil {
		// Format Time to Go time format
		statement, _ := DB.Prepare("insert into station (NAME, OPENING_TIME, CLOSING_TIME) values (?, ?, ?)")
		result, _ := statement.Exec(station.Name, station.OpeningTime, station.ClosingTime)
		if err == nil {
			newID, _ := result.LastInsertId()
			station.ID = int(newID)
			c.JSON(http.StatusOK, gin.H{
				"result": station,
			})
		} else {
			c.String(http.StatusInternalServerError, err.Error())
		}
	} else {
		c.String(http.StatusInternalServerError, err.Error())
	}
}

// RemoveStation handles the removing of resource
func RemoveStation(c *gin.Context) {
	id := c.Param("station-id")
	statement, _ := DB.Prepare("delete from station where id=?")
	_, err := statement.Exec(id)
	if err != nil {
		log.Println(err)
		c.JSON(500, gin.H{
			"error": err.Error(),
		})
	} else {
		c.String(http.StatusOK, "")
	}
}

func main() {
	var err error
	DB, err = sql.Open("sqlite3", "./railapi.db")
	if err != nil {
		log.Println("Driver creation failed!")
	}
	dbutils.Initialize(DB)
	r := gin.Default()
	// Add routes to REST verbs
	r.GET("/v1/stations/:station_id", GetStation)
	r.POST("/v1/stations", CreateStation)
	r.DELETE("/v1/stations/:station_id", RemoveStation)

	r.Run(":8000") // Default listen and serve on 0.0.0.0:8080
}

```

#### 使用中间件

```go
authorized := router.Group("/")
authorized.Use(authHandler.AuthMiddleware())
{
    authorized.POST("/recipes", recipesHandler.NewRecipeHandler)
    authorized.PUT("/recipes/:id", recipesHandler.UpdateRecipeHandler)
    authorized.DELETE("/recipes/:id", recipesHandler.DeleteRecipeHandler)
    authorized.GET("/recipes/:id", recipesHandler.GetOneRecipeHandler)
}

// gin中不需要传入handler
func AuthMiddleware() gin.HandlerFunc {
	return func(c *gin.Context) {
		if c.GetHeader("X-API-KEY") != os.Getenv("X_API_KEY") {
			c.AbortWithStatus(401)
		}
        // 中间件任务已完成，继续handler
		c.Next()
	}
}
```



#### API笔记

1. `gin.H`实际上是一个创建字典变量的快捷方法
2. `gin.Context`在处理的时候仅传入指针，然后再逐层传回并使用或修改里面的值
3. `c.JSON(http.StatusOK, recipes)`将结构体的内容转为json格式并放入response body中
   1. `c.BindJSON(&recipe)`将context中的信息转为结构体中的内容，`MustBind`的便捷形式
4. `Shouldxxx`和`bindxxx`区别就是bindxxx会在head中添加400的返回信息，而Shouldxxx不会

## Gorm

下载：`gorm.io/gorm`

驱动：`gorm.io/driver/sqlite`

**注意**：字段名必须是大写的才是可导出的！！！小写字段是访问不到的！！！

```go
package main

import (
	"fmt"
	"gorm.io/driver/sqlite"
	"gorm.io/gorm"
)

type Product struct {
	Price int
	Name  string
}

type User struct {
	gorm.Model //加入此行才能正确迁移
	Name       string
	Age        int
}

func initdb() {
	//开启数据库
	db, _ := gorm.Open(sqlite.Open("./gorm.db"), &gorm.Config{})
	user := &User{Name: "bar", Age: 10}
	// 自动迁移
	err := db.AutoMigrate(&User{})
	if err != nil {
		panic(err)
	}
	//创建数据
	db.Create(&user)
	//更新数据
	db.Model(&user).Update("age", 30)
    
    // 也可通过save更新
    db.First(&user)
    user.Name = "jinzhu 2"
    user.Age = 100
    db.Save(&user)
    
	//忽略某个参数的更新（不更新他），并更新多个参数
	db.Model(&user).Omit("age").Updates(map[string]interface{}{"Name": "bar2", "age": 30})

}
func querydb() {
	//开启数据库
	db, _ := gorm.Open(sqlite.Open("./gorm.db"), &gorm.Config{})
	// 自动迁移
	err := db.AutoMigrate(&User{})
	if err != nil {
		panic(err)
	}
	//查询id为2的user，First方法返回找到的第一个值，Last方法返回最后一个
	user2 := &User{}
	db.First(&user2, 2)
	//查询name为bar2的user
	user3 := User{}
	db.First(&user3, "name=?", "bar2")
	//以id查询一个数组
	var users []User
	db.Find(&users, []int{1, 2, 4})
	//fmt.Print(users)
	//查询所有数据
	var users2 []User
	db.Find(&users2)
	//条件查询
	var users3 []User
	updatetime := "2022-04-05 21:57:57.8203303+08:00"
	updatetime2 := "2022-04-05 21:59:57.8203303+08:00"
	db.Where("updated_at > ?", updatetime).Find(&users3)
	fmt.Print(users3)
	
	//BETWEEN
	db.Where("created_at BETWEEN ? AND ?", updatetime, updatetime2).Find(&users)
}
func main() {
	//initdb()
	querydb()
}
```



## API授权

### JWT

1. 安装：`go get github.com/dgrijalva/jwt-go`

2. 验证账户密码并返回JWT签名：

   ```go
   package handlers
   
   import (
   	"net/http"
   	"os"
   	"time"
   	"github.com/dgrijalva/jwt-go"
   	"github.com/gin-gonic/gin"
   	"github.com/mlabouardy/recipes-api/models"
   )
   
   type AuthHandler struct{}
   
   // Claims 生成签名的请求体
   type Claims struct {
   	Username string `json:"username"`
   	jwt.StandardClaims
   }
   
   // JWTOutput 签名的结构体
   type JWTOutput struct {
   	Token   string    `json:"token"`
   	Expires time.Time `json:"expires"`
   }
   
   func (handler *AuthHandler) ListRecipesHandler(c *gin.Context) {
   	var user models.User
   	if err := c.ShouldBindJSON(&user); err != nil {
   		c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
   		return
   	}
   	
   	// 若用户名密码错误则返回
   	if user.Username != "admin" || user.Password != "password" {
   		c.JSON(http.StatusUnauthorized, gin.H{"error": "Invalid username or password"})
   		return
   	}
   
   	// 签名请求
   	expirationTime := time.Now().Add(10 * time.Minute)
   	claims := &Claims{
   		Username: user.Username,
   		StandardClaims: jwt.StandardClaims{
   			ExpiresAt: expirationTime.Unix(),
   		},
   	}
   	
   	// 生成签名
   	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
   	tokenString, err := token.SignedString(os.Getenv("JWT_SECRET"))
   	if err != nil {
   		c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
   		return
   	}
   
   	jwtOutput := JWTOutput{
   		Token:   tokenString,
   		Expires: expirationTime,
   	}
   	c.JSON(http.StatusOK, jwtOutput)
   }
   ```

3. 检查是否授权的中间件：

   ```GO
   func (handler *AuthHandler) AuthMiddleware() gin.HandlerFunc {
   	return func(c *gin.Context) {
   		tokenValue := c.GetHeader("Authorization")
   		claims := &Claims{}
   		tkn, err := jwt.ParseWithClaims(tokenValue, claims, func(token *jwt.Token) (interface{}, error) {
   			// JWT密码，自行设置
   			return []byte(os.Getenv("JWT_SECRET")), nil
   		})
   		if err != nil {
   			c.AbortWithStatus(http.StatusUnauthorized)
   		}
   		if !tkn.Valid {
   			c.AbortWithStatus(http.StatusUnauthorized)
   		}
           // 中间件的任务已经完成，继续handler的任务
   		c.Next()
   	}
   }
   ```

4. 刷新签名：

   ```go
   func (handler *AuthHandler) RefreshHandler(c *gin.Context) {
       // 解析之前的签名并校验
   	tokenValue := c.GetHeader("Authorization")
   	claims := &Claims{}
   	tkn, err := jwt.ParseWithClaims(tokenValue, claims, func(token *jwt.Token) (interface{}, error) {
   		return []byte(os.Getenv("JWT_SECRET")), nil
   	})
   	if err != nil {
   		c.JSON(http.StatusUnauthorized, gin.H{"error": err.Error()})
   		return
   	}
   	if !tkn.Valid {
   		c.JSON(http.StatusUnauthorized, gin.H{"error": "Invalid token"})
   		return
   	}
   	// 生成签名
   	if time.Unix(claims.ExpiresAt, 0).Sub(time.Now()) > 30*time.Second {
   		c.JSON(http.StatusBadRequest, gin.H{"error": "Token is not expired yet"})
   		return
   	}
   
   	expirationTime := time.Now().Add(5 * time.Minute)
   	claims.ExpiresAt = expirationTime.Unix()
   	token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
   	tokenString, err := token.SignedString(os.Getenv("JWT_SECRET"))
   	if err != nil {
   		c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
   		return
   	}
   
   	jwtOutput := JWTOutput{
   		Token:   tokenString,
   		Expires: expirationTime,
   	}
   	c.JSON(http.StatusOK, jwtOutput)
   }
   ```

5. 检查授权的中间件（以session存储）：

   `go get github.com/gin-contrib/sessions`

   使用`redis`存储session：

   ```go
   store, _ := redisStore.NewStore(10, "tcp", "localhost:6379", "", []byte("secret"))
   router.Use(sessions.Sessions("recipes_api", store))
   ```

   ```go
   // 检查授权
   func (handler *AuthHandler) AuthMiddleware() gin.HandlerFunc {
   	return func(c *gin.Context) {
   		session := sessions.Default(c)
   		sessionToken := session.Get("token")
   		if sessionToken == nil {
   			c.JSON(http.StatusForbidden, gin.H{
   				"message": "Not logged",
   			})
   			c.Abort()
   		}
   		c.Next()
   	}
   }
   ```

6. 登录登出中间件（Session）：

   ```go
   // 登录
   func (handler *AuthHandler) SignInHandler(c *gin.Context) {
   	var user models.User
   	if err := c.ShouldBindJSON(&user); err != nil {
   		c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
   		return
   	}
   
   	h := sha256.New()
   
   	cur := handler.collection.FindOne(handler.ctx, bson.M{
   		"username": user.Username,
   		"password": string(h.Sum([]byte(user.Password))),
   	})
   	if cur.Err() != nil {
   		c.JSON(http.StatusUnauthorized, gin.H{"error": "Invalid username or password"})
   		return
   	}
   
   	sessionToken := xid.New().String()
   	session := sessions.Default(c)
   	session.Set("username", user.Username)
   	session.Set("token", sessionToken)
   	session.Save()
   
   	c.JSON(http.StatusOK, gin.H{"message": "User signed in"})
   }
   
   
   func (handler *AuthHandler) RefreshHandler(c *gin.Context) {
   	session := sessions.Default(c)
   	sessionToken := session.Get("token")
   	sessionUser := session.Get("username")
   	if sessionToken == nil {
   		c.JSON(http.StatusUnauthorized, gin.H{"error": "Invalid session cookie"})
   		return
   	}
   
   	sessionToken = xid.New().String()
   	session.Set("username", sessionUser.(string))
   	session.Set("token", sessionToken)
   	session.Save()
   
   	c.JSON(http.StatusOK, gin.H{"message": "New session issued"})
   }
   
   func (handler *AuthHandler) SignOutHandler(c *gin.Context) {
   	session := sessions.Default(c)
   	session.Clear()
   	session.Save()
   	c.JSON(http.StatusOK, gin.H{"message": "Signed out..."})
   }
   ```

   

## 功能

### 日志

```go
log.Fatal(http.ListenAndServe(":8000", nil))
```



### JSON

1. 使用```res1B, _ := json.Marshal(res1D)```将其转为字节列表[]byte的对象，然后调用string转为字符串

> 只有 `可导出` 的字段才会被 JSON 编码/解码。必须以大写字母开头的字段才是可导出的。

```go
package main
import (
    "encoding/json"
    "fmt"
    "os"
)
type response1 struct {
    Page   int
    Fruits []string
}
type response2 struct {
    Page   int      `json:"page"`
    Fruits []string `json:"fruits"`
}

res1D := &response1{
        Page:   1,
        Fruits: []string{"apple", "peach", "pear"}}
res1B, _ := json.Marshal(res1D)
fmt.Println(string(res1B))
```

2. 使用`err := json.Unmarshal(byt, &dat)`对数据解码，第二个参数指定转化后的数据类型，也是保存到的变量。嵌套的数据类型需要进一步转化。

```go
package main

import (
	"encoding/json"
	"fmt"
)

func main() {
	byt := []byte(`{"num":6.13,"strs":["a","b"]}`)
	var dat map[string]interface{}
	if err := json.Unmarshal(byt, &dat); err != nil {
		panic(err)
	}
	fmt.Println(dat)
	fmt.Println(dat["num"])
	//这个值的类型是接口，因此需要将其转化为数组才能使用索引
	fmt.Println(dat["strs"])
    //data.(type)类型转换
	strs := dat["strs"].([]interface{})
	fmt.Println(strs[0])
}

```



3. 将 JSON 解码为自定义数据类型。 这样做的好处是，可以为我们的程序增加附加的类型安全性， 并在访问解码后的数据时不需要类型断言。

```go
package main

import (
	"encoding/json"
	"fmt"
)

type response2 struct {
	Page   int      `json:"page"`
	Fruits []string `json:"fruits"`
}

func main() {
	res1D := &response2{
		Page:   1,
		Fruits: []string{"apple", "peach", "pear"}}
	res1B, _ := json.Marshal(res1D)
	fmt.Println(string(res1B))

	res := response2{}
	json.Unmarshal(res1B, &res)
	fmt.Println(res)
	fmt.Println(res.Fruits[0])
}

```



4.  JSON 编码流传输到指定的writer，也就是json.NewEncoder(w)的w

```go
enc := json.NewEncoder(os.Stdout)
d := map[string]int{"apple": 5, "lettuce": 7}
enc.Encode(d)
```

5.  JSON 从r.Body流解码，并赋值给tempCity

```go
decoder := json.NewDecoder(r.Body)
err := decoder.Decode(&tempCity)
```



### sqlite3

`go get github.com/mattn/go-sqlite3`

使用纯sql进行CRUD

```go
package main

import (
	"database/sql"
	"log"

	_ "github.com/mattn/go-sqlite3"
)

// Book is a placeholder for book
type Book struct {
	id     int
	name   string
	author string
}

func dbOperations(db *sql.DB) {
	// Create，准备好一个sql语句
	statement, _ := db.Prepare("INSERT INTO books (name, author, isbn) VALUES (?, ?, ?)")
	statement.Exec("A Tale of Two Cities", "Charles Dickens", 140430547)
	log.Println("Inserted the book into database!")

	// Read
	rows, _ := db.Query("SELECT id, name, author FROM books")
	var tempBook Book
	for rows.Next() {
		//对每行数据赋值给对象
		rows.Scan(&tempBook.id, &tempBook.name, &tempBook.author)
		log.Printf("ID:%d, Book:%s, Author:%s\n", tempBook.id, tempBook.name, tempBook.author)
	}
	// Update
	statement, _ = db.Prepare("update books set name=? where id=?")
	statement.Exec("The Tale of Two Cities", 1)
	log.Println("Successfully updated the book in database!")

	//Delete
	statement, _ = db.Prepare("delete from books where id=?")
	statement.Exec(1)
	log.Println("Successfully deleted the book in database!")
}

func main() {
	db, err := sql.Open("sqlite3", "./books.db")
	if err != nil {
		log.Println(err)
	}
	// Create table
	statement, err := db.Prepare("CREATE TABLE IF NOT EXISTS books (id INTEGER PRIMARY KEY, isbn INTEGER, author VARCHAR(64), name VARCHAR(64) NULL)")
	if err != nil {
		log.Println("Error in creating table")
	} else {
		log.Println("Successfully created table books!")
	}
	statement.Exec()
	dbOperations(db)
}

```

### 使用RabbitMq

1. docker安装：`docker pull rabbitmq:3.10-rc-management`

> 只有management才有web界面

2. 运行：` docker run -d --hostname my-rabbit --name rabbit0 -p 8080:15672 rabbitmq:3.10-rc-management`

### 使用postman测试

传输json数据要用body里的raw的json格式，否则有可能不行



### 使用MongoDB

1. 安装驱动：`go get go.mongodb.org/mongo-driver/mongo`

2. 连接到MongoDB

```go
package main

import (
	"context"
	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
	"go.mongodb.org/mongo-driver/mongo/readpref"
	"log"
)

func connect() {
	ctx := context.Background()
	client, err := mongo.Connect(ctx, options.Client().ApplyURI("mongodb://admin:password@localhost:27017/test?authSource=admin"))
	if err = client.Ping(context.TODO(), readpref.Primary()); err != nil {
		log.Fatal(err)
	}
	log.Println("Connected to MongoDB")
}
```

3. 将json导入

```go
package main

import (
	"context"
	"encoding/json"
	"fmt"
	"go.mongodb.org/mongo-driver/bson/primitive"
	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
	"go.mongodb.org/mongo-driver/mongo/readpref"
	"log"
	"os"
	"time"
)

type Recipe struct {
	//swagger:ignore

	ID           primitive.ObjectID `json:"id" bson:"_id"`
	Name         string             `json:"name" bson:"name"`
	Tags         []string           `json:"tags" bson:"tags"`
	Ingredients  []string           `json:"ingredients" bson:"ingredients"`
	Instructions []string           `json:"instructions" bson:"instructions"`
	PublishedAt  time.Time          `json:"publishedAt" bson:"publishedAt"`
}

func initmongodb() {
	// 从json中读取到recipes
	recipes := make([]Recipe, 0)
	fmt.Print(recipes)
	file, err := os.ReadFile("E:\\All Files\\Programmaing\\Go\\Building-Distributed-Applications-in-Gin-main\\chapter03\\recipes.json")

	if err != nil {
		panic(err)
	}
	_ = json.Unmarshal([]byte(file), &recipes)
	fmt.Print(recipes)
	ctx := context.Background()
	client, err := mongo.Connect(ctx, options.Client().ApplyURI("mongodb://admin:password@localhost:27017/test?authSource=admin"))
	if err = client.Ping(context.TODO(), readpref.Primary()); err != nil {
		log.Fatal(err)
	}
	log.Println("Connected to MongoDB")
}
	// 把切片转为数组
	var listOfRecipes []interface{}
	for _, recipe := range recipes {
		listOfRecipes = append(listOfRecipes, recipe)
	}
	// 表名为demo
	collection := client.Database("demo").Collection("recipes")
	insertManyResult, err := collection.InsertMany(ctx, listOfRecipes)
	if err != nil {
		log.Fatal(err)
	}
	log.Println("Inserted recipes: ", len(insertManyResult.InsertedIDs))
```



### 使用Redis

1. 安装模块：`go get github.com/go-redis/redis/v8`
2. 连接：

```go
package main

import (
	"github.com/go-redis/redis"
)

redisClient := redis.NewClient(&redis.Options{
    Addr:     "localhost:6379",
    Password: "",
    DB:       0,
})

status := redisClient.Ping()
log.Println("Redis：", status)   # PONG即为成功
```

3. 使用：

```go
// 取值
val, err := redisClient.Get("recipes").Result()
// 赋值，注意redis的值只能是字符串
data, _ := json.Marshal(recipes)
handler.redisClient.Set("recipes", string(data), 0)
// 删除
redisClient.Del("recipes")
```

