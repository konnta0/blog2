---
{"dg-publish":true,"permalink":"/Engineering/-.NET/Packages/NSubstitute/","dgPassFrontmatter":true,"created":"2025-02-28T12:54:55.716+09:00"}
---

#dotnet #test #mock

# 概要
.NET のテスト用のモックライブラリです
公式サイトはこちら  https://nsubstitute.github.io/
2025/02 時点の最新は `5.3.0` です
補助として [C# analyser](https://nsubstitute.github.io/help/nsubstitute-analysers/) と [VB 向けのパッケージ](https://nsubstitute.github.io/help/nsubstitute-analysers/)も存在します

公式サイトの引用
>Don’t sweat the small stuff
>Mock, stub, fake, spy, test double? Strict or loose? Nah, just substitute for the type you need!
>NSubstitute is designed for Arrange-Act-Assert (AAA) testing, so you just need to arrange how it should work, then assert it received the calls you expected once you're done. Because you've
>got more important code to write than whether you need a mock or a stub.

>些細なことでも気にするな
モック、スタブ、フェイク、スパイ、テストダブル？ストリクトかルーズか？いいえ、必要な型を代入するだけです！NSubstituteは、Arrange-Act-Assert（AAA）テストのために設計されているので、どのように動作するかをアレンジし、完了したら期待したコールを受け取ったことをアサートするだけでいい。モックが必要なのかスタブが必要なのかよりも、書くべき重要なコードがあるからだ。

# インストール
[NuGet](https://docs.microsoft.com/en-us/nuget/quickstart/use-a-package) からインストールする場合
```
Install-Package NSubstitute
```

C# Analyser や VB 向けのパッケージをインストールする場合
```
Install-Package NSubstitute.Analyzers.CSharp
もしくは
Install-Package NSubstitute.Analyzers.VisualBasic
```

> [!note] note
> NSubstitute が正しく動作するのは、インターフェイスか、テストアセンブリからオーバーライド可能なクラスメンバだけ
> 
>  非仮想メンバや内部仮想メンバを持つクラスの代入は、実際のコードが誤ってテスト内で実行される可能性がある
> 
> Analyser をインストールすると全てでは無いが警告を出してくれる
> 

# 色々使い方
サンプルとして `ICalculator` を利用します
尚、テストフレームワークは `NUnit` を利用します
```cs
public interface ICalculator  
{  
    int Add(int a, int b);  
    string Mode { get; set; }  
    event EventHandler PoweringUp;  
}
```

## 関数をモックする
```cs
[Fact]  
public void Test1()  
{  
    var calculator = Substitute.For<ICalculator>(); // モックする ICalculator を取得  
    calculator.Add(1, 2).Returns(3); // Add 関数に 1 と 2 が渡された時に 3 を返す  
    Assert.Equal(3, calculator.Add(1, 2));  // モックされて 3 が返るのでテストが成功する
}
```

## モックした関数が呼び出されたか検証する
`Received`, `DidNotReceived` 関数を利用すると該当の関数(引数含む)が呼び出された否かを検証することができる。また、エラーメッセージとしても説明してくれます

```cs
[Fact]  
public void Test2()  
{  
    var calculator = Substitute.For<ICalculator>(); // モックする ICalculator を取得  
    calculator.Add(1, 2); // Add 関数の呼び出し  
    calculator.Received().Add(1, 2); // Add 関数に引数 1 と 2 の引数の組み合わせで呼ばれたか検証する  
    calculator.DidNotReceive().Add(5, 7); // Add 関数に引数 5 と 7 の引数の組み合わせで呼ばれていないか検証する  
    // 以下だと 1 と 2 の組み合わせは既に呼ばれているのでテストが失敗する  
    // calculator.DidNotReceive().Add(1, 2);   
    // エラーは以下のようになる  
    // NSubstitute.Exceptions.ReceivedCallsException: Expected to receive no calls matching:  
    // NSubstitute.Exceptions.ReceivedCallsException    // Expected to receive no calls matching:    //  Add(1, 2)    // Actually received 1 matching call:    //  Add(1, 2)}

```


## プロパティをモックする
```cs
[Fact]  
public void Test3()  
{  
    var calculator = Substitute.For<ICalculator>(); // モックする ICalculator を取得  
  
    calculator.Mode.Returns("DEC"); // Mode プロパティが "DEC" を返すように設定  
    Assert.Equal("DEC", calculator.Mode); // Mode プロパティが "DEC" を返すか検証  
  
    calculator.Mode = "HEX"; // Mode プロパティに "HEX" を設定(上書きすることもできる)  
    Assert.Equal("HEX", calculator.Mode); // Mode プロパティが "HEX" を返すか検証  
}
```

## argument matching を利用して条件に幅を持たせる
`Arg.Any` や `Arg.Is` を用いると呼び出した関数の引数に条件を設定できます
```cs
[Fact]  
public void Test4()  
{  
    var calculator = Substitute.For<ICalculator>(); // モックする ICalculator を取得  
  
    calculator.Add(10, -5); // Add 関数の呼び出し  
    calculator.Received().Add(10, Arg.Any<int>()); // Add 関数に 10 と任意の整数が渡されたか検証  
    calculator.Received().Add(10, Arg.Is<int>(static x => x < 0)); // Add 関数に 10 と負の整数が渡されたか検証  
}
```

> [!example] tips
>  argument matching と `Returns()` を組み合わせると多くの挙動が再現できます

```cs
[Fact]  
public void Test5()  
{  
    var calculator = Substitute.For<ICalculator>(); // モックする ICalculator を取得  
  
    calculator  
       .Add(Arg.Any<int>(), Arg.Any<int>()) // 任意の整数が 2 つ渡された時に  
       .Returns(x => (int)x[0] + (int)x[1]); // 2 つの整数の和を返すように設定  
    var actual = calculator.Add(5, 10); // 具象クラスの実装はしていないが、Add 関数に 5 と 10 が渡された時に 15 が返る  
    Assert.Equal(15, actual); //  5 と 10 を渡すと 15 が返る  
}
```

## 複数の戻り値を再現する
```cs
[Fact]  
public void Test6()  
{  
    var calculator = Substitute.For<ICalculator>(); // モックする ICalculator を取得  
  
    calculator.Mode.Returns("HEX", "DEC", "BIN"); // Mode プロパティが "HEX"、"DEC"、"BIN" を返すように設定  
    Assert.Equal("HEX", calculator.Mode); // Mode プロパティが "HEX" を返すか検証  
    Assert.Equal("DEC", calculator.Mode); // Mode プロパティが "DEC" を返すか検証  
    Assert.Equal("BIN", calculator.Mode); // Mode プロパティが "BIN" を返すか検証  
  
    Assert.Equal("BIN", calculator.Mode); // 3 回目以降(設定された戻り値の個数より多い呼び出し)は最後の値が返る  
}
```

## イベントをモックする
```cs
[Fact]  
public void Test7()  
{  
    var calculator = Substitute.For<ICalculator>(); // モックする ICalculator を取得  
    var eventWasRaised = false;  
    calculator.PoweringUp += (sender, args) => eventWasRaised = true; // PoweringUp イベントが発生した時に eventWasRaised を true にする  
  
    calculator.PoweringUp += Raise.Event(); // PoweringUp イベントを発生させる (sender と eventArgs はデフォルト値)  
    Assert.True(eventWasRaised);  
}
```



参考:
https://nsubstitute.github.io/help/getting-started/