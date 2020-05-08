---
title: 数据读取之XML
date: 2017-09-18 23:26:01
tags:
- 数据读取
- XML
---

<p style="text-indent:2em">在Unity游戏中，关于游戏的存档，读档，最高分的记录，排行榜的生成…，这些都离不开数据的存储和读取，在Unity中，使用最多的数据存储包括了，Unity自带的PlayerPrefs，XML,Json，数据库等。之前我们介绍了Unity自带的PlayerPrefs存储方式，这一章，我们来讲讲另一种使用非常广泛的存储方式XML。Xml是Internet环境中跨平台的，依赖于内容的技术，是当前处理结构化文档信息的有力工具。XML是一种简单的数据存储语言，使用一系列简单的标记描述数据，而这些标记可以用方便的方式建立，虽然XML占用的空间比二进制数据要占用更多的空间，但XML极其简单易于掌握和使用。微软也提供了一系列类库来倒帮助我们在应用程序中存储XML文件。</p>

<!-- more -->

## 简介
* [百度百科](https://baike.baidu.com/item/%E5%8F%AF%E6%89%A9%E5%B1%95%E6%A0%87%E8%AE%B0%E8%AF%AD%E8%A8%80/2885849?fr=aladdin&fromid=86251&fromtitle=xml)
* [菜鸟教程](http://www.runoob.com/xml/xml-intro.html)
* [W3school](http://www.w3school.com.cn/xml/xml_intro.asp)

## 读取方式
* 针对XML文档的应用编程接口中，一般有两种模型：W3C制定的DOM(Document Object Method，文档对象模型)和流模型。流模型又分为两种变体:"推"模型（XML的简单API）和"拉"模型（.NET中的流模型）。
* XML模型表格

|         | “推”模型   |  “拉”模型  | DOM|
| --------   | -----  | ----  | ----  |
|      | “推”模型也就是常说的SAX，SAX是一种靠事件驱动的模型。它每发现一个节点就用“推”模型引发一个事件。 |   ．NET在中对XML解析是基于“拉”模型的实现方案。“拉”模型在遍历文档时会把感兴趣的部分从读取器中拉出来，不需要引发事件。在.NET中“拉”模型通过XML阅读器（XMLTextReader类）来实现     |NET中使用XML DOM分析器（XMLDocument）实现DOM模型|
| 缺点         |   必须编写这些事件的处理程序，很麻烦且复杂。不能对文档进行随机访问和修改。   |   XMLTextReader类是只读的，向前的，不能再文档中执行向后导航操作。   |需要一次性加载整个文档到内存中，对于大型文档会造成资源问题。 |
| 优点         |    不需要把XML文档完全加载到内存中，是一个分析大型XML文档的高效API。    |  允许我们以编程的方式访问文档，大大提高了灵活性，可以选择性处理节点，内存中只保存当前节点。XMLTextReader类能检验文档是否格式良好，如果不是良好格式的XML该类在读取过程中会抛出XMLException异常。  | 允许编辑和更新XML文档，可以随机访问文档中的数据，可以使用XPath查询。 |


## 用法
### 命名空间
* [System.Xml](https://www.baidu.com/link?url=TDsgz-0qFGgy0StLT820Crew64rY5n16iooXZ1Rt3jykruwNxUayrXn-G_q-EgcJ7dugLpWM63wOzeWfUwmhZS7tEcPV2sKqW3RLUBBtfOy&wd=&eqid=d2365a820000c1480000000359c01204)
### 主要类
* [XmlDocument][XmlDocument]
[XmlDocument]:https://msdn.microsoft.com/zh-cn/library/system.xml.xmldocument(v=vs.110).aspx
* [XmlTextReader][XmlTextReader]
[XmlTextReader]:https://msdn.microsoft.com/zh-cn/library/system.xml.xmltextreader(v=vs.110).aspx
* [XmlTextWriter][XmlTextWriter]
[XmlTextWriter]:https://msdn.microsoft.com/zh-cn/library/system.xml.xmltextwriter(v=vs.110).aspx
* [System.Xml.Linq][System.Xml.Linq]
[System.Xml.Linq]:https://msdn.microsoft.com/zh-cn/library/system.xml.linq(v=vs.110).aspx

### 使用方法

#### 创建XML
1. XmlSerializer序列化创建
	1.  创建XML对象
    ```Csharp
    using System.Collections;
    using System.Collections.Generic;
    using UnityEngine;
    using System.Xml;
    using System.Xml.Serialization;

    namespace Dioooooooor.Data
    {
        
        [XmlRootAttribute("Profile", IsNullable = false, Namespace = "Dioooooooor.Data")]    // 当该类为Xml根节点时，以此为根节点名称。
        public class Profile
        {
            [XmlAttribute("功能名")]   // 表现为Xml节点属性。<... 功能名="..."/>                 
            public string Function;

            [XmlArrayAttribute("人物")]    // 表现为Xml层次结构，根为"人物"，其所属的每个该集合节点元素名为类名。<人物><Preson ... /><Preson ... /></人物>
            public List<Preson> Preson = new List<Data.Preson>();
        }

        [XmlRootAttribute("Preson")]
        public class Preson
        {

            [XmlElementAttribute("姓名", IsNullable = false)]      // 表现为Xml节点。<姓名>...</姓名>
            public string Name;

            [XmlElementAttribute("年龄", IsNullable = false)]
            public int Age;

            [XmlElementAttribute("人物描述", IsNullable = false)]
            public string Describe;

            [XmlArrayAttribute("相关人物")]
            public List<AboutPreson> AboutPreson = new List<Data.AboutPreson>();
        }

        [XmlRootAttribute("AboutPreson")]
        public class AboutPreson
        {
            [XmlElementAttribute("姓名", IsNullable = false)]
            public string Name;

            [XmlElementAttribute("人物关系", IsNullable = false)]
            public string Relationship;
        }
    }
    ```
	2. 给XML对象赋值
	``` Csharp
    public Profile GetProfile()
    {
        Profile profile = new Profile();

        var viewport = GameObject.Find("Viewport");
        foreach (Transform transform in viewport.transform)
        {
            Preson preson = new Preson();
            preson.Name = transform.Find("name").Find("InputName").Find("Text").
                GetComponent<UnityEngine.UI.Text>().text;
            preson.Age = int.Parse(transform.Find("age").Find("InputAge").Find("Text").
                GetComponent<UnityEngine.UI.Text>().text);
            preson.Describe = transform.Find("describe").Find("InputDescribe").Find("Text").
                GetComponent<UnityEngine.UI.Text>().text;
            var aboutPresonArray = transform.Find("aboutPreson").Find("InputDescribe").
            Find("Text").GetComponent<UnityEngine.UI.Text>().text.Split('。');
            foreach(var a in aboutPresonArray)
            {
                AboutPreson aboutPreson = new AboutPreson();
                aboutPreson.Name = a.Split('，')[0];
                aboutPreson.Relationship = 
                a.Split('，')[0].Length > 1 ? a.Split('，')[1] : a.Split('，')[0];
                preson.AboutPreson.Add(aboutPreson);
            }
            profile.Preson.Add(preson);
        }

        return profile;
    }
	```
	3. 序列化对象生成XML
	``` Csharp
public void CreateXML()
{
    string path = Application.dataPath + "/XML/GintamaXML_Object.xml";
    Debug.LogError(path);
    if (!File.Exists(path))
    {
        var Profile = DataOperate.GetInstance().GetProfile();
        if (Profile.Preson.Count == 0)
            return;

        using (TextWriter sw = new StreamWriter(path))
        {
            XmlSerializer xz = new XmlSerializer(Profile.GetType());
            xz.Serialize(sw, Profile);
            sw.Close();
        }
    }
}
	```
	4.运行后生成的XML
	```XML
<?xml version="1.0" encoding="utf-8"?>
<Profile xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns="Dioooooooor.Data">
  <人物>
    <Preson>
      <姓名>坂田银时</姓名>
      <年龄>25-29岁</年龄>
      <人物描述>
      坂田银时，日本动漫《银魂》中的主人公。他是活跃在攘夷后期创造无数传奇的革命家，因威震敌我而被称为“白夜叉”。现住歌舞伎町一条街，与神乐、志村新八经营万事屋，是个坚守自己武士道的男子汉。
      </人物描述>
      <相关人物>
        <AboutPreson>
          <姓名>神乐</姓名>
          <人物关系>同事</人物关系>
        </AboutPreson>
        <AboutPreson>
          <姓名>志村新八</姓名>
          <人物关系>同事</人物关系>
        </AboutPreson>
      </相关人物>
    </Preson>
    <Preson>
      <姓名>神乐</姓名>
      <年龄>14岁</年龄>
      <人物描述>
      神乐是日本漫画家空知英秋作品《银魂》 的女主角。是宇宙最强的战斗种族“夜兔族”的一员。 为了打工赚钱而来到江户，但是被流氓利用雇佣，在厌倦了打斗逃出黑社会组织的时候遇到了银时与新八，之后留了在万事屋打工。
      </人物描述>
      <相关人物>
        <AboutPreson>
          <姓名>坂田银时</姓名>
          <人物关系>同事</人物关系>
        </AboutPreson>
        <AboutPreson>
          <姓名>志村新八</姓名>
          <人物关系>同事</人物关系>
        </AboutPreson>
      </相关人物>
    </Preson>
  </人物>
</Profile>
	```

    * 参考
        * [C#对象XML序列化（一）：序列化方法和常用特性](http://www.cnblogs.com/KeithWang/archive/2012/02/22/2363443.html)
        * [XmlSerializer 对象的Xml序列化和反序列化](http://www.cnblogs.com/yukaizhao/archive/2011/07/22/xml-serialization.html)
        
#### 读取XML
1. XmlSerializer反序列化读取

1. XmlDocument
 使用XmlDocument是一种基于文档结构模型的方式来读取XML文件.在XML文件中,我们可以把XML看作是由文档声明(Declare),元素(Element),属性(Attribute),文本(Text)等构成的一个树.最开始的一个结点叫作根结点,每个结点都可以有自己的子结点.得到一个结点后,可以通过一系列属性或方法得到这个结点的值或其它的一些属性.

