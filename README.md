Methods:

Prepping initial data
**Use a sed command**

a. sed ‘s/[[:<:]]Cl[[:>:]]/OH/g’ CHNO_Smiles.tsv > OH_Prepped_CHNO_Smiles.tsv

b. Enclosing Cl with [[:<:]] [[:>:]] ensures only Cl is replaced rather than a larger string containing Cl

c. The global tag g ensures all instances of Cl are removed. (Cl is chlorine)

Interleave OH and NH2 prepped files for the 2 nucleotide analogue libraries (CHO and CHNO)

**Use a paste command**

a. paste -d "\n" OH_Prepped_CHNO_Smiles.tsv NH2_Prepped_CHNO_Smiles.tsv > Prepped_CHNO_Smiles.tsv
