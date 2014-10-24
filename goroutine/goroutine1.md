
#gorutine
gorutine(読み方はゴルーチン)というのは複数の処理を並列に処理できる機能のようなものです。

```go
package main

  import (
  "fmt"
  "time"
  )

  func main() {
          test()
          time.Sleep(3 *time.Second)
          fmt.Println("fin")
  }

  func test() {
          for i := 0; i < 5; i++ {
                  fmt.Println("hello",i)
                  time.Sleep(1 *time.Second)
                }
}
}

```

これを実行すると  

hello 0
hello 1  
hello 2  
hello 3  
hello 4  
fin  
となり、func testが1~4までの数字を出力するのに４秒かかっていることから、その間func mainが呼ばれていないので、<u>並列に処理していない</u>ことがわかります。

<b>並列に処理したい</b>

並列に処理したいときは

<span style="font-size: 150%">go 並列に処理したい関数
</span>

と書くことができます。

やってみると

```go
package main

import (
  "fmt"
  "time"
)

func main() {
        test()
        time.Sleep(3 *time.Second)
        fmt.Println("fin")
}

func test() {
        for i := 0; i < 5; i++ {
                fmt.Println("hello",i)
                time.Sleep(1 *time.Second)
              }
}
}
```

hello 0  
hello 1  
hello 2  
fin  

となります、前回のはhello 4 まで出力されていたのに、今回はfunc mainと func testを並行に処理しているので、4秒たってhello 4が出力される前に３秒で終了するfunc mainが終了してしまうので、hello 2までしか表示されないということです。

以上がgoroutineの簡単な解説です

簡単にまとめると
+ goroutineは並列な処理を可能にする
- しかしfunc mainが終了してしまうとgoroutineも終了してしまう

ということです、  
またgoroutineの終了はmain関数を終了させるだけでなく、returnで抜けたり、runtime.Goexit()を実行して終わらせることもできます。
