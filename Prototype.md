デザインパターン勉強会　第六回：Prototypeパターン
# はじめに  
本エントリーは某社内で実施するデザインパターン勉強会向けの資料となります。  
本エントリーで書籍「[Java言語で学ぶデザインパターン入門](http://www.hyuki.com/dp/)」をベースに学習を進めますが、サンプルコードはC#に置き換えて解説します。

第五回：[Singletonパターン](http://qiita.com/Keenag/items/cae53d55cb953562fee4#_reference-478e63b7263c943c7f1f)  
第七回：Builderパターン

---
# 概要：Prototypeパターンとは
Prototypeパターンは、既存のインスタンスを<b>コピー</b>して新しいインスタンスを作成するデザインパターンである。  
Prototypeパターンを利用してインスタンスを生成する場合、あらかじめ「ひな形（Prototype）」となるインスタンスを用意しておき、これをコピーすることでインスタンスが生成される。  
Prototypeパターンでのクラスの役割は以下の通り。  

|名前|役割|
|:--|:--|
|Product|コピーを行うメソッドを定義するインターフェイス|
|ConcretePrototype|インスタンス（自身）をコピーし新しいインスタンスを作成する|
|Client|インスタンスをコピーするメソッドを利用し、新しいインスタンスの作成指示を行う|


次の項目に、Prototypeパターンの実装例を示す。

---
# 実装例
## サンプルクラス図  
TBD

---
## 各クラスの役割  
|名前|役割|
|:--|:--|
|IProduct|複製を行うメソッドを定義するインターフェイス|
|Manager|インスタンスの複製を指示するクラス|
|MessageBox|特定の文字を利用し、文字列を飾枠のように囲うクラス|
|UnderlinePen|特定の文字を利用し、文字列に下線を引くクラス|
|Program|動作確認用クラス|

---
## IProduct
複製を行うメソッドを定義するインターフェイス。  
Javaでは、IProductインターフェイスを継承したクラスが```Clone```メソッドを利用するために、```Cloneable```インターフェイスの継承が必要であったが、C#で記述し```MenberwiseClone```を利用する場合は、全クラスが```Object```クラスを無条件に継承しているため、インターフェイスの実装は不要である。  
但し、DeepCopy（※１）を行うためにコピー作成の部分を自作したい場合などは、```ICloneable```インターフェイスを継承することもある。

```csharp
namespace DesignPattern_Prototype.framework
{
    /// <summary>
    /// Prototype：複製を行うメソッドを定義するインターフェイス
    /// </summary>
    interface IProduct
    {
        IProduct createClone();

        void use(string s);
    }
}
```
---
## Manager  
インスタンスのコピー指示はこのクラスより行う。  
（筆者考：Hashtable型のshowcaseは便宜上作成されたものであり、Prototypeパターンの本質には関与しない？）


```csharp
using System.Collections;

namespace DesignPattern_Prototype.framework
{
    /// <summary>
    /// Client：ひな形インスタンスのコピーを指示するクラス
    /// </summary>
    class Manager
    {
        /// <summary>
        /// ひな形インスタンスを保持するHashtable
        /// </summary>
        private Hashtable showcase = new Hashtable();

        /// <summary>
        /// 引数で渡されたひな形インスタンスをshowcase内に格納する
        /// </summary>
        /// <param name="name"></param>
        /// <param name="proto"></param>
        public void register(string name, IProduct proto)
        {
            showcase.Add(name, proto);
        }

        /// <summary>
        /// showcase内に存在するひな形インスタンスのコピーを返す
        /// </summary>
        /// <param name="protoname"></param>
        /// <returns></returns>
        public IProduct create(string protoname)
        {
            IProduct p = (IProduct)showcase[protoname];
            return p.createClone();
        }
    }
}
```
---
## MessageBox
IProductインターフェイスを継承している。  
本クラスでのインスタンスのコピーはShallowCopy（値型のみのコピー）であり、Objectクラスで定義・実装されているMenberwiseCloneメソッドを利用して作成される。  
MenberwiseCloneメソッドは、呼び出し元のインスタンスのコピーを提供するため、自身以外のクラスからコピーできるようにするためにはMenberwiseCloneメソッドをラップする必要がある。  

```csharp
using DesignPattern_Prototype.framework;
using System;

namespace DesignPattern_Prototype
{
    /// <summary>
    /// ConcretePrototype：特定の文字を利用し、文字列を飾枠のように囲うクラス
    /// </summary>
    class MessageBox : IProduct
    {
        private char _decochar;

        public MessageBox(char decochar)
        {
            _decochar = decochar;
        }

        public IProduct createClone()
        {
            // Javaのcloneメソッドに対応するメソッドを利用し、自身のコピーを作成している
            // このメソッドはObjectクラスで定義・実装されているため、Objectを暗黙的に継承するすべてのクラスで利用可能である。
            // なお、Javaとは異なり、try-catchによるエラーハンドリングは不要
            return (IProduct)MemberwiseClone();
        }

        /// <summary>
        /// 特定の文字を利用し、文字列を飾枠のように囲う
        /// </summary>
        /// <param name="s"></param>
        public void use(string s)
        {
            int length = s.Length;
            for (int i = 0; i < length + 4; i++)
            {
                Console.Write(_decochar);
            }
            Console.WriteLine("");
            Console.WriteLine(_decochar + " " + s + " " + _decochar);
            for (int i = 0; i < length + 4; i++)
            {
                Console.Write(_decochar);
            }
            Console.WriteLine("");
        }
    }
}
```
---
## UnderlinePen
IProductインターフェイスを継承している。  
MessageBoxクラスと同様に、インスタンスのコピーはShallowCopy（値型のみのコピー）である。  

```csharp
using System;
using DesignPattern_Prototype.framework;

namespace DesignPattern_Prototype
{
    /// <summary>
    /// ConcretePrototype：特定の文字を利用し、文字列に下線を引くクラス
    /// </summary>
    class UnderlinePen : IProduct
    {
        private char _ulchar;

        public UnderlinePen(char ulchar)
        {
            _ulchar = ulchar;
        }

        /// <summary>
        /// 自身のコピーを返す
        /// </summary>
        /// <returns></returns>
        public IProduct createClone()
        {
            return (IProduct)MemberwiseClone();
        }

        /// <summary>
        /// 特定の文字を利用し、文字列に下線を引く
        /// </summary>
        /// <param name="s"></param>
        public void use(string s)
        {
            int length = s.Length;
            Console.WriteLine("\"" + s + "\"");
            for (int i = 0; i < length + 2; i++)
            {
                Console.Write(_ulchar);
            }
            Console.WriteLine("");
        }
    }
}
```
---
## 実行結果 
Managerがコピーを指示した、フィールドに値が投入されているインスタンスがコピーされていることが確認できる。
```
"Hello, world"
--------------
****************
* Hello, world *
****************
////////////////
/ Hello, world /
////////////////
```
---
# 想定される用途  
Prototypeパターンは、以下のような場合に用いることが想定される。
* ClientからConcretePrototypeへの依存関係を切りたい場合  
* インスタンスのコピーを利用したい場合  
（特に、図形の描画処理などの複雑な処理を経て作成されるインスタンスや、同じ値であることを保証したいインスタンスなど）

---
# 補足１：ShallowCopyとDeepCopy  

インスタンスをコピーする際、参照型の変数をどのようにコピーするかによりコピー方法の呼び名が変わる。  
コピーしたインスタンスの参照先をそのまま保持する場合はShallowCopyとなる。この場合コピー元の参照型の値を変更するとコピー先の参照型の値も連動して変更されてしまう。本記事の実装例ではShallowCopyを利用している。  
コピー元のインスタンスの参照先オブジェクトをコピーし、これをコピー先の参照型に代入した場合、DeepCopyとなる。この場合は、コピー元の参照型の値を変更しても、
コピー先の参照型とは別オブジェクトであるため、コピー先の値は変更されない。  
  
本記事ではShallowCopyを利用しているが、DeepCopyを利用したい場合、ConcretePrototypeクラスのCreateCloneメソッドで参照型の値を再設定するように変更する必要があるだろう。

---

# サンプルコード

[https://github.com/aconit96/Prototype](https://github.com/aconit96/Prototype)


# 前回・次回リンク
第五回：[Singletonパターン](http://qiita.com/Keenag/items/cae53d55cb953562fee4#_reference-478e63b7263c943c7f1f)  
第七回：[Builderパターン]
