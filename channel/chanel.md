#channelの話
goroutine同士の変数はどうやって共有すべきでしょうか？  // 正確には共有するのは変数ではなく値
グローバルで変数を宣言すればよいでしょうか？  
しかしそれでは並列処理時に変数の競合が起こってしまいますよね。そこでgoroutineの変数の共有にはchannelというものを使います。  
channelというのはgoroutine同士の通信に使用するものです。  

```go
chanel := make(chan 型)
chanel := make(chan 型 ,容量)
```
のように宣言することができます。  
channelはスライスやマップなどと同じく参照型の一種であり、値を作成するには「make」組み込み関数を使います。
channelは送受信する型をきめるなければなりません、
またデフォルトの容量は0で、指定することもできます。  

チャネルを使ったコードは次のようです
```go
package main

import "fmt"

func main() {
	chanel := make(chan int)　//チャネルを作成
	go func(s chan<- int) { //送信チャネル
		for i := 1; i < 5; i++ {　//chanelに０～４の数字を順番に送信
			s <- i //ここで送信
		}
		close(s) //チャネルのクローズ
	}(chanel)

	for { // range でもよさそうだけど、意図的にやっているのなら OK
		value, ok = <-chanel //チャネルを受信、valueには値が、okにはクローズしたか判別できるbool型が入る
    if !ok { // go fmt できてない
      break
    }
	fmt.Println(value)
  }
}
```
これを実行すると

```sh
$ go fmt main.go
1  
2  
3  
4  
```

となり最初に宣言した i の値がchannelを経由してvalueへと格納されているのがわかります。

これでchannelが送受信しているのがわかりましたね！
おさらいですが  

```go
chan <- 要素型　//送信専用チャネルの型
<- chan 要素型  //受信専用チャネルの型
```

といったかたちで、送信、受信のチャネルの型を作成できて

```go
チャネル　<- 送信する値　//送信
<- チャネル //受信
```

という形で実際に値を送受信できます。  
チャネルのループは // ??? チャネルを閉じるには？

```go
close(チャネル)
```

とすることができます.  
okにcloseされたかどうかわかるbool型が格納されるので  
これによって送信チャネルが終了したか判別することができます.よって

```go
value, ok := <-chanel

if !ok{
  break
}
```
chanelがcloseしokがfalseになると、if !ok内をくぐってbreakで抜け出すことができる、というわけです。

Effective Goにはこんなスローガンが書いてあります  

共有メモリを使って通信せず、通信によってメモリを共有せよ。

一番最初に言った通り、goはキチンとチャネルを使いこなすことによって値の競合を防ぐことができます。  
ならば共有変数の代わりになる値などもチャネルで引き回すべきですよね?  
```go
package main

import (
	"fmt"
	"os"
)

const goroutines = 10

func main() {
	counter := make(chan int, 1)
	for i := 0; i < goroutines; i++ {
		go func(counter chan int) {

			value := <-counter

			value++

			fmt.Println("count", value)

			if value == goroutines {
				os.Exit(0)
			}

			counter <- value
		}(counter)

	}

	counter <- 0

	for {

	}

}
```
このコードを実行するためには環境変数GOMAXPROCSを2以上にしなければなりません。  
環境変数GOMAXPROCSを2以上に設定することで、2つ以上のCPUを使った並列処理が可能になります。  
GOMAXPROCSが1のままだと1つのCPUでの処理となるので、for無限ループに入った時にほかのものを処理できません。  
そのため、環境変数GOMAXPROCSを2以上にする必要があります。  
もし環境変数を変えないままこのコードを実行したならば、「Ctrl」+「c」で抜け出してください。  
// GOMAXPROCS を変えての実行の仕方を書くと分かりやすそう

#selectの話

少し長引きましたが、申し訳ないです、チャネルの話はまだ続きます。
selectとは複数のチャネルから送受信待ちをするものです。
selectはcase節に指定したチャネルが通信可能になったときに、case節の中の処理を実行する、といったものです。  
 ```go
select {

	case　条件
	 	処理

	case　条件
		処理

	default:
		処理
}
```
そうですね、switchとよく似てますね、条件にマッチしたときに処理が実行されるという形です.　　
 defaultは他のどのcaseにも当てはまらなかったときに呼び出されます。

```go
// go fmt できてない
package main

import (
	"fmt"
)

func main(){

a := make(chan int)
b := make(chan int)
c := make(chan int)

go func() { for { a <- 0 } }()

for i := 0; i < 10; i++ {
    select {
    case <-a:
        fmt.Println("aを受信した")
    case <-b:
        fmt.Println("bを受信した")
    case c <- 0:
        fmt.Println("cを受信した")
    }
}
}
```
これはa,b,cというチャネルがあって、その中で実際に受信しているのはaだけですのでaだけが表示されるというわけです  
ではさっき言った通りのdefaultを使ってみましょう。
```go
package main

import (
	"fmt"
)

func main(){

a := make(chan int)
b := make(chan int)
c := make(chan int)
d := make(chan int)

go func() { for { d <- 0 } }()

for i := 0; i < 10; i++ {
		select {
		case <-a:
				fmt.Println("aを受信した")
		case <-b:
				fmt.Println("bを受信した")
		case c <- 0:
				fmt.Println("cを受信した")
		default:
				fmt.Println("なにも受信していないです")
		}
}
}
```
チャネルdは走っていないので、defaultが呼び出されましたね。  
実行可能なcase節が複数ある時はどうなるのでしょうか？  
```go
// go fmt
package main

import (
	"fmt"
)

func main(){

a := make(chan int)
b := make(chan int)
c := make(chan int)

go func() { for { a <- 0 } }()

for i := 0; i < 10; i++ {
    select {
    case <-a:
        fmt.Println("a")
    case <-b:
        fmt.Println("b")
    case c <- 0: // この c から読み出す人がいないから、全部 a になってる気がする。
        fmt.Println("c")
    }
}
}
```
ん？ランダムですね？もう一度実行すると結果が変わりますね？ // 手元では全部 a になりました。
そうなんです、selectは実行可能なcase節が複数あるとそこからランダムで実行される仕様なんです！  
以上でgoroutineとchannelの解説です！ありがとうございました！
