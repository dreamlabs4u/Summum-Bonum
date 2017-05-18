---
layout: post
title:  "Monad Explained! - The Scala way"
categories: Scala
tags: Scala
author: Toney Thomas
---

* content
{:toc}

# Monad

Monad, M[T], is an amplification (or considered as a wrapper) of a generic type T such that 

1. Any Monad, M[T] can be created by applying a creation function on T
    x: T => M[T]
    
2. It provides a mechanism to for applying a function which takes T and gives out Monad of the resultant type 

	(x: M[T], fn: T => M[Z]) => M[Z]
  

These two featurs must obey the following 3 Monadic law : - 

1. If you apply the Monad creation function to an any existing Monadic instnace then it should give out a logical equivalent Monad 

	monad1 = M[T]
    monad2 = creationFn(monad1)

    monad 1 and monad2 should be same


2. Appying a function to the result of applying the construction function should always produce a logically equivalent Monad if we apply 
   that function directly on the value 

3. Composition rule : 
	
    
```scala    
	val f = (X) => M[Y]
	val g = (Y) => M[Z]
	M[X] mx = f;
	val my : M[Y] = monadicFunction(mx, f)
	val mz1 : M[Z]  = monadicFunction(my, g)

	val h: M[Z] = monadicCompose(f, g);
	val mz2 = monadicFunction(mx, h);
```


Applying to a value a first function followed by applying to the result a second function should produce a logically identical Monad 
if you applying a composition function to the origial value.



