---
layout: post
title:  "Guass Jordan Elimination Method (C)"
date:   2021-12-06 14:34:25
categories: 
tags: 
image: 
image2: 
---

# An algorithm constructed to solve Gauss-Jordan Matrix problems.

Introduction: The Gauss-Jordan method is an elimination method used to solve a system of linear equations. The ojective is to perform a process known as reduced row echelon (diagnol matrix) so that the diagnol coefficients are non-zeroes while the rest of the matrix is. The main applications are as follows: 
  * Solving system of linear equations:
    * Find solutions for a given system that is applied in Algebra mathematics
  
  * Find the determinant:
    * Applications for a square matrix in orderr to find the determinant of that system.
  
  * Finding the Inverse of a Matrix:
    * Guass-jordan eliminitation method allows operations to determine the imverse of a square matrix
  
  * Find Ranks and Bases:
    * Use reduced row echelon form to compute ranks and bases of sqaure matrices

This algorithm was part of a project for Linear Algebra course (MAT-350) that utilizes C. The bulk of the project lies within the operations because it has to satsify all system of linear equations who's dimensions are sqaure (column and row are equal). Therefore, the algorithm has to take input of matrix size (n x n), perform matrix operations, then correctly format the final solution. 

I want to concentrate on the matrix operations for this portion because it is priamrily the challenge of the algorithms. The algorithm that was used during the matrix operations was Two pointers.
