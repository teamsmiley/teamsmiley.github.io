--- 
layout: post 
title: "tensorflow - 2 윈도우에서 1.0 설치 " 
date: 2017-02-16 00:00  
author: teamsmiley 
tags: [tensorflow]
image: /files/covers/blog.jpg
category: {machine running}
---

# tensorflow 윈도우 에서 1.0 설치

TensorFlow 1.0 버전이 릴리즈되면서 이제 Windows에서도 docker를 이용하지 않고 바로 pip3를 통해 TensorFlow를 설치할 수 있게 되었다.

## python 3 설치 

https://www.python.org/downloads/release/python-353/

다운로드 한 뒤에 설치하면 된다.

## tensorflow 설치 

pip3 install --upgrade tensorflow
 

## 설치 확인 

$ python
```python
>>> import tensorflow as tf
>>> hello = tf.constant(‘Hello, TensorFlow!’)
>>> sess = tf.Session()
>>> print(sess.run(hello))
Hello, TensorFlow!
>>> a = tf.constant(10)
>>> b = tf.constant(32)
>>> print(sess.run(a + b))
42
>>>
```
 
