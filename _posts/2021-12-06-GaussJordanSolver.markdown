---
layout: post
title:  "Guass Jordan Elimination Method (C)"
date:   2021-12-06 14:34:25
categories: 
tags: 
image: /assets/article_images/2014-11-30-mediator/night-track.jpg
image2: /assets/article_images/2014-11-30-mediator/night-track.jpg
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

I want to concentrate on the matrix operations for this portion because it is priamrily the challenge of the algorithms. The algorithm that was used during the matrix operations was Two pointers. An example can be shown below:

	printf("Now enter the matrix:\n");						
		for(i=0;i<sizeOfMatrix;i++)
			for(j=0;j<sizeOfMatrix;j++)
				scanf("%f",&input[i][j]);

	for(i=0;i<sizeOfMatrix;i++)									
	for(j=0;j<sizeOfMatrix;j++)							
	if(i==j)										
	Inverse[i][j]=1;									
	else											
	Inverse[i][j]=0;		
As we see, the algorithm uses two pointers to iterate throught matrices to check if each are equal and squared. These pointers are merely checking the size of the arrays and not computing or doing any matrix operations. 
	
	for(k=0;k<sizeOfMatrix;k++)									 
	{														
		localVariable=input[k][k];										
		for(j=0;j<sizeOfMatrix;j++)								
		{
			input[k][j]/=localVariable;									
			Inverse[k][j]/=localVariable;									

		}													
		for(i=0;i<sizeOfMatrix;i++)									
		{
			localVariable = input[i][k];									
			for(j=0;j<sizeOfMatrix;j++)							
			{												
				if(i==k)
					break;									
				input[i][j] -= input[k][j]*localVariable;						
				Inverse[i][j] -= Inverse[k][j]*localVariable;						
			}
		}
	}
This code snippet shows that when the pointers iterate through the array, while performing operations on each index of the array and the matrix operation will only be satisfied till index [i] is equaled to [k] (where solutions are stored). 
