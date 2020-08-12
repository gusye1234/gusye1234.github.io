---
layout: post
title: Cheatsheet for Python
subtitle: quick knwoledge for various libraries in Python
tags: [ note, code, python ]
comments: true
---

1. `spicy.sparse`. There are some common-used sparse matrix in `scipy`:
   1. `dok_matrix`, `coo_matrix`, fast for create matrix, and add new data. A code like `m[rows, colums] = datas` is faster than the other sparse format. What's more, the way those format store the matrix is pretty intuitive. "dok" stands for *Dictionary Of Keys*, "coo" stands for *Coordinate*.
   2. `csr_matrix`, `csc_matrix`. They are both fast at matrix computation, but slow at modifying. The main difference between them is
      * `csr` stands for *Compressed Sparse Row*, which implies we store the way along with the row. So, there are three variables to indicate the matrix. `data`, `indices`, `indptr`. `csr` can access a row's data quickly with `data[indptr[row]:indptr[row+1]]`, and their column index `indices[indptr[row]:indptr[row+1]]`.
      * `csc`, on the contrary, stands for*Compressed Sparse Column*. And Yes, `csc` can access a column's data quickly with  `data[indptr[column]:indptr[column+1]]`,and their row index `indices[indptr[column]:indptr[column+1]]`
2. 