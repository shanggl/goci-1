goci
====
> 这是我目前找到的最好的一个go语言开发oralce数据库的oci接口。
>
> 总体实现的比较好，还是标准的。
>
> 推荐使用。

oci for go. It's goci!

* 编译注意事项：
* cgo 编译时需加的头文件和连接库
* 将oci8.pc 放到 你 PKG_CONFIG_PATH 下，并确保你安装了 pkg-config
* 其中oci8.pc 的内容见具体文件
 

* 如果你使用简易的安装包，请在
* [http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html](http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html)
* 下载：
* oracle-instantclient11.2-basic-11.2.0.4.0-1.x86_64.rpm
* oracle-instantclient11.2-devel-11.2.0.4.0-1.x86_64.rpm 
* oracle-instantclient11.2-sqlplus-11.2.0.4.0-1.x86_64.rpm (可选） 

* 并安装和设置oracle环境变量。包括： $ORACLE_HOME LD_LIBRARY_PATH 
* 同时需设置 TNS_ADMIN 指向你的tnsnames.ora 文件地址,推荐同期安装sqlplus，以sqlplus能连上数据库为准
* export  TNS_ADMIN=/home/oracle/app/oracle/product/11.2.0/client_1/network/admin
* 
### ORACLE_HOME 以您系统实际情况进行修改。

例子：
        package main

        import (
                "database/sql"
                _ "goci"  // 根据实际部署情况修改
                "os"
                "log"
        )

        func main() {
                // 为log添加短文件名,方便查看行数
                log.SetFlags(log.Lshortfile | log.LstdFlags)

                log.Println("Oracle Driver example")

                os.Setenv("NLS_LANG", "")
                dsn := os.Getenv("ORACLE_DSN") // 把用户名/口令@SID  定义到此环境变量中
                if dsn == "" {
          	os.Exit(2) // 出错退出
                }
                db, _ := sql.Open("goci",dsn)

                rows, err := db.Query("select 3.14, 'foo' from dual")
                if err != nil {
                        log.Fatal(err)
                }
                defer db.Close()

                for rows.Next() {
                        var f1 float64
                        var f2 string
                        rows.Scan(&f1, &f2)
                        log.Println(f1, f2) // 3.14 foo
                }
                rows.Close()

                // 先删表,再建表
                db.Exec("drop table sdata")
                db.Exec("create table sdata(name varchar2(256))")

                db.Exec("insert into sdata values('中文')")
                db.Exec("insert into sdata values('1234567890ABCabc!@#$%^&*()_+')")

                rows, err = db.Query("select * from sdata")
                if err != nil {
                        log.Fatal(err)
                }

                for rows.Next() {
                        var name string
                        rows.Scan(&name)
                        log.Printf("Name = %s, len=%d", name, len(name))
                }
                rows.Close()
        }


        此程序在Ubuntu 15.04 的 64 位机器上编译测试正常通过。
        其中Oracle的客户端通过 alien 安装
