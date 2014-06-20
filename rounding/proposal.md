 
Some documentation is  already there on internal representation of numbers in ledger. It stores numbers as rational numbers avoid rounding problems arising of floating point representation.

http://www.ledger-cli.org/3.0/doc/ledger3.html#Specifying-Amounts

The only time when rounding occurs is while displaying. But there are some other rounding problems in accountancy and it will be good to have a discussion on them.

From the point of view of accountants,

1.  Control over displayed precision

There is no option to force a desired display precision. There is no option to increase or decrease display precision via command line.
for example 
D $1
2014-06-09 day 2
  A		$1.89
  C	
  
The ledger pay attention to the  the precision used in specifying the amount, whether thousand marks were used, etc. This is done so that printing the commodity looks the same as the way you use it. Using options like D will be overridden.
You may use option "D" to increase the display precision but not decrease. Point to note is that there is no effect on calculations.


2. Too much accuracy.

It is good that ledger is very accurate in calculations but it sometimes becomes inconsistent with real world practices due to this accuracy. At times we may need to truncate some amounts even though amounts specified in the journal may be of higher precision. Uptill now, the precision is really controlled by amounts and they are always rounded and not truncated.


3. Accumulated inaccuracy

If rounding up was done per transaction, then the error per transaction may add up to significant difference than the actual total.   
  For example :
2014-06-09 day 2
  A		2 AAA @$5.52
  C	
  
2014-06-08 day 1
  A		1 BBB @$5.52
  C
  
In the above example if there is rounding up in each transaction then the overall result will be off by few cents.
This problem does not occur in ledger as rounding is deffered till the last point that is at the time of display.
  