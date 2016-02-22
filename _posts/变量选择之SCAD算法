---
layout: post
title: "变量选择之SCAD算法"
author: "yphuang"
date: 2015-11-30
categories: blog
tags: [变量选择,SCAD,glmnet,R]

---

## SCAD的提出



据说学术界有一种现象叫做『大牛挖坑，小牛灌水』。而我等『小菜』就只有『吹水』的份了。

不过还真不要小看本『小菜』，根据著名的『[六度分割理论](https://en.wikipedia.org/wiki/Six_degrees_of_separation)』,我跟大牛的距离也是近的很呢。

不信我跟你算算。将我引入统计学习领域大门的[钟威老师](http://wise.xmu.edu.cn/people/faculty/353d37aa_7187_4486_9712_9f70cf6c4222.html),师承自统计学习大牛[ Runze Li老师](http://sites.stat.psu.edu/~rli/)，而Runze Li老师的导师居然是统计学习领域的泰斗级人物[范剑青老师,Jianqing Fan](https://orfe.princeton.edu/~jqfan/)!

看来六度分割理论还真有那么一点道理呀！

Runze Li老师在变量选择方法的研究中，有着非常突出的成果，著名的SCAD惩罚方法即由他在其博士论文中首次提出。而**变量选择**作为统计学习领域的一大热点，当然不可不知一二。于是，我们今天来学习一下Runze Li老师提出的SCAD惩罚方法。


***

八卦结束，切入正题。

***

## 论文笔记

SCAD的提出源自于论文 **Variable Selection via Nonconcave Penalized Likelihood and its Oracle Properties**。以下是这篇学术论文的学习笔记。

### 摘要

本文提出了一种用于同时达到选择变量和预测模型系数的目的的方法——SCAD。这种方法的罚函数是对称且非凹的，并且可处理奇异阵以产生稀疏解。此外，本文提出了一种算法用于优化对应的带惩罚项的似然函数。这种方法具有广泛的适用性，可以应用于广义线性模型，强健的回归模型。借助于波和样条，还可用于非参数模型。更进一步地，本文证明该方法具有Oracle性质。模拟的结果显示该方法相比主流的变量选择模型具有优势。并且，模型的预测误差公式显示，该方法实用性较强。

### SCAD的理论理解

在总结了现有模型的一些缺点之后，本文提出构造罚函数的一些目标：

* 罚函数是奇异的(singular)

* 连续地压缩系数

* 对较大的系数产生无偏的估计

SCAD模型的Oracle性质，使得它的预测效果跟真实模型别无二致。

并且，这种方法可以应用于高维非参数建模。

SCAD的目标函数如下：

$$
\begin{aligned}
  \begin{array}{rcl}
&\ &\frac{1}{2}\boldsymbol{(y - X\beta)'(y - X\beta)} + n\sum_{j =1}^{d}p_{\lambda}(|\beta_j|)  \\
&\ &\text{Where } p_{\lambda}(|\beta_j|) = \lambda^2 - (|\beta_j - \lambda|)^{2}I(|\theta|<\lambda)
  \end{array}
\end{aligned}
$$

SCAD的罚函数与$\theta$的(近似)关系如下图所示。

![](https://github.com/yphuang/yphuang.github.io/blob/master/img/SCAD_figure1.jpg)

可见，罚函数可以用二阶泰勒展开逼近。

$$
p_{\lambda} \approx p_{\lambda}(|\beta_{j0}|) +\frac{1}{2}\left\{ {p_{\lambda}}'(|\beta_{j0}|)/|\beta_{j0}|  \right\}({\beta_j}^2 - {\beta_{j0}}^2)
$$


Hard Penality,lasso,SCAD的系数压缩情况VS系数真实值的情况如下图所示。

![](https://github.com/yphuang/yphuang.github.io/blob/master/img/SCAD_figure2.jpg)

可以看到，lasso压缩系数是始终有偏的，Hard penality是无偏的，但压缩系数不连续。而SCAD既能连续的压缩系数，也能在较大的系数取得渐近无偏的估计。

这使得SCAD具有Oracle性质。

### SCAD的缺点

* 模型形式过于复杂

* 迭代算法运行速度较慢

* 在low noise level的情况下表现较优，但在high noise level的情况下表现较差。

## SCAD的实现

### SCAD迭代公式

SCAD的目标函数如下：

$$
\frac{1}{2}\boldsymbol{(y - X\beta)'(y - X\beta)} + n\sum_{j =1}^{d}p_{\lambda}(|\beta_j|)
$$

$$
\text{Where } p_{\lambda}(|\beta_j|) = \lambda^2 - (|\beta_j - \lambda|)^{2}I(|\theta|<\lambda)
$$


在$$\beta_{j0}\neq 0$$时，罚函数可以用二阶泰勒展开逼近。

$$
p_{\lambda} \approx p_{\lambda}(|\beta_{j0}|) +\frac{1}{2}\left\{ {p_{\lambda}}'(|\beta_{j0}|)/|\beta_{j0}|  \right\}({\beta_j}^2 - {\beta_{j0}}^2)
$$

从而，有如下迭代公式：

$$
\begin{aligned}
  \begin{array}{rcl}
\frac{\partial l}{\partial \beta_j} &=& \sum_{i = i}^{n}y_{i}x_{ij} - \sum_{i = 1}^{n} x_{ij} \frac{exp[\alpha + \beta' x_i]}{1 + exp[\alpha + \beta' x_i]} + n \times {p_{\lambda}}'(|\beta_{j0}|)/|\beta_{j0}|\beta_{j}  \\
\frac{\partial^2 l }{\partial {\beta_j}^2} &=& - \sum_{i = 1}^{n}\left[ \frac{(x_{ij})^2 exp[ \alpha + \beta' x_i ]}{[1 + exp(\alpha + \beta' x_i)]^2} \right] + n \times {p_{\lambda}}'(|\beta_{j0}|)/|\beta_{j0}| \\
\therefore {\beta_j}^{(m)} &=& {\beta_j}^{(m-1)} - \left[\frac{\partial^2 l(\beta)}{\partial {\beta_j}^2}  \right]^{-1} \frac{\partial l(\beta)}{\partial \beta_j}  
  \end{array}
\end{aligned}  
$$


根据以上公式，代入迭代步骤，即可实现算法。

## SCAD的R实现


```r
##------数据模拟--------

library(MASS)
##mvrnorm()

##定义一个产生多元正态分布的随机向量协方差矩阵
Simu_Multi_Norm<-function(x_len, sd = 1, pho = 0.5){
  #初始化协方差矩阵
  V <- matrix(data = NA, nrow = x_len, ncol = x_len)
  
  #mean及sd分别为随机向量x的均值和方差
  
  #对协方差矩阵进行赋值pho(i,j) = pho^|i-j|
  for(i in 1:x_len){ ##遍历每一行
    for(j in 1:x_len){ ##遍历每一列
      V[i,j] <- pho^abs(i-j)
    }
  }
  
  V<-(sd^2) * V
  return(V)
}


##产生模拟数值自变量X
set.seed(123)
X<-mvrnorm(n = 200, mu = rep(0,10), Simu_Multi_Norm(x_len = 10,sd  = 1, pho = 0.5))

##产生模拟数值：响应变量y
beta<-c(1,2,0,0,3,0,0,0,-2,0)

#alpha<-0

#prob<-exp(alpha + X %*% beta)/(1+exp(alpha + X %*% beta))

prob<-exp( X %*% beta)/(1+exp( X %*% beta))

y<-rbinom(n = 200, size = 1,p = prob)

##产生model matrix
mydata<-data.frame(X = X, y = y)

#X<-model.matrix(y~., data = mydata)



##包含截矩项的系数
#b_real<-c(alpha,beta)
b_real<-beta

########----定义惩罚项相关的函数-----------------


##定义惩罚项

####运行发现，若lambda设置为2，则系数全被压缩为0.

####本程序根据rcvreg用CV选出来的lambda设置一个较为合理的lambda。


p_lambda<-function(theta,lambda = 0.025){
  p_lambda<-sapply(theta, function(x){
    if(abs(x)< lambda){
      return(lambda^2 - (abs(x) - lambda)^2)
    }else{
      return(lambda^2)
    }
  }
  
  )
  
  return(p_lambda)
}


##定义惩罚项导数
p_lambda_d<-function(theta,a = 3.7,lambda = 0.025){
  if(abs(theta) > lambda){
    if(a * lambda > theta){
      return((a * lambda - theta)/(a - 1))
    }else{
      return(0)
    }
  }else{
    return(lambda)
  }
}

# ##当beta_j0不等于0，定义惩罚项导数近似
# p_lambda_d_apro<-function(beta_j0,beta_j,a = 3.7, lambda = 2){
#   return(beta_j * p_lambda_d(beta = beta_j0,a = a, lambda = lambda)/abs(beta_j0))
# }
# 
# 
# ##当beta_j0 不等于0，指定近似惩罚项,使用泰勒展开逼近
# p_lambda_apro<-function(beta_j0,beta_j,a = 3.7, lambda = 2){
#   if(abs(beta_j0)< 1e-16){
#     return(0)
#   }else{
#     p_lambda<-p_lambda(theta = beta_j0, lambda = lambda) + 
#       0.5 * (beta_j^2 - beta_j0^2) * p_lambda_d(theta = beta_j0, a = a, lambda = lambda)/abs(beta_j0)
#   }
# }



#define the log-likelihood function
loglikelihood_SCAD<-function(X, y, b){
  linear_comb<-as.vector(X %*% b)
  ll<-sum(y*linear_comb) + sum(log(1/(1+exp(linear_comb)))) - nrow(X)*sum(p_lambda(theta = b))
  return (ll)
}


##初始化系数
#b0<-rep(0,length(b_real))

#b0<- b_real+rnorm(length(b_real), mean = 0, sd = 0.1)

##将无惩罚时的优化结果作为初始值
b.best_GS<-b.best

b0<-b.best_GS

##b1用于记录更新系数
b1<-b0

##b.best用于存放历史最大似然值对应系数
b.best_SCAD<-b0

# the initial value of loglikelihood

ll.old<-loglikelihood_SCAD(X = X,y = y, b = b0)


# initialize the difference between the two steps of theta
diff<-1  
#record the number of iterations
iter<-0
#set the threshold to stop iterations
epsi<-1e-10
#the maximum iterations  
max_iter<-100000
#初始化一个列表用于存放每一次迭代的系数结果
b_history<-list(data.frame(b0))

#初始化列表用于存放似然值
ll_list<-list(ll.old)



#######-------SCAD迭代---------
while(diff > epsi & iter < max_iter){
  for(j in 1:length(b_real)){
    if(abs(b0[j]) < 1e-06){
      next()
    }else{
      
      #线性部分
      linear_comb<-as.vector(X %*% b0)
      
      #分子
      nominator<-sum(y*X[,j] - X[,j] * exp(linear_comb)/(1+exp(linear_comb))) + 
        nrow(X)*b0[j]*p_lambda_d(theta = b0[j])/abs(b0[j])
      
      
      #分母,即二阶导部分
      denominator<- -sum(X[,j]^2 * exp(linear_comb)/(1+exp(linear_comb))^2) +
        nrow(X)*p_lambda_d(theta = b0[j])/abs(b0[j])
      
      #2-(3) :更新b0[j]
      b0[j]<-b0[j] - nominator/denominator
      
      #2-(4)
      if(abs(b0[j]) < 1e-06){
        b0[j] <- 0
      }
      
      #       #更新似然值
      #       ll.new<- loglikelihood_SCAD(X = X, y = y, b = b0)
      #       
      #       
      #       
      #       #若似然值有所增加，则将当前系数保存
      #       if(ll.new > ll.old){
      #         #更新系数
      #         b.best_SCAD[j]<-b0[j]
      #       }
      #       
      #       #求差异
      #       diff<- abs((ll.new - ll.old)/ll.old)
      #       ll.old <- ll.new
      #       iter<- iter+1 
      #       b_history[[iter]]<-data.frame(b0)
      #       ll_list[[iter]]<-ll.old
      #       ##当达到停止条件时，跳出循环
      #       if(diff < epsi){
      #         break
      #       }
      #       
      
    }
  }
  
  #更新似然值
  ll.new<- loglikelihood_SCAD(X = X, y = y, b = b0)
  
  
  
  #若似然值有所增加，则将当前系数保存
  if(ll.new > ll.old){
    #更新系数
    b.best_SCAD<-b0
  }
  
  #求差异
  diff<- abs((ll.new - ll.old)/ll.old)
  ll.old <- ll.new
  iter<- iter+1 
  b_history[[iter]]<-data.frame(b0)
  ll_list[[iter]]<-ll.old
  
  
}


b_hist<-do.call(rbind,b_history)
#b_hist

ll_hist<-do.call(rbind,ll_list)
#ll_hist

#
iter

##
ll.best<-max(ll_hist)
ll.best

##
b.best_SCAD

##对比
cbind(coeff_glm,b.best,b.best_SCAD,b_real)


##----------ncvreg验证-----------
library(ncvreg)
my_ncvreg<-ncvreg(X,y,family = c("binomial"),penalty = c("SCAD"),lambda = 2)
my_ncvreg$beta


my_ncvreg<-ncvreg(X,y,family = c("binomial"),penalty = c("SCAD"))
summary(my_ncvreg)
my_ncvreg$beta

###用cv找最优的lambda

scad_cv<-cv.ncvreg(X,y,family = c("binomial"),penalty='SCAD')
scad_cv$lambda.min
mySCAD=ncvreg(X,y,family = c("binomial"),penalty='SCAD',lambda=scad_cv$lambda.min)

summary(mySCAD)

ncv_SCAD<-mySCAD$beta[-1]

##对比
myFinalResults<-cbind(无惩罚项回归=coeff_glm, GS迭代 = b.best,
                            GS_SCAD迭代 = b.best_SCAD, ncvreg = ncv_SCAD,真实值 = b_real)
save(myFinalResults,file = "myFinalResults.rda")
```
