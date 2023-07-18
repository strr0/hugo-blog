---
title: "隐式微分方程"
date: 2023-04-03T16:00:00+08:00
tags: ["differential"]
math: true
draft: false
---

诸如$F(x, y, y') = 0$的微分方程  
①$y = f(x, y')$  
令p = y'，方程两边对x求导  
$p = \frac{\delta f}{\delta x} + \frac{\delta f}{\delta p}\frac{dp}{dx}$  
若$p = \varphi(x, c)$，则$y = f(x, \varphi(x, c))$  
若$x = \psi(p, c)$，则$\begin{cases} x = \psi(p, c) \\\\ y = f(\psi(p, c), p) \end{cases}$  
若$\phi(x, p, c) = 0$，则$\begin{cases} \phi(x, p, c) = 0 \\\\ y = f(x, p) \end{cases}$  
②$x = f(y, y')$  
令p = y'，方程两边对y求导  
$\frac{1}{p} = \frac{\delta f}{\delta y} + \frac{\delta f}{\delta p}\frac{dp}{dy}$  
若$p = \varphi(y, c)$，则$x = f(y, \varphi(y, c))$  
若$y = \psi(p, c)$，则$\begin{cases} y = \psi(p, c) \\\\ x = f(\psi(p, c), p) \end{cases}$  
若$\phi(y, p, c) = 0$，则$\begin{cases} \phi(y, p, c) = 0 \\\\ x = f(y, p) \end{cases}$  
需考虑p = 0情况  
③$F(x, y') = 0$  
设$y' = p = \psi(t)$，代入原式得$x = \varphi(t)$  
$dy = pdx = \psi(t)\varphi'(t)dt$  
解得$\begin{cases} x = \varphi(t) \\\\ y = \int \psi(t)\varphi'(t)dt + C \end{cases}$  
④$F(y, y') = 0$  
设$y' = p = \psi(t)$，代入原式得$y = \varphi(t)$  
$dx = \frac{1}{p}dy = \frac{\varphi'(t)}{\psi(t)}dt$  
解得$\begin{cases} y = \varphi(t) \\\\ x = \int \frac{\varphi'(t)}{\psi(t)}dt + C \end{cases}$  
需考虑p = 0情况  