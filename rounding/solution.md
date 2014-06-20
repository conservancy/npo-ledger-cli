ledger does not round any amount but simply converts it to rational number representation and create rounded amounts only to display in report.Hence, amounts are not rounded until the last moment. But in real world accounting,truncation or rounding takes place after each transaction when calculated amount is rounded to a normally accepted standard. for example,an amount of $41.2334 will be rounded to $41.23 and this will be used in further calculations. A point to note is that we knowingly add truncation or rounding error to ledger if we adopt this method since ledger does not round at all. But this is important to maintain consistency with other accountancy applications, it is good to add an option which will generate a report using this method. This will not change the default behavior until this option is applied.
 
 For example , this posting.
2014/06/18  TEST
 A                               10 AAA  @ 10.1234 EUR
 A                               10 BBB  @ 10.5692 EUR
 C
if we set a precision of 2
this should be calculated as 10*10.1234 = 101.234 rounded to 101.23 and 10*10.5692 rounded to 105.69  and final result is 101.23+105.69=206.92 EUR debited from C.

Let us add a sub-directive precision to commodity directive which will round per transaction when applied. 