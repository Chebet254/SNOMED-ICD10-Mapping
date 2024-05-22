#Load package
````
require(Rdiagnosislist)
library(tidyverse)
````
#Load UK and International SNOMED CT release files into an
#R environment called 'SNOMED'
````
SNOMED <- loadSNOMED(c(
  './SnomedCT_InternationalRF2_PRODUCTION_20220731T120000Z/',
  './SnomedCT_UKClinicalRF2_PRODUCTION_20210317T000001Z/'))

# Save the 'SNOMED' environment to a file on disk
saveRDS(SNOMED, file = 'mySNOMED.RDS')

# Reload the 'SNOMED' environment from file
SNOMED <- readRDS('mySNOMED.RDS')
#sample
SNOMED <- sampleSNOMED()

#Make sure the SNOMED environment is available and contains the SNOMED dictionary
SNOMEDconcept('Heart F', SNOMED = SNOMED)

#The argument 'exact' can be used to specify whether a regular expression
# search should be done, e.g.
SNOMEDconcept('pancreatitis', exact = FALSE)
````
#The 'description' function can be used to return the descriptions of the concepts found. It returns a data.table with the fully specified  name for each term.
````
description(SNOMEDconcept('Acute pancreatitis', exact = FALSE))

# The 'semantic type' function returns the semantic type of the concept
# from the Fully Specified Name
semanticType(SNOMEDconcept('Heart failure'))

# A list of concepts with a description containing the term 'heart'
# (not that all synonyms are searched, not just the Fully Specified Names)
heart <- SNOMEDconcept('Heart|heart', exact = FALSE, SNOMED = sampleSNOMED())

# A list of concepts containing the term 'fail'
fail <- SNOMEDconcept('Fail|fail', exact = FALSE, SNOMED = sampleSNOMED())

# Concepts with heart and fail
intersect(heart, fail)

# Concepts with heart and not fail
setdiff(heart, fail)

# Concepts with heart or fail
union(heart, fail)
````
#relationships between SNOMED CONCEPTS
````
SNOMED <- sampleSNOMED()

# Parents (immediate ancestors)
parents('Acute heart failure')
# Ancestors
ancestors('Acute heart failure')
# Children (immediate descendants)
children('Acute heart failure')
# Descendants
descendants('Acute heart failure')
````
#Attributes of SNOMED CT concepts
````
# List all the attributes of a concept
att <- attrConcept('Heart failure')
# 'Finding site' of a particular disorder
relatedConcepts('Heart failure', 'Finding site')
# Disorders with a 'Finding site' of 'Heart'
relatedConcepts('Heart', 'Finding site', reverse = TRUE)
````
#SNOMED CODELISTS 
#simple- simple listof concepts, 1 row per concept+2 columns
#tree- hierachichal list with soncept +descendants 
#exptree- descendats are enumerated
````
SNOMED <- sampleSNOMED()
# Create a codelist containing all the descendants of
# the concept 'Heart failure'
my_heart_failure_codelist <- SNOMEDcodelist(
  SNOMEDconcept('Heart failure'), include_desc = TRUE,
  format = 'simple', codelist_name = 'Heart failure')
# Convert to tree format
tree <- SNOMEDcodelist(my_heart_failure_codelist, format = 'tree')

#SNOMED CT simple refsets
# Obtain a list of available refsets with descriptions and counts
refst <- merge(SNOMED$REFSET[, .N, by = list(conceptId = refsetId)],
      SNOMED$DESCRIPTION[, list(conceptId, term)], by = 'conceptId')
# Obtain a refset as a SNOMEDconcept vector
renal_ref <- getRefset('Renal clinical finding simple reference set')

# Find out whether a concept is included in a refset
SNOMEDconcept('Renal failure') %in% renal_ref
````
#Mapping between SNOMED CT and ICD-10 and OPCS4
#SIMPLEMAP- contains mapping to CTV3 &ICD-O
#EXTENDEDMAP- maps to ICD10 & OPCS4
#getmaps function returns a list since there could be multiple entries per SNOMED concept
````
my_heart_failure_codelist <- SNOMEDcodelist(
  SNOMEDconcept('Heart failure'), include_desc = TRUE)

my_heart_failure_codelist <- getMaps(my_heart_failure_codelist, to = c('icd10'))

#gives a list so return one concept per row
single_row <- getMaps(my_heart_failure_codelist, to = c('icd10'), single_row_per_concept = FALSE)
````





