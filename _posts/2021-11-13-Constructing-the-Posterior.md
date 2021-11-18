---
layout: post
title: "Constructing the Posterior"
subtitle: "Bayes Item Response Modeling"
date: 2021-11-13
background: '/img/posts/06.jpg'
---
# Constructing the Posterior

$P(Y_k= 1|θ) =Φ(θ)$ define the probability of a correct response to item $k$
let $θ$ be a priori uniformly distributed on the interval $[−3,3]$ such
that $.001< Φ(θ)< .998$.
Note:因为认为$P(Y_k= 1|θ) =Φ(θ)$ 服从标准正态分布，所以将$θ$认为服从一个$[−3,3]$的均匀分布，通过查表可知$.001< Φ(θ)< .998$

由于$dichotomous$ $responsesy= (1,1,0,0,0)^t$,所以每个事件独立
The likelihood function for $θ$ equals
	$p(y|θ) =Φ(θ)^2(1−Φ(θ))^3$
因为$p(θ|y) =p(y|θ)p(θ)/p(y) ∝p(y|θ)p(θ)$，并且$θ$服从一个$[−3,3]$的均匀分布，所以可以认为$p(θ|y)∝Φ(θ)^2(1−Φ(θ))^3$
之后取对数求导，并令倒数得0，可以解得$Φ(θ)=2/5$时，p(θ|y)最大，根据正太分布表，找到$Φ(θ)=2/5$时，$θ=-.25$