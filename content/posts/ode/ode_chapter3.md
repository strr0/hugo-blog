---
title: "线性非齐次微分方程"
date: 2023-03-30T10:00:00+08:00
tags: ["differential"]
math: true
draft: false
---

1. $\frac{dy}{dx} = P(x)y + Q(x)$①  
假设Q(x) = 0，则$\frac{dy}{dx} = P(x)y$为线性齐次微分方程  
解得$y = Ce^{\int P(x)dx}$  
设原式解为$y = c(x)e^{\int P(x)dx}$②  
$\frac{dy}{dx} = \frac{dc(x)}{dx}e^{\int P(x)dx} + c(x)P(x)e^{\int P(x)dx}$③  
联立①②③可得$Q(x) = \frac{dc(x)}{dx}e^{\int P(x)dx}$  
所以$c(x) = \int Q(x)e^{-\int P(x)dx}dx + C$  
$y = e^{\int P(x)dx}(\int Q(x)e^{-\int P(x)dx}dx + C)$  

2. $\frac{dy}{dx} = P(x)y + Q(x)y^n$  
两边同时除以$y^n$  
$y^{-n} \frac{dy}{dx} = P(x)y^{1-n} + Q(x)$  
令$z = y^{1-n}$，$\frac{dz}{dx} = (1-n)y^{-n}\frac{dz}{dx} = (1-n)P(x)z + (1-n)Q(x)$  