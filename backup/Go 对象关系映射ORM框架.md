# gorm

初学go时在项目中我们是直接使用go自带的“database/sql”数据库连接API，“github.com/go-sql-driver/mysql”MySQL驱动来写原生的sql处理事务。目前可以通过封装好的orm框架，节省重复操作，提高代码可读性。

- O = Object对象，即程序中的struct
- R = Relation关系，即关系型数据库mysql
- M = Mapping映射
结构体 -> 数据表、结构体实例 -> 数据行、结构体字段 -> 表字段
优点：提高开发效率，不需要使用sql语句；
缺点：执行性能较差、灵活性较差、弱化了SQL能力

## 数据模型定义

### 表名、列名与结构体对应

在gorm中，表名是结构体名的复数形式、列名是字段名的蛇形小写。
即表名users、结构体名User
数据库定义如下：
```
CREATE TABLE `areas` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键id',
  `area_id` int(11) NOT NULL COMMENT '区县id',
  `area_name` varchar(45) NOT NULL COMMENT '区县名',
  `city_id` int(11) NOT NULL COMMENT '城市id',
  `city_name` varchar(45) NOT NULL COMMENT '城市名称',
  `province_id` int(11) NOT NULL COMMENT '省份id',
  `province_name` varchar(45) NOT NULL COMMENT '省份名称',
  `area_status` tinyint(3) NOT NULL DEFAULT '1' COMMENT '该条区域信息是否可用 ： 1:可用  2：不可用',
  `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `updated_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_IN

```
gorm中对应结构体定义如下：
```
type Area struct {
	Id int
	AreaId int
	AreaName string
	CityId int
	CityName string
	ProvinceId int
	ProvinceName string
	AreaStatus int
	CreatedAt time.Time
	UpdatedAt time.Time
}
```
可通过如下参数设置，禁用表名复数，即User对应表明user
```
// 全局禁用表名复数
db.SingularTable(true) //如果设置为true，结构体User的默认表名为user，使用TableName 设置的表名不会受到影响
```
## CRUD使用

### 数据库连接

```
package main

import (
	"fmt"
	_ "github.com/go-sql-driver/mysql"
	"github.com/jinzhu/gorm"
)

var db *gorm.DB

type User struct {
	Id int
	Name string
	Age int
	Sex byte
	Phone string
}

func init()  {
	var err error
	//dsn格式 user:pass@tcp(ip:port)/dbname?charset=utf8mb4&parseTime=True&loc=Local
	dsn := "root:root@tcp(192.168.116.90:3306)/gorm?charset=utf8mb4&parseTime=True&loc=Local"
	db,err = gorm.Open("mysql",dsn)
	if err != nil{
		panic(err)
	}
	//设置最大连接数
	db.DB().SetMaxOpenConns(100)
	//设置最大空闲连接数
	db.DB().SetMaxIdleConns(50)
	//设置全局表名禁用复数
	db.SingularTable(true)
}
```

### 插入

```
func(user *User) Insert(){
	//这里使用Table()函数，如果没有指定全局表名禁用复数，或者是表名跟结构体名不一样的时候
	//可以在自己的sql中指定表名。如下示例
	db.Table("user").Create(user)
}

func main(){
	var u =User{
		Name: "xieys",
		Age: 20,
	}
	u.Insert()
}
```

### 更新

- Model方法必须与Update方法一起使用
- 效果相当于，Model中设置更新的主键key（如果没有where制定，默认更新的key为id），Update中设置更新的值
- 如果Model中没有制定id值，且没有where条件，那么将更新全表

```
func main(){
	var u =User{
		Id: 10, //注意如果是更新，并且没有where条件的话，必须制定id值，否则会更新全表
		Name: "xieys",
		Age: 21,
	}
	db.Model(&u).Update(u)
}
相当于sql语句：update user set name="xieys",age="21" where id=10
-------------------------------------
func main(){
	var u =User{
		Age: 24,
	}
	db.Model(&u).Where("id = 8").Update(u)
}
这里没有携带Id，所以在更新的时候需要加Where条件，否则更新全表
相当于sql语句：update user set age=24 where id=8
**使用struct的时候，只会更新struct中这些非空的字段。对于string类型字段的""，int类型字段0，bool类型字段的false都会认为是空白纸，不会去更新表**
-------------------------------------
func main(){
	db.Model(&User{}).Where("age = ?",22).Update("name","xxx")
}
相当于sql语句：update user set name="xxx" where age = 22
-------------------------------------
func main(){
	db.Model(&User{}).Where("name = ?","xxx").Update("age",0)
}
相当于sql语句：update user set age = 0 where name="xxx"
-------------------------------------
Select 指定只更新某个字段，没有指定的字段将被忽略
func main(){
	db.Model(&u).Select("name").Update(map[string]interface{}{"name":"","age":0}) //这里age字段将被忽略，不会被更新，更新的只有name字段
	// 如果想要更新name和age字段应该这样写
	//db.Model(&u).Select("name","age").Update(map[string]interface{}{"name":"","age":0})
}
-------------------------------------
忽略掉某些字段
当你的更新的参数为结构体，而结构体中某些字段你又不想去更新，那么可以使用Omit方法过滤掉这些不想update到库的字段,忽略字段使用Select也可以实现，具体看示例4

func main(){
	var u =User{
		Id: 8,
		Name: "xx"
		Age: 24,
	}
	db.Model(&u).Omit("name").Update(&u) //这里name字段不会更新，只会更新age字段
	//db.Model(&u).Select("name").Update(&u) //这里是name字段会更新，age字段被忽略
}
```

### 删除

```
单记录删除
func main(){
	user := User{Id : 8}
	db.Delete(&user)
}
// delete user where id = 8
----------------------
批量删除
func main(){
	db.Delete(&User{},"id > ?",8)
}
// delete user where id > 8
```

### 事务
