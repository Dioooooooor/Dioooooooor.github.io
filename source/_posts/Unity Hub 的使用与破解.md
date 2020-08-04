---
title: Unity Hub 的使用与破解
date: 2020-05-12 10:04:23
category: Unity
tags:
- UnityHub
- 破解
cover: https://gitee.com/dioooooooor/ImageHosting/raw/master//UnityBlackLarge.png
---

## 安装

下载链接

[https://store.unity.com/cn/download?ref=personal](https://store.unity.com/cn/download?ref=personal)

安装完成之后许可证是未激活状态

![](https://gitee.com/dioooooooor/ImageHosting/raw/master//UnityHub.jpg)

## 破解

1. 安装node，地址

   [https://nodejs.org/zh-cn/](https://nodejs.org/zh-cn/)

2. 安装asar解密工具

   ```powershell
    npm install -g asar
   ```

3. 解密app.asar到app文件夹

   * 进入UnityHub文件夹，cmd调用asar解密，powershell可能会报错

   ```powershell
   asar extract .\app.asar app
   ```

   解密完之后删除app.asar

4. 修改resources\app\src\services\licenseService目录下的**licenseClient.js**

   ```javascript
   getLicenseInfo(callback) {
       // load license
       // get latest data from licenseCore
       //licenseInfo.activated = licenseCore.getLicenseToken().length > 0;//注释这行
       licenseInfo.activated = true;//新增这行
       licenseInfo.flow = licenseCore.getLicenseKind();
   ```

5. 修改resources\app\src\services\licenseService目录下的**licenseCore.js**

   ```javascript
   verifyLicenseData(xml) {
       return new Promise((resolve, reject) => {
           resolve(true);//新增这行
         if (xml === '') {
   ```

6. 完成后再次打开Unity Hub，显示许可证安装成功

   ![](https://gitee.com/dioooooooor/ImageHosting/raw/master//UnityHubRegisted.jpg)

7. 许可证安装成功并不代表通过UnityHub安装的Unity就是专业版，还是要破解安装的Unity