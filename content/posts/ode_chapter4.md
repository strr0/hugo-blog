---
title: "恰当微分方程"
date: 2023-03-30T19:00:00+08:00
tags: ["differential"]
math: true
draft: false
---

$M(x, y)dx + N(x, y)dy = 0$  
上式为恰当微分方程的充要条件是$\frac{\delta M(x, y)}{\delta y} = \frac{\delta N(x, y)}{\delta x}$  
必要性  
$\because M(x, y)dx + N(x, y)dy = 0$ 为微分方程  
$\therefore M(x, y) = \frac{\delta u(x, y)}{\delta x}$①  
$N(x, y) = \frac{\delta u(x, y)}{\delta y}$  
$\therefore \frac{\delta M(x, y)}{\delta y} = \frac{\delta ^ 2 u(x, y)}{\delta y \delta x} = \frac{\delta ^ 2 u(x, y)}{\delta x \delta y} = \frac{\delta N(x, y)}{\delta x}$  
充分性  
对①积分  
$\int M(x, y) dx + \phi(y) = u(x, y)$  
再对y求偏导  
$\frac{\delta \int M(x, y) dx}{\delta y} + \frac{\delta \phi(y)}{\delta y} = N(x, y)$  
$\frac{\delta \phi(y)}{\delta y} = N(x, y) - \frac{\delta \int M(x, y) dx}{\delta y}$  
两边对x求偏导  
$\frac{\delta^2 \phi(y)}{\delta x \delta y} = \frac{\delta N(x, y)}{\delta x} - \frac{\delta^2 \int M(x, y) dx}{\delta x \delta y}$  
$\therefore 0 = \frac{\delta N(x, y)}{\delta x} - \frac{\delta M(x, y)}{\delta y}$  

恰当微分方程解法：  
方法一：利用恰当微分方程判定求解  
$u(x, y) = \int M(x, y) dx + \int (N(x, y) - \frac{\delta \int M(x, y) dx}{\delta y}) dy$  
方法二：利用曲线积分求解  
$u(x, y) = \int_{x_{0}}^{x} M(x, y) dx + \int_{y_{0}}^{y} N(x, y) dy$  
方法三：分项组合方法  
1. $ydx + xdy = d(xy)$  
2. $\frac{ydx - xdy}{y^2} = d(\frac{x}{y})$  
3. $\frac{xdy - ydx}{x^2} = d(\frac{y}{x})$  
4. $\frac{ydx - xdy}{xy} = d(\ln |\frac{x}{y}|)$  
5. $\frac{ydx - xdy}{x^2 + y^2} = d(arccot \frac{y}{x})$  
6. $\frac{ydx - xdy}{x^2 - y^2} = \frac{1}{2}d(\ln |\frac{x-y}{x+y}|)$  