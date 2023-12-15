# Dictionary Match Counter

## Description

This SAS program performs text mining on a dataset containing textual information. It includes the creation of a dictionary, parsing of text data, and counting the frequency of dictionary words in each document.

Tested in SAS Studio 5.2 8.5 (SAS Viya 3.5).

# Author
Antonio Alisio de Meneses Cordeiro

## Usage

Run the script in your SAS environment.


```SAS
/*************************************************************
* Count dictionary matches in text
* Author: Antonio Alisio de Meneses Cordeiro
* Date: 10/12/2023
* Description: This SAS program performs text mining on a dataset
*              containing textual information. It includes the
*              creation of a dictionary, parsing of text data,
*              and counting the frequency of dictionary words
*              in each document.
*************************************************************/

/* sample dictionary words*/
DATA mycas.dictionary;
   length word varchar(*);
   input word;
   cards;
   apple
   banana
   orange
   grape
   ;
run;

/* sample text and associated IDs */
data mycas.TEXT_DATASET;
   infile datalines delimiter='|' missover;
   length text varchar(*);
   input text$ id;
   datalines;
   This is an apple and a banana|1
   I love eating an oranges for lunch|2
   Grapes are delicious, especially red grapes|3
   I have an apple and an orange for lunch|4
   ;
run;

/* Connect to CAS session */
cas mysession;

/* Assign a library reference to the CAS session */
libname mycas cas sessref=mysession;

/* Perform text mining on the 'mycas.TEXT_DATASET' table */
proc textmine data=mycas.TEXT_DATASET;
   doc_id id;
   var text;
   parse
      notagging nonoungroups
      termwgt        = none
      cellwgt        = none
      reducef        = 1
      entities       = none
      outpos          = mycas.outpos
      ;
run;

/* Create a new table store parsed text information */
data mycas.outpos/sessref=mysession;
   set mycas.outpos;
   format Term;
   format Parent;
   format DOCUMENT;
run;

/* Filter out only the words in the dictionary from the parsed text */
data mycas.outpos_filtered;
   merge mycas.dictionary(IN=A)
         mycas.outpos(IN=B rename=(Parent=word));
   by word;
   if B AND A;
run;

/* Count word frequency by document in the filtered output */
data mycas.WORD_BY_DOC_COUNT/sessref=mysession;
   set mycas.outpos_filtered;
   by DOCUMENT;
   if first.DOCUMENT then dict_word_count = 0;
   dict_word_count + 1;
   if last.DOCUMENT then output;
   keep document dict_word_count;
   rename document = id;
run;

/* Merge word frequency information back to the original text dataset */
data mycas.TEXT_DATASET_WITH_COUNT/sessref=mysession;
   merge mycas.TEXT_DATASET(IN=A)
         mycas.WORD_BY_DOC_COUNT(IN=B);	
   by id;
   if A;
   if dict_word_count = . then dict_word_count = 0;
   keep text dict_word_count;
run;
```

## License

This code is licensed under MIT License.