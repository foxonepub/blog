---
title: We Just Made Fox.ONE Webview 30% Faster
tags: ["Blockchain", "Cryptocurrency", "Webview", "Electron"]
date: 2018-10-18 00:35:28
---

We released [Fox.ONE](https://www.fox.one/) 1.5.1 Desktop 6 months ago. In that release, the speed of opening web pages has been greatly improved.

We measured the time for DOM loading and page loading in different network conditions:

![img](https://cdn-images-1.medium.com/max/1000/0*WHV0Qv58_zS26ljB)



![img](https://cdn-images-1.medium.com/max/1000/0*n5sNndjSNF6dllBs)

Obviously, both DOM loading and the page loading have improved a lot(from 9% to 31%). How did we do it?

## Webview Issues	

The desktop version of Fox.ONE built upon Electron, Electron builder and Vue.JS. Electron is the framework for creating desktop apps with Web technologies, while Electron builder handles packaging, code-sign, cross-platform CI, and auto-updating.

We chose Electron because the Web technology provides a lot of convenience for development, but due to the Electron project’s reliance on Chromium, some problems upstream of Chromium were seamlessly translated to Electron. The webview is a big problem.

Webview can be considered as a secure iframe running in a separate process. If we embed a web page in the Electron App (instead of opening it in a new window), we can put it in the webview.

At first, Fox.ONE used webview and everything seemed to be good. But soon we found the problems:

1. Although webview runs in a separate process, it is in the same the Renderer process as DOM structure. So rendering the webview affects the entire Renderer process.
2. There are some issues ([1](https://github.com/electron/electron/issues/6139), [2](https://github.com/electron/electron/issues/5110), [3](https://github.com/electron/electron/issues/8505)) in the webview, we can’t solve these problems, and the Electron team can’t too — even Chromium team have no plan to solve it.

In the end, we decided to use **browserview** instead of **webview**.

## Browserview vs Webview

The biggest difference is that browserview is hosted in the main process instead of the renderer process. It is very similar to how pages are handled in Google Chrome, and it’s the way you get a high page responsiveness.

Since the GUI is divided into two processes, we have to deal with the special layout system for browserview.



![webview vs browserview apprach](https://cdn-images-1.medium.com/max/1000/0*pesjz8qBNb3LjaC_)


## Problem with Browserview

we found that two problems with browserview:

1. **Browserview** lacks a rich API set like webview. In browserview, you won’t be able to use plugins, preload scripts, screenshots, etc.
2. **Browserview** is not in the renderer process, so you can’t use CSS to control its layout.

For the first point, we directly manipulate webContents to access some methods and properties missed in browserview. For the second point, we implemented a special browserview manager to control the external appearance of the browserview.

## Use Browserview

As mentioned above, we implemented a browserview manager to manage all browserviews. The manager uses **ipcMain** and **ipcRenderer** to establish communication.

When the user performs operations on the client (such as forward, rewind, refresh, switch pages, etc.), the instructions are sent to the **browserview manager** via the Electron event mechanism, and then the manager operates the **browserview** and the **webContents** to execute the instructions.



![img](https://cdn-images-1.medium.com/max/1000/0*_5DchFCX9uoW-bjn)

## Conclusion

Although **browserview** is still an experimental feature in Electron, the API is not complete and lacks the mechanisms like webviews, if you need to embed an external web page in the app, using browserview is a good choice under some trade-offs.

## Get New Fox.ONE!

New Fox.ONE is available on Mac, iPhone, Android, macOS, Windows, and Linux.

Check out the official website: [https://www.fox.one](https://www.fox.one/) and get it.
