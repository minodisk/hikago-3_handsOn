#channelの話
goroutine同士の変数はどうやって共有すべきでしょうか？  
グローバルで変数を宣言すればよいでしょうか？  
しかしそれでは並列処理時に変数の競合が起こってしまいますよね。そこでgoroutineの変数の共有にはchanelというものを使います。  
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

	for {
		value, ok = <-chanel //チャネルを受信、valueには値が、okにはクローズしたか判別できるbool型が入る
    if !ok {
      break
    }
	fmt.Println(value)
  }
}
```
これを実行すると

1  
2  
3  
4  

となり最初に宣言した i の値がchannelを経由してvalueへと格納されているのがわかります。

```
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
チャネルのループは
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
