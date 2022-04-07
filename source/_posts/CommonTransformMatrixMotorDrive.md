---
title: some common transform matrices for the motor drive
comment: true
mathjax: true
date: 2022-04-07 21:46:58
description: This article is about the common transform matrices for the motor drive. It help me quickly look up useful transform matrices. These transformers is based on the principle of equal amplitude. These matrices is frequently used in power electronics. It contains the classical three phase machine and prospective  dual three phase machine.
tags:
  - Motor Drive
  - Power Electronics
categories:
  - Power Electronics

---

# preface

$$
\theta = \omega t + \phi
$$

- $\theta$ angular position
- $\omega$ angular frequency
- $\phi$ initial phase
 
# three phase machine  

## $abc \longrightarrow dq0$

$$
\begin{equation}
	\begin{bmatrix}
		d\\
		q\\
		0
	\end{bmatrix}
	=\frac{2}{3}
	\begin{bmatrix}
		\cos(\theta) & \cos(\theta-\frac{2\pi}{3}) & \cos(\theta+\frac{2\pi}{3})\\
		-\sin(\theta) & -\sin(\theta-\frac{2\pi}{3}) & -\sin(\theta+\frac{2\pi}{3})\\
		\frac{1}{2} & \frac{1}{2} & \frac{1}{2}
	\end{bmatrix}
	\begin{bmatrix}
		a\\
		b\\
		c
	\end{bmatrix}
\end{equation}
$$

## $dq0 \longrightarrow abc$

$$
\begin{equation}
	\begin{bmatrix}
		a\\
		b\\
		c
	\end{bmatrix}
	=\frac{2}{3}
	\begin{bmatrix}
		\cos(\theta) & -\sin(\theta) & 1\\
		\cos(\theta-\frac{2\pi}{3}) & -\sin(\theta-\frac{2\pi}{3}) & 1\\
		\cos(\theta+\frac{2\pi}{3}) & -\sin(\theta+\frac{2\pi}{3}) & 1
	\end{bmatrix}
	\begin{bmatrix}
		d\\
		q\\
		0
	\end{bmatrix}
\end{equation}
$$

## $abc \longrightarrow \alpha \beta 0$

$$
\begin{equation}
	\begin{bmatrix}
		\alpha\\
		\beta\\
		0
	\end{bmatrix}
	=\frac{2}{3}
	\begin{bmatrix}
		\cos(0) & \cos(-\frac{2\pi}{3}) & \cos(\frac{2\pi}{3})\\
		-\sin(0) & -\sin(-\frac{2\pi}{3} & -\sin(\frac{2\pi}{3})\\
		\frac{1}{2} & \frac{1}{2} & \frac{1}{2}
	\end{bmatrix}
	\begin{bmatrix}
		a\\
		b\\
		c
	\end{bmatrix}
	=\frac{2}{3}
	\begin{bmatrix}
		1 & -\frac{1}{2} & -\frac{1}{2}\\
		0 & \frac{\sqrt{3}}{2} & -\frac{\sqrt{3}}{2}\\
		\frac{1}{2} & \frac{1}{2} & \frac{1}{2}
	\end{bmatrix}
	\begin{bmatrix}
		a\\
		b\\
		c
	\end{bmatrix}
\end{equation}
$$

## $\alpha \beta 0 \longrightarrow abc$

$$
\begin{equation}
	\begin{bmatrix}
		a\\
		b\\
		c
	\end{bmatrix}
	=
	\begin{bmatrix}
		1 & 0 & 1\\
		-\frac{1}{2} & \frac{\sqrt{3}}{2} & 1\\
		-\frac{1}{2} & -\frac{\sqrt{3}}{2} & 1
	\end{bmatrix}
	\begin{bmatrix}
		\alpha\\
		\beta\\
		0
	\end{bmatrix}
\end{equation}
$$

## $\alpha \beta \longrightarrow dq$

$$
\begin{equation}
	\begin{bmatrix}
		d\\
		q
	\end{bmatrix}
	=
	\begin{bmatrix}
		\cos(\theta) & \sin(\theta)\\
		-\sin(\theta) & \cos(\theta)
	\end{bmatrix}
	\begin{bmatrix}
		\alpha\\
		\beta
	\end{bmatrix}
\end{equation}
$$

## $dq \longrightarrow \alpha \beta$

$$
\begin{equation}
	\begin{bmatrix}
		\alpha\\
		\beta
	\end{bmatrix}
	=
	\begin{bmatrix}
		\cos(\theta) & -\sin(\theta)\\
		\sin(\theta) & \cos(\theta)
	\end{bmatrix}
	\begin{bmatrix}
		d\\
		q
	\end{bmatrix}
\end{equation}
$$

## $ABC\longrightarrow NP0$

- $N$ (negative sequence)
- $P$ (positive sequence)
- $0$ (zero sequence)

$$
\begin{equation}
	\begin{bmatrix}
		P\\
		N\\
		0
	\end{bmatrix}
	=
	\frac{1}{3}
	\begin{bmatrix}
		1 & \alpha & \alpha^2\\
		1 & \alpha^2 & \alpha\\
		1 & 1 & 1\\
	\end{bmatrix}
	\begin{bmatrix}
		A\\
		B\\
		C\\
	\end{bmatrix}
\end{equation}
$$

$$\alpha = e^{j\frac{2\pi}{3}}$$

## $NP0 \longrightarrow ABC$

$$
\begin{equation}
	\begin{bmatrix}
		A\\
		B\\
		C\\
	\end{bmatrix}
	=
	\begin{bmatrix}
		1 & 1 & 1\\
		\alpha^2 & \alpha & 1\\
		\alpha & \alpha^2 & 1\\
	\end{bmatrix}
	\begin{bmatrix}
		P\\
		N\\
		0
	\end{bmatrix}
\end{equation}
$$

---


# dual three-phase machine

## $ADBCDEF \longrightarrow \alpha \beta x y o_1 o_2$

$$
\begin{equation}
	\label{eq:VSD Matrix}
	\begin{bmatrix}
		\alpha\\
		\beta\\
		x\\
		y\\
		o_1\\
		o_2
	\end{bmatrix}
	=\mathbf{T}_{VSD}
	\begin{bmatrix}
		A\\
		B\\
		C\\
		D\\
		E\\
		F
	\end{bmatrix}
	=\frac{1}{6}
	\begin{bmatrix}
		2 & -1 & -1 & \sqrt{3} & -\sqrt{3} & 0\\
		0 & \sqrt{3} & -\sqrt{3} & 1 & 1 & -2\\
		2 & -1 & -1 & -\sqrt{3} & \sqrt{3} & 0\\
		0 & -\sqrt{3} & \sqrt{3} & 1 & 1 & -2\\
		2 & 2 & 2 & 0 & 0 & 0\\
		0 & 0 & 0 & 2 & 2 & 2
	\end{bmatrix}
	\begin{bmatrix}
		A\\
		B\\
		C\\
		D\\
		E\\
		F
	\end{bmatrix}
\end{equation}
$$

## $ABCDEF \longrightarrow \alpha \beta x y o_1 o_2$

$$
\begin{equation}
	\label{eq: inverse VSD Matrix}
	\begin{bmatrix}
		A\\
		B\\
		C\\
		D\\
		E\\
		F
	\end{bmatrix}
	=
	\begin{bmatrix}
		1 & 0 & 1 & 0 & 1 & 0\\
		-\frac{1}{2} & \frac{\sqrt{3}}{2} & -\frac{1}{2} & -\frac{\sqrt{3}}{2} & 1 & 0\\
		-\frac{1}{2} & -\frac{\sqrt{3}}{2} & -\frac{1}{2} & \frac{\sqrt{3}}{2} & 1 & 0\\
		\frac{\sqrt{3}}{2} & \frac{1}{2} & -\frac{\sqrt{3}}{2} & \frac{1}{2} & 0 & 1\\
		-\frac{\sqrt{3}}{2}& \frac{1}{2} & \frac{\sqrt{3}}{2} & \frac{1}{2} & 0 & 1\\
		0 & -1 & 0 & -1 & 0 & 1
	\end{bmatrix}
	\begin{bmatrix}
		\alpha\\
		\beta\\
		x\\
		y\\
		o_1\\
		o_2
	\end{bmatrix}
\end{equation}
$$
