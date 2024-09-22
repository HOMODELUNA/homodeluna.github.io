---
layout: post
title:  racket map 源码剖析
date:   2024-09-22 22:38:34 +0800
categories: language racket
---

[github](https://github.com/racket/racket/blob/master/racket/collects/racket/private/map.rkt)上的源代码:

如果让普通程序员自己实现,那么他很容易写出
```lisp
(define (map f lst)
  (if (null? lst) null
      (cons (f (car lst)) 
            (map f (cdr lst)))))
```
然后让我们看看racket解释器内部是如何实现这个函数的:

```lisp
; 这些函数预计是常用的,编者就鼓励编译器把他们内联掉

(begin-encourage-inline

   (define map2 ; 这个函数后来被provide出去了
      (let ([map
             (case-lambda 
              [(f l)
               (if (or-unsafe (and (procedure? f)
                                   (procedure-arity-includes? f 1)
                                   (list? l)))
                   (let loop ([l l])     ;此处是核心流程
                     (cond               ;用一个lambda来保存f这个函数,这样迭代时改变的只有l
                      [(null? l) null]
                      [else
                       (let ([r (cdr l)]) ; so `l` is not necessarily retained during `f`
                         (cons (f (car l)) (loop r)))]))
                   (gen-map f (list l)))]
              ; 后面是涉及多参数map的一些泛化情况
              [(f l1 l2)
               (if (or-unsafe
                    (and (procedure? f)
                         (procedure-arity-includes? f 2)
                         (list? l1)
                         (list? l2)
                         (= (length l1) (length l2))))
                   (let loop ([l1 l1] [l2 l2])
                     (cond
                      [(null? l1) null]
                      [else 
                       (let ([r1 (cdr l1)]
                             [r2 (cdr l2)])
                         (cons (f (car l1) (car l2)) 
                               (loop r1 r2)))]))
                   (gen-map f (list l1 l2)))]
              [(f l . args) (gen-map f (cons l args))])])
        map))
        ...)
```

令人意外的地方就是这个map除了和用户第一眼想到的函数有点细节优化之外,大体上是一样的,没有玄学