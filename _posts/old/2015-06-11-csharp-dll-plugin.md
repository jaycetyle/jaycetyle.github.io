---
layout: post
title: "[C#] 用 dll Reflection 做 Plugin 吧！"
description: "用 C# 的 dll 反射做 Plugin"
modified: 2015-01-20
tags: [巫師, C#]
categories: [軟體開發]
image:
    feature: 
    credit: 
    creditlink: 
---

　　這次的內容主要介紹 C# 應用程式如何透過 dll 及反射 (Reflection) 來做 plugin，方法沒有很困難，不過一個 plugin 就是一個 dll 檔，也就等於是一個專案，所以在範例中會開蠻多專案的，手續會稍微繁複一點。現在就開始撰寫我們的範例程式吧！

<!--more-->

### 建立 dll 插件
　　首先要開第一個 dll 專案做為 plugin 的介面 (interface)。雖然我覺得會想看這篇的人應該都已經知道怎麼開 dll 專案了，不過我還是簡單介紹一下，開 dll 專案沒有很困難，在新增專案的時候，專案類型選擇 ClassLibrary，編譯出來的程式就會是 dll 了。

<figure class="large center"> <img src="/images/2015/csharp-plugin-01.png" alt=""> </figure>

　　我的專案名稱是取名為 IPlugin，然後這個介面很單純，程式碼如下

{% highlight C# %}
namespace IPlugin
{
    public interface IPlugin
    {
        string Name { get; }
    }
}
{% endhighlight %}

　　只有取得名稱的功能，夠簡單了吧！接下來開二個 dll 專案，也就是實際實作 IPlugin 介面的類別，我的專案名稱取名叫做 Foo。當然，因為要實作介面，所以要記得把 IPlugin.dll 加到專案的 Reference 裡面

<figure class="half center"> <img src="/images/2015/csharp-plugin-02.png" alt=""> </figure>

　　Foo.cs 裡面的內容一樣超級簡單，只是實作 Name 的 Property 而已～
{% highlight C# %}
namespace Foo
{
    public class Foo : IPlugin.IPlugin
    {
        public virtual string Name { get { return "Foo"; } }
    }
}
{% endhighlight %}

　　為了讓我們的範例效果更明顯一點，我還創了第三個 dll 專案，專案名稱叫做 Bar ，至於程式內容和創建專案的方法我想各位也已經知道了，和 Foo 一樣，只是把 Foo 全部改成 Bar 而已，你也可以創幾個你自己喜歡的來測試。

### 建立主程式

　　把介面和實體的 Plugin 類別都準備好以後，就要來寫我們想要載入 plugin 的主程式啦。我這邊是開了一個 Console 專案作為範例，記得要 Reference IPlugin.dll ，不然會認不得 IPlugin 型別，而 Foo.dll 和 Bar.dll 就不必了，因為我們的**插件是要在執行期動態載入**的！

　　主程式的程式碼內容如下

{% highlight C# %}
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.IO;
using System.Threading.Tasks;
using System.Reflection;

namespace Console
{
    class Program
    {
        static void Main(string[] args)
        {
            // 1. 取得 dll 檔案名稱
            string[] dllFileNames = null;
            dllFileNames = Directory.GetFiles(AppDomain.CurrentDomain.BaseDirectory, "*.dll");

            // 2. 取得 Assembly
            ICollection assemblies = new List(dllFileNames.Length);
            foreach (string dllFile in dllFileNames)
            {
                AssemblyName an = AssemblyName.GetAssemblyName(dllFile);
                Assembly assembly = Assembly.Load(an);
                assemblies.Add(assembly);
            }

            // 3. 取得插件型別
            Type pluginType = typeof(IPlugin.IPlugin);
            ICollection pluginTypes = new List();
            foreach (Assembly assembly in assemblies)
            {
                if (assembly == null)
                    continue;
                Type[] types = assembly.GetTypes();
                foreach (Type type in types)
                {
                    if (type.IsInterface || type.IsAbstract)
                        continue;
                    else if (type.GetInterface(pluginType.FullName) != null)
                        pluginTypes.Add(type);
                }
            }

            // 4. 產生物件並實際操作物件
            foreach (Type type in pluginTypes)
            {
                IPlugin.IPlugin plugin = (IPlugin.IPlugin)Activator.CreateInstance(type);
                System.Console.WriteLine(plugin.Name);
            }
        }
    }
}
{% endhighlight %}

　　稍微長一點了，我做個重點說明，每一個段落分別對應到程式碼的註解數字。

1. 搜尋插件：這裡呼叫了 GetFiles 來取得指定路徑下所有附檔名為 dll 的檔案名稱，而我的路徑指定為 *AppDomain.CurrentDomain.BaseDirectory*，也就是執行檔的所在目錄，所以之後 dll 插件記得要放在案執行檔相同目錄下才有效。

2. 取得 Assembly ，你可以把它想像成"開檔"：流程很簡單，我們已經知道所有 dll 的檔案位置了，只要透過 Assembly.GetAssemblyName 及 Assembly.Load 就可以把 dll 的內容給讀取進來，並把它並放在 List 裡面。

3. 我們已經快要接近目標了，第三步就是去搜尋所有 Assembly 內的類別中是實作 IPlugin 介面的。這裡的小技巧是使用 GetInterface 這一個函數來判斷我們搜尋到的類別是不是實作 IPlugin ，是個話他會 return 一個 Type，不是的話他會 return null，函數說明可以參考 [MSDN](https://msdn.microsoft.com/en-us/library/ayfa0fcd(v=vs.110).aspx)。

4. 到這裡我們已經取得所有插件的類別啦，最後一步是透過 Activator.CreateInstance 這個函數來實際產生物件，有了物件我們就可以實際執行插件的內容囉！這個範例就是把 plugin 裡面的 Name 給印出來 WriteLine(plugin.Name)，實際輸出大概會長這樣
<figure class="large center"> <img src="/images/2015/csharp-plugin-03.png" alt=""> </figure>

　　如果你的程式沒有印出Foo, Bar 或者是你插件定義的內容，**確認一下是否已經把插件的 dll 放到程式的執行目錄下**，也可以試著把插件的 dll 移出執行目錄比較結果，不必重新編譯主程式就會動態載入囉！