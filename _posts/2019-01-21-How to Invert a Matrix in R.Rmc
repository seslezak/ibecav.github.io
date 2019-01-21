#Creating a Hand Rolled Inverse Matrix in *R*
#Lesson One
###Build the matrix (note: this example is a symmetrical matrix); the transpose of A is equal to A


A <- matrix(c(1,0,1,0,2,1,1,1,1), ncol = 3)
A

#Build the Identity Matrix
###Attach it on the right. We want to manipulate the matrix to get the I-Matrix on the left. What's on the right will be the inverse matrix.

AB <- cbind(A, diag(c(1, 1, 1)))
AB

#Start Manipulating the Matrix
###We can swap, add or subtract, and divide rows, depending on what we need to do. Note the matrix changes as we manipulate each row.
###We want a 1 in the [1, 1] position. In this case, we already have that, so there is no need to manipulate. In general, we divide the row by the value in [1, 1] as shown here.

AB
AB[1,] <- AB[1, ]/AB[1,1]
AB

###No change, as expected.
#Move on to Row 2
###We want a 0 in the [2, 1] position. In this case, we already have that, so there is no need to manipuate, In general, we subtract the values in Row 1 multiplied by the value in [2, 1] from the values in Row 2, as shown here.

AB
AB[2,] <- AB[2,] - 0 * AB[1, ]
AB

###Again, no change.
#On to Row 3
###Remember, we want a 0 in the [3, 1] position. We subtract the values in Row 1 multiplied by the value in [3, 1] from the existing values in Row 3.

AB
AB[3,] <- AB[3,] - 1 * AB[1, ]
AB

###Now, Column 1 contains 1, 0, 0
#Continue Building the Identity Matrix on the Left
###We want a 1 in the [2, 2] position. We can divide the row by the value in [2, 2].

AB
AB[2,] <- AB[2,]/AB[2,2]
AB

###Notice we are not modifying the original AB matrix.  We are modifying the matrix step-by-step.
#Continue Building Column 2
###Our identity matrix needs a 0 in position [3, 2] so we subtract Row 2 multiplied by the value in [3, 2] from Row 3.

AB
AB[3,] <- AB[3,] - 1 * AB[2,]
AB

###Looks good.
#Continue Building Colulmn 3
###We need a 1 in the [3, 3] position. To replace -0.5 we divide Row 3 by the value in [3, 3] or -0.5.

AB
AB[3,] <- AB[3,]/AB[3,3]
AB

###We have ones in the diagonal and zeroes below it.
#Continue Clean Up in Row 3
###How do we get 0 in position [2, 3]?  Subtract the value in [2, 3] multiplied by Row 3 from the existing values in Row 2.

AB
AB[2,] <- AB[2,] - 0.5 * AB[3,]
AB

###One more to go. The right side has been evolving, too.
#Setting the Final Zero in [1, 3]
###Following our strategy, take the value in [1, 3], multiply it by Row 3, then subtract the product from Row 1.

AB
AB[1,] <- AB[1,] - 1 * AB[3,]
AB

###Behold the Identity Matrix is now on the left. The inverse matrix is on the right.
#Confirm Our Results
###Split off the Inverse Matrix. Use the *R* function *solve()* to confirm we are correct.

B <- AB[, 4:6]; B
solve(A)

###It checks out!  Matrix inverted!

#Lesson Two
###Let's build a new example using the same strategies
###Create the original matrix on the left and the identity matrix on the right

A <- matrix(c(2, 2, 3, 3, 5, 9, 5, 6, 7), ncol = 3)
A
AB <- cbind(A, diag(c(1, 1, 1)))
AB

#Turn the value 2 in [1, 1] into a one with division

AB
AB[1,] <- AB[1, ]/AB[1,1]
AB

#To change the 2 in [2, 1] to 0, we multiply Row 1 by element [2, 1] and subtract from Row 2

AB
AB[2,] <- AB[2,] - 2 * AB[1, ]
AB

#To change the 3 in [3, 1] to 0, we multiply Row 1 by element [3, 1] and subtract from Row 3

AB
AB[3,] <- AB[3,] - 3 * AB[1, ]
AB

###We now have 1, 0, 0 in Column 1
#Create a 1 in [2, 2]
###Divide Row 2 by the value in [2, 2]

AB
AB[2,] <- AB[2,]/AB[2,2]
AB

#We need a 0 in [3, 2]
###Multiply Row 2 by the value in [3, 2] and subtract from Row 3

AB
AB[3,] <- AB[3,] - 4.5 * AB[2,]
AB

#Next, Create a 1 in [3, 3]
###Divide Row 3 by the value in [3, 3]

AB
AB[3,] <- AB[3,]/AB[3,3]
AB

###The diagonal and below resemble the Identity Matrix
#Next, to Convert [2, 3] to 0
###We are working with Row 2
###Multiply [2, 3] by Row 3 and subtract from Row 2

AB
AB[2,] <- AB[2,] - 0.5 * AB[3,]
AB

#Next, to Convert [1, 3] to 0
###We are working with Row 1
###Multiply [1, 3] by Row 3 and subtract from Row 1

AB
AB[1,] <- AB[1,] - 2.5 * AB[3,]
AB

#Finish Up with the Final 0 in [1, 2]
###[1, 2] times Row 2 then subtract from Row 1

AB
AB[1,] <- AB[1,] - 1.5 * AB[2,]
AB

###We have the Identity Matrix on the Left and the inverse matrix on the right
#Check Results with *solve()*

B <- AB[, 4:6]
B
solve(A)

#Lesson Three
###A different matrix and no commentary.
###Follow along...

A <- matrix(c(1,9,2,8,3,7,4,6,5), ncol = 3)
A
AB <- cbind(A, diag(c(1, 1, 1)))
AB

#Step 1

AB
AB[1,] <- AB[1, ]/AB[1,1]
AB

#Step 2

AB
AB[2,] <- AB[2,] - AB[2, 1] * AB[1, ]
AB

#Step 3

AB
AB[3,] <- AB[3,] - AB[3, 1] * AB[1, ]
AB

#Step 4

AB
AB[2,] <- AB[2,]/AB[2,2]
AB

#Step 5

AB
AB[3,] <- AB[3,] - AB[3, 2] * AB[2,]
AB

#Step 6

AB
AB[3,] <- AB[3,]/AB[3,3]
AB

#Step 7

AB
AB[2,] <- AB[2,] - AB[2, 3] * AB[3,]
AB

#Step 8

AB
AB[1,] <- AB[1,] - AB[1, 3] * AB[3,]
AB

#Step 9

AB
AB[1,] <- AB[1,] - AB[1, 2] * AB[2,]
AB

#Step 10

B <- AB[, 4:6]
B
solve(A)
round(AB, 3)

