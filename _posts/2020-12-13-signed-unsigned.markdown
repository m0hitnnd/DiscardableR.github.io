---
layout: post
title:  "Signed Unsigned with Shuri & Scott"
date:   2020-12-12 23:23:19 +0530
categories: jekyll update
---

***Scott*** - Heyy *Shuri*!!  
***Shuri*** - Heyy *Scott*!!  
***Scott*** - Need your help to understand about data types and their range.  
***Shuri*** - You mean...Integer, Float, Double, etc ?  
***Scott*** - Yes Yes.  
***Scott*** - I am very confused. There's signed data type, unsigned data type, Int32, Float32,  Int64, double....  
***Shuri*** - Okay Okay Okay...got it. Let's take Integer data type to understand this.  
***Scott*** - Perfect!  
***Shuri*** - So, Integer can be of 8 bits which is represented by Int8, 16 bits represented by Int16, 32 bits represented by Int32 and so on...  
***Shuri*** - A bit can store either value 0 or 1. That means 2 possible values.  
```0    or    1```  
2 bits can store 2 values(0 or 1) for each bit, that means we can represent or store 22 possible values with 2 bits.


``` 
0   0    ->   0
0   1    ->   1
1   0    ->   2
1   1    ->   3
```  
                 
***Scott*** - Okay, that means with Int2, we can represent 0, 1, 2 & 3. So, then the range  for Int2 will be [0, 3], right?  
***Shuri*** - Umm...wrong. The range for Unsigned Int2 would be [0, 3],  not for a signed Int2.  
***Shuri*** - Before you even ask, Unsigned Ints are non negative.  And, Signed Int will have both negative and positive values.  
***Scott*** - Okay..but how do we represent a negative value in bit? You can only put 0 or 1 right?  
                You can't put a -ve or +ve sign.  
***Shuri*** - Correct, you can't put -ve or +ve sign. But, there are ways to represent a negative value.  
                One way is to invert the digits and add 1 to it.  
                The resultant would represent its negative value. Lets take 01 which represents 1.  
                                                   01 -> invert digits  -> 10 -> add 1 -> 11  
                                                   Now, 11 represents -1.  
***Shuri*** - Also, inverting of digits and adding 1 to it is called 2's complement.  
***Scott*** - So, 11 which we used to represent 3 for unsigned Int2,  
               can now now be used to represent -1 for signed Int2 right?  
***Shuri*** - That's correct.  And, 1 0 can be used to represent -2, since `2's complement` of 1 0 is 1 0.  
***Shuri*** - This way with the same number of bits we can be used to represent both positive and negative values.  
          Signed Int2 will have these values:  
```
0   0    ->   0
0   1    ->   1
1   0    ->  -2
1   1    ->  -1
```  

***Scott*** - Got it. Now if we look at signed Int2's range, it's `[-2, 1]`. It does not include 2 and 3 as we have used these to represent negative numbers, -2  & -1 respectively.  
***Shuri*** - Exactly!  
***Scott*** - Okay...now what if I want to store 3 for a signed Int?  
***Shuri*** -  For a signed Int, we don't have space to store 3 with just 2 bits, as with 2 bits we can only store values till 1.  
                 With 3 bits, you can store ranging from -4 to 3.  
```
0   0   1   ->  1                              1   0   1    ->   -3
0   1   0   ->  2                              1   1   0    ->   -2
0   1   1   ->  3                              1   1   1    ->   -1
```  

***Shuri*** - Now if you see, we can actually deduce a formula for both signed and unsigned ranges.  
For n bits, unsigned range would be `[0, 2n - 1]`, and for signed range it would be `[-2n-1, 2n-1 - 1]`.  
***Scott*** - So, is it correct to say that signed data types reduces the range, as with unsigned we can represent more values?  
***Shuri*** - I would not say it reduces the range, it actually shifts the range from positive values to negative values.  
***Scott*** - Makes sense. Okay, I have one last question, when we declare just Integer (Int) as data type, is that unsigned or signed?  
              And, how many bits it stores, because when we write Int we don't mention number of bits there?  
***Shuri*** - Good question!! By default it's signed. To make it unsigned, we have to write UInt.  
              And, how many bits it stores is dependent on the platform.  
              On 32 bit platform, it behaves as Int32. On 64 bit platform, it behaves as Int64.  
              You can actually check Swift documentation for Int which tells the same thing.  

***Scott*** - Noiceeee. Thanks ***Shuri***🙂  
***Shuri*** - Anytime 🙂 And, don't forget to check other Data Types and their range.  
***Scott*** - Already on it...  