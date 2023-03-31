---
title: "变量分离和变量变换"
date: 2023-03-29T20:00:00+08:00
tags: ["differential"]
math: true
draft: false
---

1. 变量分离  
$\frac{dy}{dx} = f(x)\varphi(y)$  
$\frac{dy}{\varphi(y)} = f(x)dx$  
$\int \frac{dy}{\varphi(y)} = \int f(x)dx + C$  
2. 变量变换  
形如$\frac{dy}{dx} = g(\frac{y}{x})$  
令$u = \frac{y}{x}$，则$\frac{dy}{dx} = \frac{xdu}{dx} + u = g(u)$  
$\frac{du}{g(u) - u} = \frac{dx}{x}$  
$\int \frac{du}{g(u) - u} = \int \frac{dx}{x} + C$  
形如$\frac{dy}{dx} = f(ax + by + c)$  
令u = ax + by + c，则$\frac{du}{dx} = a + \frac{bdy}{dx} = a + bf(u)$  
形如$\frac{dy}{dx} = f(\frac{a_{1}x + b_{1}y + c_{1}}{a_{2}x + b_{2}y + c_{2}})$  
①$c_{1} = 0$且$c_{2} = 0$  
$\frac{dy}{dx} = f(\frac{a_{1} + \frac{b_{1}y}{x}}{a_{2} + \frac{b_{2}y}{x}})$  
②$a_{1} = 0$且$a_{2} = 0$  
$\frac{dy}{dx} = f(\frac{b_{1}y + c_{1})}{b_{2}y + c_{2}})$  
③$b_{1} = 0$且$b_{2} = 0$  
$\frac{dy}{dx} = f(\frac{a_{1}x + c_{1}}{a_{2}x + c_{2}})$  
④$a_{1}x + b_{1}y = k(a_{2}x + b_{2}y)$  
令$u = a_{1}x + b_{1}y$，则$\frac{du}{dx} = a_{1} + b_{1}\frac{dy}{dx} = a_{1} + b_{1}f(\frac{u + c_{1}}{ku + c_{2}})$  
⑤$\begin{vmatrix} a_{1} & b_{1} \\ a_{2} & b_{2} \end{vmatrix} \neq 0$  
$\begin{cases} a_{1}x + b_{1}y + c_{1} = 0 \\ a_{2}x + b_{2}y + c_{2} = 0 \end{cases}$ 解得$\begin{cases} x = \alpha \\ y = \beta \end{cases}$  
令$\begin{cases} X = x - \alpha \\ Y = y - \beta \end{cases}$有$\begin{cases} a_{1}X + b_{1}Y = 0 \\ a_{2}X + b_{2}Y = 0 \end{cases}$  
$\frac{dY}{dX} = f(\frac{a_{1}X + b_{1}Y}{a_{2}X + b_{2}Y})$  