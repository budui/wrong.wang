---
title: "Ubuntu关闭GUI"
date: 2020-07-25T10:45:31+08:00
---

## 一、持久关闭

执行以下命令，持久关闭Ubuntu桌面版的GUI环境（通过Ctrl+Alt+F1-F6快捷键进入命令行界面）：

```bash
sudo systemctl set-default multi-user.target
```
执行以下命令，持久开启Ubuntu桌面版的GUI环境（通过Ctrl+Alt+F7快捷键进入GUI界面）：

```bash
sudo systemctl set-default graphical.target
```

## 二、临时关闭

执行以下命令，临时关闭Ubuntu桌面版的GUI环境：

```bash
sudo service lightdm stop
# when ubuntu 20.04
sudo service gdm stop
```

执行以下命令，临时开启Ubuntu桌面版的GUI环境：

```bash
sudo service lightdm start
# when ubuntu 20.04
sudo service gdm start
```