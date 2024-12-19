---
title: "二阶行列式"
date: 2023-03-31T17:30:00+08:00
categories: ["algebra"]
math: true
draft: false
---

### 二元线性方程组

$\begin{cases} a_{11}x_{1} + a_{12}x_{2} = b_{1} ①\\\\ a_{21}x_{1} + a_{22}x_{2} = b_{2} ②\end{cases}$

① × $a_{22}$ - ② × $a_{12}$得

$x_{1} = \frac{\begin{vmatrix} b_{1} & a_{12} \\\\ b_{2} & a_{22} \end{vmatrix}}{\begin{vmatrix} a_{11} & a_{12} \\\\ a_{21} & a_{22} \end{vmatrix}}$

同理可得

$x_{2} = \frac{\begin{vmatrix} a_{11} & b_{1} \\\\ a_{21} & b_{2} \end{vmatrix}}{\begin{vmatrix} a_{11} & a_{12} \\\\ a_{21} & a_{22} \end{vmatrix}}$

### 三元线性方程组

$\begin{cases} a_{11}x_{1} + a_{12}x_{2} + a_{13}x_{3} = b_{1} ③ \\\\ a_{21}x_{1} + a_{22}x_{2} + a_{23}x_{3} = b_{2} ④ \\\\ a_{31}x_{1} + a_{32}x_{2} + a_{33}x_{3} = b_{3} ⑤ \end{cases}$

假设③ × u + ④ × v + ⑤ × w可消去$x_{2} x_{3}$，则

$\begin{cases} a_{11}ux_{1} + a_{21}vx_{1} + a_{31}wx_{1} = b_{1}u + b_{2}v + b_{3}w ⑥ \\\\ a_{12}u + a_{22}v + a_{32}w = 0 ⑦ \\\\ a_{13}u + a_{23}v + a_{33}w = 0 ⑧ \end{cases}$

提取⑦⑧

$\begin{cases} a_{12}\frac{u}{w} + a_{22}\frac{v}{w} = -a_{32} \\\\ a_{13}\frac{u}{w} + a_{23}\frac{v}{w} = -a_{33} \end{cases}$

解得

$\frac{u}{w} = \frac{\begin{vmatrix} -a_{32} & a_{22} \\\\ -a_{33} & a_{23} \end{vmatrix}}{\begin{vmatrix} a_{12} & a_{22} \\\\ a_{13} & a_{23} \end{vmatrix}} = \frac{\begin{vmatrix} a_{22} & a_{23} \\\\ a_{32} & a_{33} \end{vmatrix}}{\begin{vmatrix} a_{12} & a_{13} \\\\ a_{22} & a_{23} \end{vmatrix}}$

$\frac{v}{w} = \frac{\begin{vmatrix} a_{12} & -a_{32} \\\\ a_{13} & -a_{33} \end{vmatrix}}{\begin{vmatrix} a_{12} & a_{22} \\\\ a_{13} & a_{23} \end{vmatrix}} = -\frac{\begin{vmatrix} a_{12} & a_{13} \\\\ a_{32} & a_{33} \end{vmatrix}}{\begin{vmatrix} a_{12} & a_{13} \\\\ a_{22} & a_{23} \end{vmatrix}}$

令$u = \begin{vmatrix} a_{22} & a_{23} \\\\ a_{32} & a_{33} \end{vmatrix}$，$v = -\begin{vmatrix} a_{12} & a_{13} \\\\ a_{32} & a_{33} \end{vmatrix}$，$w = \begin{vmatrix} a_{12} & a_{13} \\\\ a_{22} & a_{23} \end{vmatrix}$

代入⑥得

$a_{11}\begin{vmatrix} a_{22} & a_{23} \\\\ a_{32} & a_{33} \end{vmatrix}x_{1} - a_{21}\begin{vmatrix} a_{12} & a_{13} \\\\ a_{32} & a_{33} \end{vmatrix}x_{1} + a_{31}\begin{vmatrix} a_{12} & a_{13} \\\\ a_{22} & a_{23} \end{vmatrix}x_{1} = b_{1}\begin{vmatrix} a_{22} & a_{23} \\\\ a_{32} & a_{33} \end{vmatrix} - b_{2}\begin{vmatrix} a_{12} & a_{13} \\\\ a_{32} & a_{33} \end{vmatrix} + b_{3}\begin{vmatrix} a_{12} & a_{13} \\\\ a_{22} & a_{23} \end{vmatrix}$

$\begin{vmatrix} a_{11} & a_{12} & a_{13} \\\\ a_{21} & a_{22} & a_{23} \\\\ a_{31} & a_{32} & a_{33} \end{vmatrix} x_{1} = \begin{vmatrix} b_{1} & a_{12} & a_{13} \\\\ b_{2} & a_{22} & a_{23} \\\\ b_{3} & a_{32} & a_{33} \end{vmatrix}$

解得

$x_{1} = \frac{\begin{vmatrix} b_{1} & a_{12} & a_{13} \\\\ b_{2} & a_{22} & a_{23} \\\\ b_{3} & a_{32} & a_{33} \end{vmatrix}}{\begin{vmatrix} a_{11} & a_{12} & a_{13} \\\\ a_{21} & a_{22} & a_{23} \\\\ a_{31} & a_{32} & a_{33} \end{vmatrix}}$

同理可得

$x_{2} = \frac{\begin{vmatrix} a_{11} & b_{1} & a_{13} \\\\ a_{21} & b_{2} & a_{23} \\\\ a_{31} & b_{3} & a_{33} \end{vmatrix}}{\begin{vmatrix} a_{11} & a_{12} & a_{13} \\\\ a_{21} & a_{22} & a_{23} \\\\ a_{31} & a_{32} & a_{33} \end{vmatrix}}$，$x_{3} = \frac{\begin{vmatrix} a_{11} & a_{12} & b_{1} \\\\ a_{21} & a_{22} & b_{2} \\\\ a_{31} & a_{32} & b_{3} \end{vmatrix}}{\begin{vmatrix} a_{11} & a_{12} & a_{13} \\\\ a_{21} & a_{22} & a_{23} \\\\ a_{31} & a_{32} & a_{33} \end{vmatrix}}$