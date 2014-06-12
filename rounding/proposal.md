 
Some documentation is  already there on internal representation of numbers in ledger. It stores numbers as rational numbers avoid rounding problems arising of floating point representation.
http://www.ledger-cli.org/3.0/doc/ledger3.html#Specifying-Amounts

The only time when rounding occurs is while displaying. But there are some other rounding problems in accountancy and it will be good to have a discussion on them.

From the point of view of accountants,

1.  Control over displayed precision
There is no option to force a desired display precision. There is no option to increase or decrease display precision via command line.


2. Too much accuracy.
It is good that ledger is very accurate in calculations but it sometimes becomes inconsistent with real world practices due to this accuracy. At times we may need to truncate some amounts even though amounts specified in the journal may be of higher precision. Uptill now, the precision is really controlled by amounts and they are always rounded and not truncated.


3. Accumulated inaccuracy
Small amounts matters when they accumulate to significant amounts in total. If rounding up is done per transaction then according to ledger principle even if the transaction is balanced and the overall balance too, but the actual balance must include this significant amount too.