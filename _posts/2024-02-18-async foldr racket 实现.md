---
layout: post
title:  async foldr racket 实现
date:   2024-02-18 22:03:48 +0800
categories: racket
---

<link rel="stylesheet" href="/assets/css/katex.min.css" >

racket 由于其严格求值的特性,实现`foldr`时无法做到提早退出,只能是严格的$$O(n)$$时间复杂度可空间复杂度.可haskell里面的`foldr`可不是这样的,在haskell里,一个表达式不一定被完全求值,因此`foldr`完全可以提早退出,而不计算剩下的部分.

我想在racket里面也设计这个功能,取道于racket的惰性求值函数:`delay`,`lazy`和`force`.按照racket的命名风格,这个函数应该叫`foldr/async`.

```racket
#lang racket

;; 不同于普通的foldr, proc 是一个(v,l) -> ? 类型的东西
;; 而v的参数是一个promise,必须 force 才能使用.
;; 这个函数最终也会返回一个promise,需要force才能使用
(define (foldr/async proc fallback lst)
  (if (empty? lst) (delay fallback)
      (proc (car lst)
            (lazy (foldr/async proc fallback (cdr lst))))))
;; 自动替你force最终结果,免得你忘掉
(define (foldr/async/force proc fallback lst)
  (force (foldr/async proc fallback lst)))
```
一些测试代码: 
```racket
(define (prime? n)
  (let then ([i 2])
    (cond [((sqr i) . > . n) #t]
          [((remainder n i) . = . 0) #f]
          [else (then (add1 i))])))

(define (noisy-cons v fb)
  (printf "get ~a \n" v)
  (if (prime? v)
      (cons v (force fb))
      '()))

(foldr/async/force noisy-cons '() '(1 2 3 4 5 6))
```

测试效果如下:
```
get 1 
get 2   
get 3   
get 4   
'(1 2 3)
```

可见这个函数提早中止了,没有计算剩下的部分.