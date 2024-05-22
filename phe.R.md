### ICD10-SNOMED MAPPING
````
require(Rdiagnosislist)
library(readr)
library(tidyverse)
library(dplyr)
library(stringr)
library(bit64)
library(plyr)
````
#load dataset
````
setwd("./OneDrive_1_11-9-2022/")
phe_speciality <- read_csv("./phe_speciality.csv")
phe_map <- read_csv("./phe_map.csv")
phe_dictionary <- read_csv("./phe_dictionary.csv")
ExtendedMapFull <- read_table("./der2_iisssccRefset_ExtendedMapFull_INT_20220731.txt")
#remove decimals/dots in extended map file
ExtendedMapFull$mapTarget <-  gsub('\\.', '', as.character(ExtendedMapFull$mapTarget))
Description_map <- read.table("./sct2_Description_Full-en_INT_20220731.txt", sep = "\t", fill = TRUE, comment.char="", 
                          quote = "\"",header = TRUE, as.is = T, strip.white = T)
Map_with_terms <- full_join(ExtendedMapFull, Description_map, by = c("referencedComponentId" = "conceptId"))
Map_with_terms <- Map_with_terms %>% select(referencedComponentId, mapTarget, term) %>% distinct()
````
#load snomed codes 
````
SNOMED <- loadSNOMED(c(
  './SnomedCT_InternationalRF2_PRODUCTION_20220731T120000Z/',
  './SnomedCT_UKClinicalRF2_PRODUCTION_20210317T000001Z/'))
````
#merge phe map and dictionary
````
phe_map_and_dictionary <- left_join(phe_map, phe_dictionary)
````
#filter gastroenterology & hepatology
````
gastro <-  phe_map_and_dictionary %>% filter(spec_01 == 'Gastroenterology') %>% filter(subspec_01 == 'Hepatology') %>% 
  filter(type == 'Disease') 
````

### OUTPUT ONE
````
gastro_one <- gastro
gastro_one <- gastro_one %>% select(icd10, phecode, phenotype)
#write.csv(gastro_one, "Hepatology Phenotypes.csv")
````
#remove X from icd10 
````
gastro_one$icd10 <- gsub("X","",as.character(gastro_one$icd10))
#finding icd10 in extended map to find referenceid
searched_df <- Map_with_terms %>% filter(mapTarget %in% gastro_one$icd10)
colnames(searched_df) <- c("referencedComponentId", "mapTarget", "ExtendedMapTerm")
#remove (disorder)
searched_df$ExtendedMapTerm <-  str_replace(searched_df$ExtendedMapTerm, " \\s*\\([^\\)]+\\)", "")
searched_df <- searched_df %>% distinct()
output_one <- left_join(gastro_one, searched_df, by = c('icd10' = 'mapTarget')) %>% distinct()
#missing icd10 -csv saved after finding missing codes in descendants
missing_in_extendedmap <- output_one[is.na(output_one$referencedComponentId), ]
output_one <- output_one[!is.na(output_one$referencedComponentId), ] #remove NA snomed codes
#write.csv(output_one, "Output1.csv")
````
### OUTPUT TWO
````
output_one$referencedComponentId <- as.integer64(output_one$referencedComponentId)
for (i in 1:length(output_one)) { #x =1  
  codelist_one = SNOMEDcodelist(SNOMEDconcept(output_one$referencedComponentId))
  codelist_one= getMaps(codelist_one, to = c('icd10'), single_row_per_concept = FALSE) #one to one mapping
}

colnames(codelist_one) <- c("conceptId", "ICD10_code", "SNOMEDTerm")
output_two <- left_join(output_one, codelist_one, by = c('referencedComponentId' = 'conceptId')) %>% distinct()
str(output_two)
output_two$referencedComponentId <- as.character(output_two$referencedComponentId)
#write.csv(output_two, "Output2.csv")
````

#add descendants output 1b
````
output_one$referencedComponentId <- as.integer64(output_one$referencedComponentId)
for (i in 1:length(output_one)) { #x =1
  output_one_codelist = SNOMEDcodelist(SNOMEDconcept(output_one$referencedComponentId), include_desc = TRUE)
  output_one_desc = getMaps(output_one_codelist, to = c('icd10'), single_row_per_concept = FALSE) #one to one mapping
}

#join with descendants
output_one_desc$Snomed_rship <- 'Descendant'
output_one_desc$Snomed_rship[which(output_one_desc$conceptId  %in% output_one$referencedComponentId )] <- 'Concept'
#output_one_desc$conceptId <- as.character(output_one_desc$conceptId)
#write.csv(output_one_desc, "Output_one_desc.csv")
view(output_one_desc)


#add descendants output 2 -similar to ouput one desc
output_two$referencedComponentId <- as.integer64(output_two$referencedComponentId)
for (i in 1:length(output_two)) { #x =1
  output_two_codelist = SNOMEDcodelist(SNOMEDconcept(output_two$referencedComponentId), include_desc = TRUE)
  output_two_desc = getMaps(output_two_codelist, to = c('icd10'), single_row_per_concept = FALSE)
}

#descendants(df$referencedComponentId)
output_two_desc$relationship <- 'Descendant'
output_two_desc$relationship[which(output_two$ICD10_code %in% output_two_desc$icd10_code )] <- 'Concept'
#write.csv(output_2b, "Output2b.csv")
````
#find missing icd10 codes
````
missing_in_extendedmap$missing_icd <- ''
missing_in_extendedmap$missing_icd[which(missing_in_extendedmap$icd10 %in% output_one_desc$icd10_code )] <- 'Found_in_Desc'
#write.csv(missing_in_extendedmap, "Missing ICD 10.csv")
````
#count
````
length(unique(gastro_one$phecode))
length(unique(gastro_one$icd10))
#output1: 
length(unique(output_one$referencedComponentId))
length(unique(output_one$icd10))
length(unique(output_one$phecode))
length(unique(missing_in_extendedmap$icd10))
length(unique(output_one_desc$conceptId))

#output1desc
length(unique(output_one_desc$conceptId))
length(unique(output_one_desc$icd10))
length(unique(output_1c$conceptId))
#output 2
length(unique(output_two$referencedComponentId))
length(unique(output_two$ICD10_code))
length(unique(output_two_desc$conceptId))
length(unique(output_two_desc$icd10_code))
````

