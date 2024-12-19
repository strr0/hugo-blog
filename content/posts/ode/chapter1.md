---
title: "一阶微分方程的初等解法"
date: 2023-03-28T20:00:00+08:00
categories: ["differential"]
math: true
draft: false
---

## 1 常见不定积分

$\int x^a dx = \frac{1}{a+1}x^{a+1} + C$

$\int a^x dx = \frac{1}{\ln{a}}a^x + C$

$\int \frac{1}{x} dx = \ln{|x|} + C$

$\int e^x dx = e^x + C$

$\int \cos{x} dx = \sin{x} + C$

$\int \sin{x} dx = -\cos{x} + C$

$\int \frac{1}{\sqrt{1 - x^2}} dx = \arcsin{x} + C$

$\int \frac{1}{1 + x^2} dx = \arctan{x} + C$

$\int {\sec}^2{x} dx = \tan{x} + C$

$\int {\csc}^2{x} dx = -\cot{x} + C$

$\int \tan{x} dx = -\ln{|\cos{x}|} + C$

$\int \cot{x} dx = \ln{|\sin{x}|} + C$

$\int \sec{x} dx = \ln{|\sec{x} + \tan{x}|} + C$

$\int \csc{x} dx = \ln{|\csc{x} - \cot{x}|} + C$

$\int \frac{1}{x^2+a^2} dx = \frac{1}{a}\arctan{\frac{x}{a}} + C$

$\int \frac{1}{x^2-a^2} dx = \frac{1}{2a}\ln{|\frac{x-a}{x+a}|} + C$

## 2 变量分离和变量变换

### 2.1 变量分离

$\frac{dy}{dx} = f(x)\varphi(y)$

$\frac{dy}{\varphi(y)} = f(x)dx$

$\int \frac{dy}{\varphi(y)} = \int f(x)dx + C$

### 2.2 变量变换

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

⑤$\begin{vmatrix} a_{1} & b_{1} \\\\ a_{2} & b_{2} \end{vmatrix} \neq 0$

$\begin{cases} a_{1}x + b_{1}y + c_{1} = 0 \\\\ a_{2}x + b_{2}y + c_{2} = 0 \end{cases}$ 解得$\begin{cases} x = \alpha \\\\ y = \beta \end{cases}$

令$\begin{cases} X = x - \alpha \\\\ Y = y - \beta \end{cases}$有$\begin{cases} a_{1}X + b_{1}Y = 0 \\\\ a_{2}X + b_{2}Y = 0 \end{cases}$

$\frac{dY}{dX} = f(\frac{a_{1}X + b_{1}Y}{a_{2}X + b_{2}Y})$

## 3 线性非齐次微分方程

### 3.1 $\frac{dy}{dx} = P(x)y + Q(x)$①

假设Q(x) = 0，则$\frac{dy}{dx} = P(x)y$为线性齐次微分方程

解得$y = Ce^{\int P(x)dx}$

设原式解为$y = c(x)e^{\int P(x)dx}$②

$\frac{dy}{dx} = \frac{dc(x)}{dx}e^{\int P(x)dx} + c(x)P(x)e^{\int P(x)dx}$③

联立①②③可得$Q(x) = \frac{dc(x)}{dx}e^{\int P(x)dx}$

所以$c(x) = \int Q(x)e^{-\int P(x)dx}dx + C$

$y = e^{\int P(x)dx}(\int Q(x)e^{-\int P(x)dx}dx + C)$

### 3.2 $\frac{dy}{dx} = P(x)y + Q(x)y^n$

两边同时除以$y^n$

$y^{-n} \frac{dy}{dx} = P(x)y^{1-n} + Q(x)$

令$z = y^{1-n}$，$\frac{dz}{dx} = (1-n)y^{-n}\frac{dz}{dx} = (1-n)P(x)z + (1-n)Q(x)$

## 4 恰当微分方程

$M(x, y)dx + N(x, y)dy = 0$

上式为恰当微分方程的充要条件是$\frac{\delta M(x, y)}{\delta y} = \frac{\delta N(x, y)}{\delta x}$

### 4.1 充要性

#### 必要性

$\because M(x, y)dx + N(x, y)dy = 0$ 为微分方程

$\therefore M(x, y) = \frac{\delta u(x, y)}{\delta x}$①

$N(x, y) = \frac{\delta u(x, y)}{\delta y}$

$\therefore \frac{\delta M(x, y)}{\delta y} = \frac{\delta ^ 2 u(x, y)}{\delta y \delta x} = \frac{\delta ^ 2 u(x, y)}{\delta x \delta y} = \frac{\delta N(x, y)}{\delta x}$

#### 充分性

对①积分

$\int M(x, y) dx + \phi(y) = u(x, y)$

再对y求偏导

$\frac{\delta \int M(x, y) dx}{\delta y} + \frac{\delta \phi(y)}{\delta y} = N(x, y)$

$\frac{\delta \phi(y)}{\delta y} = N(x, y) - \frac{\delta \int M(x, y) dx}{\delta y}$

两边对x求偏导

$\frac{\delta^2 \phi(y)}{\delta x \delta y} = \frac{\delta N(x, y)}{\delta x} - \frac{\delta^2 \int M(x, y) dx}{\delta x \delta y}$

$\therefore 0 = \frac{\delta N(x, y)}{\delta x} - \frac{\delta M(x, y)}{\delta y}$

### 4.2 恰当微分方程解法

#### 方法一：利用恰当微分方程判定求解

$u(x, y) = \int M(x, y) dx + \int (N(x, y) - \frac{\delta \int M(x, y) dx}{\delta y}) dy$

#### 方法二：利用曲线积分求解

$u(x, y) = \int_{x_{0}}^{x} M(x, y) dx + \int_{y_{0}}^{y} N(x, y) dy$

#### 方法三：分项组合方法

$ydx + xdy = d(xy)$

$\frac{ydx - xdy}{y^2} = d(\frac{x}{y})$

$\frac{xdy - ydx}{x^2} = d(\frac{y}{x})$

$\frac{ydx - xdy}{xy} = d(\ln |\frac{x}{y}|)$

$\frac{ydx - xdy}{x^2 + y^2} = d(arccot \frac{y}{x})$

$\frac{ydx - xdy}{x^2 - y^2} = \frac{1}{2}d(\ln |\frac{x-y}{x+y}|)$

#### 积分因子

对非恰当微分方程$M(x, y)dx + N(x, y)dy = 0$两边同时乘以$\mu$使得原方程转化为恰当微分方程$\mu M(x, y)dx + \mu N(x, y)dy = 0$

$\frac{\delta \mu M(x, y)}{\delta y} = \frac{\delta \mu N(x, y)}{\delta x}$

$M(x, y)\frac{\delta \mu}{\delta y} + \mu\frac{\delta M(x, y)}{\delta y} = N(x, y)\frac{\delta \mu}{\delta x} + \mu\frac{\delta N(x, y)}{\delta x}$

$N(x, y)\frac{\delta \mu}{\delta x} - M(x, y)\frac{\delta \mu}{\delta y} = \mu(\frac{\delta M(x, y)}{\delta y} - \frac{\delta N(x, y)}{\delta x})$

①$\mu$与y无关

$N(x, y)\frac{d\mu}{dx} = \mu(\frac{\delta M(x, y)}{\delta y} - \frac{\delta N(x, y)}{\delta x})$

$\frac{1}{\mu}d\mu = \frac{\frac{\delta M(x, y)}{\delta y} - \frac{\delta N(x, y)}{\delta x}}{N(x, y)}dx$

$\mu = Ce^{\int \frac{\frac{\delta M(x, y)}{\delta y} - \frac{\delta N(x, y)}{\delta x}}{N(x, y)}dx}$

②$\mu$与x无关

$-M(x, y)\frac{d\mu}{dy} = \mu(\frac{\delta M(x, y)}{\delta y} - \frac{\delta N(x, y)}{\delta x})$

$\frac{1}{\mu}d\mu = -\frac{\frac{\delta M(x, y)}{\delta y} - \frac{\delta N(x, y)}{\delta x}}{M(x, y)}dy$

$\mu = Ce^{-\int \frac{\frac{\delta M(x, y)}{\delta y} - \frac{\delta N(x, y)}{\delta x}}{M(x, y)}dy}$

## 5 隐式微分方程

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