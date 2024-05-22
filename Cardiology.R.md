## CARDIOLOGY
#filter disease
````
cardiology <-  phe_map_and_dictionary %>% filter(spec_01 == 'Cardiology')  %>% 
  filter(type == 'Disease') 
````
## PART 1 -OUTPUT ONE
````
cardiology <- cardiology %>% select(icd10, phecode, phenotype)
#write.csv(cardiology, "D:/HDR UK/IHI/OneDrive_1_11-9-2022/IHI/Cardiology Phenotypes.csv")
#remove X from icd10 
cardiology$icd10 <- gsub("X","",as.character(cardiology$icd10))
````

### finding icd10 in extended map to find referenceid
````
cardiology_searched_df <- Map_with_terms %>% filter(mapTarget %in% cardiology$icd10)
colnames(cardiology_searched_df) <- c("referencedComponentId", "mapTarget", "ExtendedMapTerm")
#remove (disorder)
cardiology_searched_df$ExtendedMapTerm <-  str_replace(cardiology_searched_df$ExtendedMapTerm, " \\s*\\([^\\)]+\\)", "")
cardiology_searched_df <- cardiology_searched_df %>% distinct()
output_one_cardiology <- left_join(cardiology, cardiology_searched_df, by = c('icd10' = 'mapTarget')) %>% distinct()
#missing icd10
missing_cardiology <- output_one_cardiology[is.na(output_one_cardiology$referencedComponentId), ]
output_one_cardiology <- output_one_cardiology[!is.na(output_one_cardiology$referencedComponentId), ] #remove NA snomed codes 
#write.csv(output_one_cardiology, "D:/HDR UK/IHI/OneDrive_1_11-9-2022/IHI/Output1_Cardiology.csv")
````
## OUTPUT 2
````
output_one_cardiology$referencedComponentId <- as.integer64(output_one_cardiology$referencedComponentId)
for (i in 1:length(output_one_cardiology)) { #x =1  
  codelist_one_cardiology = SNOMEDcodelist(SNOMEDconcept(output_one_cardiology$referencedComponentId))
  codelist_one_cardiology= getMaps(codelist_one_cardiology, to = c('icd10'), single_row_per_concept = FALSE) #one to one mapping
}

colnames(codelist_one_cardiology) <- c("conceptId", "ICD10_code", "SNOMEDTerm")
output_two_cardiology <- left_join(output_one_cardiology, codelist_one_cardiology, by = c('referencedComponentId' = 'conceptId')) %>% distinct()
#write.csv(output_two_cardiology, "D:/HDR UK/IHI/OneDrive_1_11-9-2022/IHI/Output2_cardiology.csv")

#output 1 desc
for (i in 1:length(output_one_cardiology)) { #x =1
  output_one_cardiology_codelist = SNOMEDcodelist(SNOMEDconcept(output_one_cardiology$referencedComponentId), include_desc = TRUE)
  output_one_cardiology_desc = getMaps(output_one_cardiology_codelist, to = c('icd10'), single_row_per_concept = FALSE) #one to one mapping
}
````

#join with descendants
````
output_one_cardiology_desc$Snomed_rship <- 'Descendant'
output_one_cardiology_desc$Snomed_rship[which(output_one_cardiology_desc$conceptId  %in% output_one_cardiology$referencedComponentId )] <- 'Concept'
write.csv(output_one_cardiology_desc, "D:/HDR UK/IHI/OneDrive_1_11-9-2022/IHI/Output_1b_cardiology.csv")
````

#output 2 desc #sanity check-output 1b
````
for (i in 1:length(output_two_cardiology)) { #x =1
  output_two_cardiology_codelist = SNOMEDcodelist(SNOMEDconcept(output_two_cardiology$referencedComponentId), include_desc = TRUE)
  output_two_cardiology_desc = getMaps(output_two_cardiology_codelist, to = c('icd10'), single_row_per_concept = FALSE)
}
#descendants(df$referencedComponentId)
output_two_cardiology_desc$relationship <- 'Descendant'
output_two_cardiology_desc$relationship[which(output_two_cardiology$ICD10_code %in% output_two_cardiology_desc$icd10_code )] <- 'Concept'
#write.csv(output_2b, "D:/HDR UK/IHI/OneDrive_1_11-9-2022/IHI/Output2b.csv")

#find missing icd10 codes
missing_cardiology$missing_icd <- ''
missing_cardiology$missing_icd[which(missing_cardiology$icd10 %in% output_one_cardiology_desc$icd10_code )] <- 'Found_in_Desc'
m <- missing_cardiology %>% filter(missing_icd == 'Found_in_Desc')
write.csv(missing_cardiology, "D:/HDR UK/IHI/OneDrive_1_11-9-2022/IHI/Missing Cardiology.csv")
````
### summaries 
````
# n unique phecodes (phe_dictionary and phe_map)
length(unique(cardiology$phecode))

# n unique icd10 codes (phe_dictionary and phe_map)
length(unique(cardiology$icd10))

# n unique snomed ids (output 1)
length(unique(output_one_cardiology$referencedComponentId))
# # n unique icd10 codes (output 1)
length(unique(output_one_cardiology$icd10))
# missing icd10 codes (output 1) 
length(unique(missing_cardiology$icd10))

# n unique icd10 codes (output2)
length(unique(output_two_cardiology$ICD10_code))

# n unique snomed ids (output 2)
length(unique(output_two_cardiology$referencedComponentId))

length(unique(output_one_cardiology_desc$conceptId))
length(unique(output_one_cardiology_desc$icd10_code))
length(unique(output_two_cardiology_desc$conceptId))
length(unique(output_two_cardiology_desc$icd10_code))
````
