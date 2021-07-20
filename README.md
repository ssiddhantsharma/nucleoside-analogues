Prepping files:
Create a virtual environment -> py4chemoinformatics and install rdkit and openbabel
Replace Cl (placeholder for nitrogenous base) with OH and NH2 functional groups 
Replace Cl with both groups because either could react (via an elimination mechanism) with a nitrogenous base (and the NH2 group could additionally react to form the nitrogenous base in situ)
Use a sed command
sed ‘s/[[:<:]]Cl[[:>:]]/OH/g’ CHNO_Smiles.tsv > OH_Prepped_CHNO_Smiles.tsv
Enclosing Cl with [[:<:]] [[:>:]] ensures only Cl is replaced rather than a larger string containing Cl 
The global tag g ensures all instances of Cl are removed 
Interleave OH and NH2 prepped files for the 2 nucleotide analogue libraries (CHO and CHNO)
Use a paste command
paste -d "\n" OH_Prepped_CHNO_Smiles.tsv NH2_Prepped_CHNO_Smiles.tsv > Prepped_CHNO_Smiles.tsv

Matching Script:
Create a workflow using datawarrior
Import the library sets (the nucleotide analogue datasets) and network outputs (Formose ammonia, formose, glucose ammonia, glucose and pyruvic acid) into datawarrior and find the canonical code from Smiles (Chemistry -> From Chemical Structure -> Add Canonical Code)
Copy the canonical code into a tsv file with the Smiles 
Write and execute a find_matches script which takes the filepaths of the library and test sets and returns a dataframe containing all matches, their library and test smiles representations, their canonical code and the generation they first appear in 
Imports the library and test files as dataframes using pandas
Stores the canonical codes from the 2 files in arrays
Iterates through the test array, testing if each canonical code is present in the library array
Appends the generation and smiles data of the matching canonical codes to arrays and generates a dictionary from those arrays
Produces a dataframe from the dictionary using the pd.DataFrame function 
Create an automated workflow using jupyter notebook only 
Define a function which takes a row of a dataframe as an input, converts the Smiles representation into the first layer of Inchikey (ignoring all but the first 14 characters to remove stereochemical considerations as the stereochemistry of the network output is not specified)
Write a remove_degeneracy script which:
Imports a target library or test dataset
Uses the apply method to find the inchikey representation for all molecules of the test and library sets
Iterates through all molecules, appending the Smiles and Inchikey of molecules with an Inchikey not already found, to arrays of molecules which aren’t structural isomers (to remove degenerate structural isomers and speed up the matching script)
Process the library and test sets using the remove_degeneracy script (no degeneracy found in the test sets, significant degeneracy found in the library sets)
Write an automated find_matches script which:
Imports the library sets and network outputs with degeneracy removed as dataframes
Stores the library and test inchikey codes in arrays and tests if each test code is found in the library codes
Appends the generation and smiles data of the matching inchikey codes to arrays and generates a dictionary from those arrays
Produces a dataframe from the dictionary using the pd.DataFrame function 
Use these 2 workflows to find matches for all 5 network outputs and both nucleotide analogue libraries, writing the output dataframes to files using the df.to_csv method 

Descriptor Plotting:
Define a function to take the filepaths of 2 tsv files as its arguments and write to a file a concatenation of the 2 files
Use the function to combine CHO and CHNO matches where necessary to create combined tsv files 
Define a function to sort molecules of matches datasets by the generation they first appeared in 
Use the function to sort combined matches files conserving all data (Smiles, inchikey etc) associated with each molecule
Matches vs Generations plots:
Define a generations_counter function to calculate the cumulative number of molecules which have appeared by a certain generation and output this as an array
Define a generations_subplot function to read in a file, calculate the cumulative generations data using the generations_counter function and plot a subplot 
Use the generations subplot function to plot a figure of no. molecules against generation for all 5 networks, plotting both matches and the whole network size 
Mols2Grid plot:
Take the first match of each generation for each network and append it to a tsv file of representative matches
Detail the order of the matches by network and generation in a list
Define a display function using the DrawMolsToGridImage method to display a set of molecules with the legend set to a list of details passed as a second argument 
Use the display function to present the representative matches with the legend set to the list containing the matches details
Energy minimisation
Define a minimise function to take an input dataframe and write an SDF file containing all molecules with the molecules having the data contained by the dataframe as attached properties and Use the ! magic to run a terminal command of the form 
!obabel pp_out.sdf -O Minimised.sdf --minimize --ff MMFF94 
which minimises the energy of the molecules stored in an input SDF file using the MMFF94 forcefield 
Use this minimise function to process all matches datasets
Datawarrior descriptor plots
For each network, import the sdf file of matches with their energy minimised
Calculate molecular formula (Chemistry -> From Chemical Structure -> Add Molecular Formula)
Calculate the additional descriptors: monoisotopic mass, cLogP, cLogS, no. H bond acceptors, no. H bond donors, polar surface area and druglikeness (Chemistry -> From Chemical Structure -> Calculate Chemical Properties)
For each descriptor after monoisotopic mass, plot a scatter graph with the descriptor on the y axis and monoisotopic mass on the x axis, with marker colour dependent on generation (creating a legend below the graph)
Additional dataplots in jupyter notebook
Define functions to calculate molecular weight and formula from smiles using the Descriptors and rdMolDescriptors modules and use the apply method to apply these functions to each row of a dataframe of matches (and thus process all 5 networks’ matches)
Define an atom_finder function to calculate the number of atoms of an element passed to it from the molecular formula passed to it
Use the apply method and the atom_finder function to find the number of carbon, hydrogen and oxygen atoms of each molecule for all 5 matches datasets
Define a descriptor plotter function which takes as its arguments the network to plot, the descriptor to plot (against molecular weight) and the no. generations
Define a function which finds the index of the last molecule of each generation 
Plot successively the target descriptor against molecular weight for each generation separately (using the generation marker indexes previously obtained)
Create a legend with G0, G1 … the final generation to plot the target descriptor against molecular weight with molecule generation as the legend 
Create a similar function to produce a Van Krevelen plot with molecule generation as the legend
Define functions to calculate the ratio of oxygen to carbon atoms and hydrogen to carbon atoms
Use the same generations marker index function to successively plot H/C against O/C for each generation for a target network’s matches dataset
Calculate solubility using script from https://github.com/PatWalters/solubility/blob/master/esol.py
Calculate no. heteroatoms using datawarrior
Create plots of ESOL (solubility) and no. heteroatoms against molecular weight

Identifying hits with reference databases:
Define a function which finds the maximum relative molecular mass of all molecules of the matches of a network
Define a function which parses in all SMILES of a reference database for molecules with a relative molecular mass less than a threshold value (set to the maximum relative molecular mass of the matches from a network output)
Define a function which finds all hits with 3 reference databases (HMDB, Kegg, ECMDB) for the matches of a specified network output, returns a dataframe containing number of hits against generation for each database and plots cumulative hits against generation for all 3 databases and the entire matched set

Identifying pathways to produce a matched molecule:
Define a molecule class 
Give the class a self.path attribute (which will store the chain of molecules preceding that molecule in the tree and the molecule itself)
Define a map_tree function which takes the smiles of a target match and the relationships file for the network in which the match is found as its arguments
Read in the rels file and define the base molecules present in G0 of the network
Create a tree data structure with the target smiles (the match) as the root node
Create a while loop which only breaks when all molecules of the deepest layer of the tree are base molecules
Iterate over all nodes of the tree (molecules) and if they are of the deepest layer of the tree and not base molecules, append them to the active nodes list (breaking the loop of the active nodes list is then empty)
Iterate over the list of active nodes and identify the precursors of each molecule using a find_components function and creates a node containing each precursor, with the molecule of the active node as its parent 
The find_components function calls index_finder and precursor_finder functions
The index finder function iterates through the rels files and identifies the index of reactions which produce the target smiles, appending the index of the reaction and its associated deltaG (Gibbs free energy change of reaction) value
The function then iterates through all the possible reactions and identifies the reaction with the lowest deltaG for which none of the reagents, found using the precursor_finder function, are not in the path of the target molecule (to prevent creation of a loop)
The function then returns the index of the lowest deltaG valid reaction
The precursor_finder function iterates through the rels file and identifies the reactions with the index returned by the index_finder function and returns all reagents of that reaction 

