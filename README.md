Methods:

Prepping initial data
**Use a sed command**

a. sed ‘s/[[:<:]]Cl[[:>:]]/OH/g’ CHNO_Smiles.tsv > OH_Prepped_CHNO_Smiles.tsv

b. Enclosing Cl with [[:<:]] [[:>:]] ensures only Cl is replaced rather than a larger string containing Cl

c. The global tag g ensures all instances of Cl are removed. (Cl is chlorine)

Interleave OH and NH2 prepped files for the 2 nucleotide analogue libraries (CHO and CHNO)

**Use a paste command**

a. paste -d "\n" OH_Prepped_CHNO_Smiles.tsv NH2_Prepped_CHNO_Smiles.tsv > Prepped_CHNO_Smiles.tsv

**Create an automated workflow using jupyter notebook only **

Define a function which takes a row of a dataframe as an input, converts the Smiles representation into the first layer of Inchikey (ignoring all but the first 14 characters to remove stereochemical considerations as the stereochemistry of the network output is not specified)

Write a remove_degeneracy script which:
Imports a target library or test dataset
Uses the apply method to find the inchikey representation for all molecules of the test and library sets
Iterates through all molecules, appending the Smiles and Inchikey of molecules with an Inchikey not already found, to arrays of molecules which aren’t structural isomers (to remove degenerate structural isomers and speed up the matching script)
Process the library and test sets using the remove_degeneracy script (no degeneracy found in the test sets, significant degeneracy found in the library sets, degeneracy between the 2 library sets found), combine the 2 nucleoside library test sets into one combined set with all degeneracy removed 

Write an automated find_matches script which:
Imports the library sets and network outputs with degeneracy removed as dataframes
Stores the library and test inchikey codes in arrays and tests if each test code is found in the library codes
Appends the generation and smiles data of the matching inchikey codes to arrays and generates a dictionary from those arrays
Produces a dataframe from the dictionary using the pd.DataFrame function 

Use this workflow to find matches for all 5 network outputs and both nucleotide analogue libraries, writing the output dataframes to files using the df.to_csv method

