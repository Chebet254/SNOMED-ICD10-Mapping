# Load package
````
require(Rdiagnosislist)
library(tidyverse)
````
### Load UK and International SNOMED CT release files into an R environment called 'SNOMED'
````
SNOMED <- loadSNOMED(c(
  './SnomedCT_InternationalRF2_PRODUCTION_20220731T120000Z/',
  './SnomedCT_UKClinicalRF2_PRODUCTION_20210317T000001Z/'))
````
# Save the 'SNOMED' environment to a file on disk
````
saveRDS(SNOMED, file = 'mySNOMED.RDS')
#sample
#SNOMED <- sampleSNOMED()
````
## 1 ACUTE PANCREATITIS 
````
SNOMEDconcept('Acute pancreatitis', exact = FALSE)

#mapping to ICD 
Acute_P$referencedComponentId
pancreatitis_codelist <- SNOMEDcodelist(
  SNOMEDconcept(c('53616007',  '197458008', '1197669006', '303002',  '197460005', '863978006',
                  '15528006',  '235960001',  '722544004', '59154002',  '721724009',
                  '19742005',  '197456007',  '197456007')), include_desc = TRUE)


pancreatitis_codelist <- getMaps(pancreatitis_codelist, to = c('icd10'))
#gives a list so return one concept per row
pancreatitis_codelist <- getMaps(pancreatitis_codelist, to = c('icd10'), single_row_per_concept = FALSE)
````
#filter k859 only
````
pancreatitis_codelist <- pancreatitis_codelist %>% filter(icd10_code == 'K859')
#write.csv(pancreatitis_codelist, "Pancreatitis.csv")
````
## 2. HEPATIC FAILURE
````
SNOMEDconcept('Hepatic failure', exact = FALSE)

#mapping to ICD 
hepatic_failure_codelist <- SNOMEDcodelist(
  SNOMEDconcept('Hepatic failure'), include_desc = TRUE)

hepatic_failure_codelist <- getMaps(hepatic_failure_codelist, to = c('icd10'))

#gives a list so return one concept per row
hepatic_failure_codelist <- getMaps(hepatic_failure_codelist, to = c('icd10'), single_row_per_concept = FALSE)

#filter k859 only
hepatic_failure_codelist <- hepatic_failure_codelist %>% filter(icd10_code == 'K704')
write.csv(hepatic_failure_codelist, "Hepatic-Failure.csv")
````
## 3. NECROSIS 
#no concept for both
````
SNOMEDconcept('Acute necrosis of liver', exact = FALSE)

#mapping to ICD 
acute_necrosis_codelist <- SNOMEDcodelist(
  SNOMEDconcept('Acute necrosis of liver'), include_desc = TRUE)

Acute_necrosis_codelist <- getMaps(acute_necrosis_codelist, to = c('icd10'))

#gives a list so return one concept per row
Acute_necrosis_codelist <- getMaps(Acute_necrosis_codelist, to = c('icd10'), single_row_per_concept = FALSE)

#filter k762 only
Acute_necrosis_codelist <- Acute_necrosis_codelist %>% filter(icd10_code == 'K762')
#write.csv(Acute_necrosis_codelist, "Necrosis.csv")
````
## 4. HYPERTROPHY
#combined ulcer
#ulcer$referencedComponentId
#mapping to ICD
````
ulcer_codelist <- SNOMEDcodelist(
  SNOMEDconcept(c('197453004',  '235918000',  '235918000',  '235976005',  '235928009',
                  '1234822009',  '722882004',  '235927004',  '123608004',  '721720000',
                  '308863004',  '308863004',   '10518000',  '860789005',   '66556007',
                  '235930006',   '53192009',  '726613003')), include_desc = TRUE)

ulcer_codelist <- getMaps(ulcer_codelist, to = c('icd10'))
ulcer_codelist <- getMaps(ulcer_codelist, to = c('icd10'), single_row_per_concept = FALSE)
ulcer_codelist <- ulcer_codelist %>% filter(icd10_code == 'K838') 
````
#codes from phe.r that might be useful later
````
#gastro <- gastphe_map_and_dictionary <- merge(phe_map, phe_dictionary, by = 'phecode')
#ro %>% select(c(1:4,9:11,category))
#write.csv(gastro, "Gastro.csv")
#gastro[1,1:2]
#ExtendedMapFull %>% select(gastro[1,2])
````
#acute pancreatitis
````
pancreatitis <- gastro %>% filter(phenotype == 'Acute pancreatitis')
#hepatic failure
hepatic_failure <- gastro %>% filter(phenotype == 'Hepatic failure')
````
#acute pancreatitis from extended maps
````
Acute_P <- ExtendedMapFull %>%  filter(mapTarget == 'K85.9')
Acute_P$referencedComponentId <- as.character(Acute_P$referencedComponentId)
pancreatitis_codelist$conceptId <- as.character(pancreatitis_codelist$conceptId)
Acute_P <- left_join(Acute_P, pancreatitis_codelist, by = c('referencedComponentId' = 'conceptId')) 
Acute_P <- left_join(Acute_P, phe_map, by = c("icd10_code" = "icd10"))
Acute_P <- Acute_P %>% select(phecode, mapTarget, term, referencedComponentId)
#export
#write.csv(Acute_P, "Acute_Pancreatitis.csv")
````
#hepatic failure 
````
hepatic_f <- ExtendedMapFull %>% filter(mapTarget == 'K70.4')
hepatic_f$referencedComponentId <-as.character(hepatic_f$referencedComponentId) 
hepatic_failure_codelist$conceptId <- as.character(hepatic_failure_codelist$conceptId)
hepatic_f <- left_join(hepatic_f, hepatic_failure_codelist,by = c('referencedComponentId' = 'conceptId'))
hepatic_f <- left_join(hepatic_f, phe_map, by = c("icd10_code" = "icd10"))
hepatic_f <- hepatic_f %>% select(phecode, mapTarget, term, referencedComponentId)
#write.csv(hepatic_f, "Hepatic_f.csv")
````
#acute and subacute necrosis
````
gastro %>% filter(phenotype == 'Acute and subacute necrosis of liver')
Acute_necrosis <- ExtendedMapFull %>% filter(mapTarget == 'K76.2')
Acute_necrosis$referencedComponentId <- as.character(Acute_necrosis$referencedComponentId)
Acute_necrosis_codelist$conceptId <- as.character(Acute_necrosis_codelist$conceptId)
Acute_necrosis <- left_join(Acute_necrosis, Acute_necrosis_codelist, by = c('referencedComponentId' = 'conceptId'))
Acute_necrosis <- left_join(Acute_necrosis, phe_map, by = c("icd10_code" = "icd10"))
Acute_necrosis <- Acute_necrosis %>% select(phecode, mapTarget, term, referencedComponentId)
#write.csv(Acute_necrosis, "Necrosis.csv")
````
#hypertrophy 
````
gastro %>% filter(phenotype == 'Adhesions Atrophy Hypertrophy Ulcer of bile duct')
ulcer <- ExtendedMapFull %>% filter(mapTarget == 'K83.8')
ulcer$referencedComponentId <- as.character(ulcer$referencedComponentId)
ulcer_codelist$conceptId <- as.character(ulcer_codelist$conceptId)
ulcer <- left_join(ulcer, ulcer_codelist, by = c('referencedComponentId' = 'conceptId'))
ulcer <- left_join(ulcer, phe_map,by = c("icd10_code" = "icd10"))
ulcer <- ulcer %>% select(phecode, mapTarget, term, referencedComponentId) 
write.csv(ulcer, "Ulcer.csv")
````
#subset
#Using dplyr::filter
````
df <- dplyr::filter(ExtendedMapFull, mapTarget %in% c("K76.2", "K70.4", "K83.8", 'K85.9'))
````
#descendants 
````
codelist_one_desc$relationship <- 'Descendants'
codelist_one_desc$relationship[which(codelist_one$conceptId %in% codelist_one_desc$conceptId)] <- 'Concept'
write.csv(codelist_one_desc, "codelist_one_desc.csv")
````
#code 2
gastro[3,2]
gastro_two <- ExtendedMapFull %>% filter(mapTarget == 'D01.5')

#for loop to select individual phecodes 
````
df$referencedComponentId <- as.character(df$referencedComponentId)

for (x in 1:length(df)) { #x =1  
  codelist = SNOMEDcodelist(SNOMEDconcept(df$referencedComponentId))
  codelist_desc = SNOMEDcodelist(SNOMEDconcept(df$referencedComponentId), include_desc = TRUE)
  codelist = getMaps(codelist, to = c('icd10'), single_row_per_concept = FALSE) #one to one mapping
  codelist_desc = getMaps(codelist_desc, to = c('icd10'), single_row_per_concept = FALSE)
}
code
#descendants(df$referencedComponentId)
codelist_desc$relationship <- 'Descendants'
codelist_desc$relationship[which(codelist$conceptId %in% codelist_desc$conceptId )] <- 'Concept'
#join with phecode
codelist <- left_join(codelist, phe_map, by= c("icd10_code" = "icd10"))
codelist_desc <- left_join(codelist_desc, phe_map,by =  c("icd10_code" = "icd10"))
#join to get phenotype
codelist <- left_join(codelist, gastro, by = 'phecode') 
codelist <- codelist %>% select(phecode, phenotype, conceptId, icd10_code, term)%>% unique()
codelist_desc <- left_join(codelist_desc, gastro, by = 'phecode')
````
## PART 2
searched_df$referencedComponentId <- as.character(searched_df$referencedComponentId)
for (i in 1:length(searched_df)) { #x =1  
  codelist_one = SNOMEDcodelist(SNOMEDconcept(searched_df$referencedComponentId))
  codelist_one_desc = SNOMEDcodelist(SNOMEDconcept(searched_df$referencedComponentId), include_desc = TRUE)
  codelist_one = getMaps(codelist_one, to = c('icd10'), single_row_per_concept = FALSE) #one to one mapping
  codelist_one_desc = getMaps(codelist_one_desc, to = c('icd10'), single_row_per_concept = FALSE)
}

#write.csv(codelist_one, "D:/HDR UK/IHI/OneDrive_1_11-9-2022/IHI/codelist_one.csv")
#left join to get phecode
phe_map_and_dictionary <- phe_map_and_dictionary[ , 1:3]
str(phe_map_and_dictionary)
output_two <- left_join(codelist_one, phe_map_and_dictionary, by = c('icd10_code' = 'icd10'))
output_two <- output_two[,1:5 ]
````


 
