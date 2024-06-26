---
layout: default
title: 在 VS Code 中调试 UE
description: 介绍如何在 ubuntu 系统环境下搭建 UE 调试环境
---

Ubuntu 22.04.4、UE5.3.1

1.打开编辑器，创建一个 C++ 工程项目：

<p align="center">
  <img src="../../images/DebugUEInVSCode-1-CreateProject.png">
</p>

选择一个模版工程，以 Third Person 为例，默认创建【BLUEPRINT】蓝图工程，点击【C++】改为创建 C++ 工程。点击【Create】后成功创建工程，编辑器退出。若已安装 vscode，在创建工程成功后会自动打开 vscode 软件，vscode 首次打开 UE 工程项目会提示安装一些扩展插件和 .NET8 等工具。

2.再次用编辑器打开创建的 C++ 工程项目，配置打包时二进制程序构建类型为【Debug】：

<p align="center">
  <img src="../../images/DebugUEInVSCode-2-BinaryConfiguration.png">
</p>

3.点击【PackageProject】开始打包：

<p align="center">
  <img src="../../images/DebugUEInVSCode-3-PackageProject.png">
</p>

4.选择打包路径为项目根目录：

<p align="center">
  <img src="../../images/DebugUEInVSCode-4-SetPackagePath.png">
</p>

打包成功后将在项目根目录下生成一个【Linux】文件夹，存放的是打包后的项目各种资产、配置和程序文件：

<p align="center">
  <img src="../../images/DebugUEInVSCode-5-PackagePath.png">
</p>

项目应用程序二进制文件生成位置是：

<p align="center">
  <img src="../../images/DebugUEInVSCode-6-BinaryPath.png">
</p>


5.在 vscode 中更改【Run and Debug】程序。在项目工程管理文件中将 Launch program 替换成我们上一步打包后生成的程序：

<p align="center">
  <img src="../../images/DebugUEInVSCode-7-BeforeSetLaunch.png">
</p>

具体来说就是将上图中标示程序替换成下图中标示程序：

<p align="center">
  <img src="../../images/DebugUEInVSCode-8-AfterSetLaunch.png">
</p>

因为我们在UE中配置的打包构建类型是 Debug，所以在这里我们修改的也是 Launch ThirdPerson (Debug) 这一项的 program。

6.在 vscode 中选择【Run and Debug】程序。选择我们刚刚配置的 Launch ThirdPerson (Debug) 开始调试。

<p align="center">
  <img src="../../images/DebugUEInVSCode-9-SelectLaunchProgram.png">
</p>

[back](./../)