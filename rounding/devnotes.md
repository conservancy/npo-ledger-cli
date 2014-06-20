Purpose: Adding new sub-directive under commodity directive in ledger 

  Adding a new sub directive was easy. They are defined in textual.cc. To add under commodity directive look for function void instance_t::commodity_directive(char * line). Each sub directive is a keyword to be matched. Simply add a new if else statement checking for the keyword. Add suitable code.  Arguments passed to the sub directive are held by variable 'b'.
  

