---
title: 数据读取之PlayerPrefs
data: <strong> 2017.9.15 </strong>
categories:
- Unity
tags: 
- 数据读取
- PlayerPrefs
cover: https://raw.githubusercontent.com/Dioooooooor/BlogImageHosting/master/image/UnityBlackLarge.png
---
<p style="text-indent:2em">在Unity游戏中，关于游戏的存档，读档，最高分的记录，排行榜的生成...，这些都离不开数据的存储和读取，在Unity中，使用最多的数据存储包括了，Unity自带的PlayerPrefs，XML,Json，数据库等，本章我们介绍的就是Unity自带的PlayerPrefs存储方式。</p>

<!-- more -->

## 官方描述

Stores and accesses player preferences between game sessions.

* Editor/Standalone 

On macOS PlayerPrefs are stored in ~/Library/Preferences folder, in a file named unity.[company name].[product name].plist, where company and product names are the names set up in Project Settings. The same .plist file is used for both Projects run in the Editor and standalone players.

On Windows, PlayerPrefs are stored in the registry under HKCU\Software\[company name]\[product name] key, where company and product names are the names set up in Project Settings.

On Linux, PlayerPrefs can be found in ~/.config/unity3d/[CompanyName]/[ProductName] again using the company and product names specified in the Project Settings.

On Windows Store Apps, Player Prefs can be found in %userprofile%\AppData\Local\Packages\[ProductPackageId]>\LocalState\playerprefs.dat

On Windows Phone 8, Player Prefs can be found in application's local folder, See Also: Directory.localFolder

On Android data is stored (persisted) on the device. The data is saved in SharedPreferences. C#/JavaScript, Android Java and Native code can all access the PlayerPrefs data. The PlayerPrefs data is physically stored in /data/data/pkg-name/shared_prefs/pkg-name.xml.

On WebGL, PlayerPrefs are stored using the browser's IndexedDB API.

在游戏会话之间存储和访问玩家喜好。

* 编辑/单机

1. 在macOS上，PlayerPref存储在〜/ Library / Preferences文件夹中，名为unity。[company name]。[product name] .plist，其中公司和产品名称是在“项目设置”中设置的名称。相同的.plist文件用于在编辑器和独立播放器中运行的项目。

2. 在Windows上，PlayerPref存储在HKCU \ Software \ [公司名称] \ [产品名称]键下的注册表中，其中公司和产品名称是“项目设置”中设置的名称。

3. 在Linux上，可以使用“项目设置”中指定的公司和产品名称，再次在〜/ .config / unity3d / [CompanyName] / [ProductName]中找到PlayerPrefs。

4. 在Windows Store Apps中，可以在％userprofile％\ AppData \ Local \ Packages \ [ProductPackageId]> \ LocalState \ playerprefs.dat中找到Player Prefs

5. 在Windows Phone 8上，可以在应用程序的本地文件夹中找到Player Prefs，另请参见：Directory.localFolder

6. 在Android数据被存储（持久）在设备上。数据保存在SharedPreferences中。 C＃/ JavaScript，Android Java和Native代码都可以访问PlayerPrefs数据。 PlayerPrefs数据实际存储在/data/data/pkg-name/shared_prefs/pkg-name.xml中。

7. 在WebGL上，PlayerPrefs使用浏览器的IndexedDB API进行存储。

## 用法介绍

* 静态方法

| 方法                                                                        | 作用                                                                  |
| ------------------                                                         | -------------------------------------:                                |
| **public static void DeleteAll();**                                        | 从首选项中删除所有键和值(谨慎使用)                                       |
| **public static void DeleteKey(string key)**                               | 从首选项中删除键及其相应的值                                             |
| **public static float GetFloat(string key, float defaultValue = 0.0F)**    | 返回与首选项文件中的键对应的值（如果存在）如果不存在，它将返回defaultValue  |
| **public static int GetInt(string key, int defaultValue = 0)**             | 返回与首选项文件中的键对应的值（如果存在）如果不存在，它将返回defaultValue  |
| **public static string GetString(string key, string defaultValue = "")**   | 返回与首选项文件中的键对应的值（如果存在）如果不存在，它将返回defaultValue  |
| **HasKey**                                                                 | 如果在首选项中存在键，则返回true                                         |
| **Save**                                                                   | 将所有修改的首选项写入磁盘                                               |
| **public static void SetFloat(string key, float value)**                   | 设置由键标识的首选项的值                                                 |
| **public static void SetInt(string key, int value)**                       | 设置由键标识的首选项的值                                                 |
| **public static void SetString(string key, string value)**                 | 设置由键标识的首选项的值                                                 |
<font color = red >**注: 首选项(Preferences)指Unity的参数设置面板** </font>

## 方法使用
* 由于这个方法比较简单，所以这里引用[**yimingsilence博客**](http://blog.csdn.net/yimingsilence/article/details/43452471)中的内容，使用两个脚本，模拟函数的使用情况

    * 脚本一：
    ```Csharp
    using UnityEngine;
    using System.Collections;

    public class Scene1Script : MonoBehaviour 
    {  
        //姓名
        private string mName="路人甲";
        //年龄
        private int mAge=20;
        //成绩
        private float mGrade=75.5F;

        void OnGUI()
        {
            GUILayout.Label("Unity3D数据存储示例程序",GUILayout.Height(25));
            //姓名
            GUILayout.Label("请输入姓名：",GUILayout.Height(25));
            mName=GUILayout.TextField(mName,GUILayout.Height(25));
            //年龄
            GUILayout.Label("请输入年龄：",GUILayout.Height(25));
            mAge=int.Parse(GUILayout.TextField(mAge.ToString(),GUILayout.Height(25)));
            //成绩
            GUILayout.Label("请输入成绩：",GUILayout.Height(25));
            mGrade=float.Parse(GUILayout.TextField(mGrade.ToString(),GUILayout.Height(25)));
            
            //提交数据
            if(GUILayout.Button("提交数据",GUILayout.Height(25)))
            {
            //保存数据
            PlayerPrefs.SetString("Name",mName);
            PlayerPrefs.SetInt("Age",mAge);
            PlayerPrefs.SetFloat("Grade",mGrade);
            
            //切换到新场景
            Application.LoadLevel("Scene1");
            }
        }
    }
    ```

    * 脚本二：
    ```Csharp
    using UnityEngine;
    using System.Collections;

    public class Scene2Script : MonoBehaviour 
    {

        private string mName;
        private int mAge;
        private float mGrade;
        
        void Start () 
        {
            //读取数据
            mName=PlayerPrefs.GetString("Name","DefaultValue");
            mAge=PlayerPrefs.GetInt("Age",0);
            mGrade=PlayerPrefs.GetFloat("Grade",0F);
        }
        
        void OnGUI()
        {
            GUILayout.Label("Unity3D数据存储示例程序",GUILayout.Height(25));
            //姓名
            GUILayout.Label("姓名："+mName,GUILayout.Height(25));
            //年龄
            GUILayout.Label("年龄："+mAge,GUILayout.Height(25));
            //成绩
            GUILayout.Label("成绩："+mGrade,GUILayout.Height(25));
            
            //删除数据
            if(GUILayout.Button("清除数据",GUILayout.Height(25)))
            {
                PlayerPrefs.DeleteAll();
            }
            
            //返回Scene0
            if(GUILayout.Button("返回场景",GUILayout.Height(25)))
            {
                Application.LoadLevel("Scene0");
            }
            
        }
    }
    ```