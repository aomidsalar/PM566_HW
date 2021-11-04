---
title: "Homework 3"
author: "Audrey Omidsalar"
date: "11/5/2021"
output:
  html_document:
    toc: yes
    toc_float: yes
    keep_md: yes
  github_document:
  always_allow_html: true
---



# APIs

### Look for papers that show up under the term *sars-cov-2 trial vaccine*


```r
# Downloading the website
website <- read_html(x = "https://pubmed.ncbi.nlm.nih.gov/?term=sars-cov-2+trial+vaccine")

# Finding the counts
counts <- xml2::xml_find_first(website, "/html/body/main/div[9]/div[2]/div[2]/div[1]")

# Turning it into text
counts <- as.character(counts)

# Extracting the data using regex
numberpapers  <- stringr::str_extract(counts, "[:digit:]+.*[:digit:]")
```

There are 2,329 papers with the search term *sars-cov-2 trial vaccine*

### Download each paper's abstracts


```r
query_ids <- GET(
  url   = "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esearch.fcgi",
  query = list(
    db = "pubmed",
    term = "sars-cov-2 trial vaccine",
    retmax = 1000
  )
)
# Extracting the content of the response of GET
ids <- httr::content(query_ids)
# Turn the result into a character vector
ids <- as.character(ids)

# Find all the ids 
ids <- stringr::str_extract_all(ids, "<Id>[:digit:]+</Id>")[[1]]

# Remove all the leading and trailing <Id> </Id>. Make use of "|"
ids <- stringr::str_remove_all(ids, "<Id>|</Id>")
# Take first 250 ids
ids <- ids[1:250]
```


```r
publications <- GET(
  url   = "https://eutils.ncbi.nlm.nih.gov/entrez/eutils/efetch.fcgi",
  query = list(
    db = "pubmed",
    id = I(paste(ids, collapse = ",")),
    retmax = 1000,
    rettype = "abstract"
    )
)
# Turning the output into character vector
publications <- httr::content(publications)
publications_txt <- as.character(publications)
```


```r
# publication list
pub_char_list <- xml2::xml_children(publications)
pub_char_list <- sapply(pub_char_list, as.character)
# extracting titles
titles <- str_extract(pub_char_list, "<ArticleTitle>[[:print:][:space:]]+</ArticleTitle>")
titles <- str_remove_all(titles, "</?[[:alnum:]-=\"]+>")
# extracting abstracts
abstracts <- str_extract(pub_char_list, "<Abstract>[[:print:][:space:]]+</Abstract>")
#abstracts <- str_extract(pub_char_list, "<Abstract>(\\n|.)+</Abstract>")
abstracts <- str_remove_all(abstracts, "</?[[:alnum:]]+>")
abstracts <- str_replace_all(abstracts, "\\s+", " ")
# extracting journal name
journalname <- str_extract(pub_char_list, "<Title>[[:print:][:space:]]+</Title>")
#journalname <- str_extract(pub_char_list, "<Title>(\\n|.)+</Title>")
journalname <- str_remove_all(journalname, "</?[[:alnum:]]+>")
journalname <- str_replace_all(journalname, "\\s+", " ")
# extracting publication date
pubdate <- str_extract(pub_char_list, "<PubDate>[[:print:][:space:]]+</PubDate>")
#pubdate <- str_extract(pub_char_list, "<PubDate>(\\n|.)+</PubDate>")
pubdate <- str_remove_all(pubdate, "</?[[:alnum:]]+>")
pubdate <- str_replace_all(pubdate, "\\s+", " ")
##how many missing abstracts are there?
table(is.na(abstracts))
```

```
## 
## FALSE  TRUE 
##   218    32
```


```r
database <- data.frame(
  PubMedId = ids,
  Title = titles,
  Name_of_Journal = journalname,
  Publication_Date = pubdate,
  Abstract = abstracts
)
knitr::kable(database[1:10,], caption = "Some Papers on PubMed about SARS-CoV2 Trial Vaccines")
```



Table: Some Papers on PubMed about SARS-CoV2 Trial Vaccines

|PubMedId |Title                                                                                                                                              |Name_of_Journal                                                                                      |Publication_Date |Abstract                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
|:--------|:--------------------------------------------------------------------------------------------------------------------------------------------------|:----------------------------------------------------------------------------------------------------|:----------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|34729549 |Adverse events of active and placebo groups in SARS-CoV-2 vaccine randomized trials: A systematic review.                                          |The Lancet regional health. Europe                                                                   |2021 Oct 28      |<AbstractText Label="Background" NlmCategory="UNASSIGNED">For safety assessment in clinical trials, adverse events (AEs) are reported for the drug under evaluation and compared with AEs in the placebo group. Little is known about the nature of the AEs associated with clinical trials of SARS-CoV-2 vaccines and the extent to which these can be traced to nocebo effects, where negative treatment-related expectations favor their occurrence. <AbstractText Label="Methods" NlmCategory="UNASSIGNED">In our systematic review, we compared the rates of solicited AEs in the active and placebo groups of SARS-CoV-2 vaccines approved by the Western pharmaceutical regulatory agencies.We implemented a search strategy to identify trial-III studies of SARS-CoV-2 vaccines through the PubMed database. We adopted the PRISMA Statement to perform the study selection and the data collection and identified three trial: two mRNA-based (37590 participants) and one adenovirus type (6736 participants). <AbstractText Label="Findings" NlmCategory="UNASSIGNED">Relative risks showed that the occurrence of AEs reported in the vaccine groups was higher compared with the placebo groups. The most frequently AEs in both groups were fatigue, headache, local pain, as injection site reactions, and myalgia. In particular, for first doses in placebo recipients, fatigue was reported in 29% and 27% in BNT162b2 and mRNA-1273 groups, respectively, and in 21% of Ad26.COV2.S participants. Headache was reported in 27% in both mRNA groups and in 24% of Ad26.COV2.S recipients. Myalgia was reported in 10% and 14% in mRNA groups (BNT162b2 and mRNA-1273, respectively) and in 13% of Ad26.COV2.S participants. Local pain was reported in 12% and 17% in mRNA groups (BNT162b2 and mRNA-1273, respectively), and in 17% of Ad26.COV2.S recipients. These AEs are more common in the younger population and in the first dose of placebo recipients of the mRNA vaccines. <AbstractText Label="Interpretation" NlmCategory="UNASSIGNED">Our results are in agreement with the expectancy theory of nocebo effects and suggest that the AEs associated with COVID-19 vaccines may be related to the nocebo effect. <AbstractText Label="Funding" NlmCategory="UNASSIGNED">Fondazione CRT - Cassa di Risparmio di Torino, IT (grant number 66346, "GAIA-MENTE" 2019). © 2021 The Authors.                                                                                                                                                                                                                                                                                                                           |
|34726743 |Analysis of the Effectiveness of the Ad26.COV2.S Adenoviral Vector Vaccine for Preventing COVID-19.                                                |JAMA network open                                                                                    |2021 Nov 01      |<AbstractText Label="Importance" NlmCategory="UNASSIGNED">Continuous assessment of the effectiveness and safety of the US Food and Drug Administration-authorized SARS-CoV-2 vaccines is critical to amplify transparency, build public trust, and ultimately improve overall health outcomes. <AbstractText Label="Objective" NlmCategory="UNASSIGNED">To evaluate the effectiveness of the Johnson &amp; Johnson Ad26.COV2.S vaccine for preventing SARS-CoV-2 infection. <AbstractText Label="Design, Setting, and Participants" NlmCategory="UNASSIGNED">This comparative effectiveness research study used large-scale longitudinal curation of electronic health records from the multistate Mayo Clinic Health System (Minnesota, Arizona, Florida, Wisconsin, and Iowa) to identify vaccinated and unvaccinated adults between February 27 and July 22, 2021. The unvaccinated cohort was matched on a propensity score derived from age, sex, zip code, race, ethnicity, and previous number of SARS-CoV-2 polymerase chain reaction tests. The final study cohort consisted of 8889 patients in the vaccinated group and 88 898 unvaccinated matched patients. <AbstractText Label="Exposure" NlmCategory="UNASSIGNED">Single dose of the Ad26.COV2.S vaccine. <AbstractText Label="Main Outcomes and Measures" NlmCategory="UNASSIGNED">The incidence rate ratio of SARS-CoV-2 infection in the vaccinated vs unvaccinated control cohorts, measured by SARS-CoV-2 polymerase chain reaction testing. <AbstractText Label="Results" NlmCategory="UNASSIGNED">The study was composed of 8889 vaccinated patients (4491 men [50.5%]; mean [SD] age, 52.4 [16.9] years) and 88 898 unvaccinated patients (44 748 men [50.3%]; mean [SD] age, 51.7 [16.7] years). The incidence rate ratio of SARS-CoV-2 infection in the vaccinated vs unvaccinated control cohorts was 0.26 (95% CI, 0.20-0.34) (60 of 8889 vaccinated patients vs 2236 of 88 898 unvaccinated individuals), which corresponds to an effectiveness of 73.6% (95% CI, 65.9%-79.9%) and a 3.73-fold reduction in SARS-CoV-2 infections. <AbstractText Label="Conclusions and Relevance" NlmCategory="UNASSIGNED">This study's findings are consistent with the clinical trial-reported efficacy of Ad26.COV2.S and the first retrospective analysis, suggesting that the vaccine is effective at reducing SARS-CoV-2 infection, even with the spread of variants such as Alpha or Delta that were not present in the original studies, and reaffirm the urgent need to continue mass vaccination efforts globally.                                                                                                                                                        |
|34715931 |Lessons from Israel's COVID-19 Green Pass program.                                                                                                 |Israel journal of health policy research                                                             |2021 10 29       |As of the beginning of March 2021, Israeli law requires the presentation of a Green Pass as a precondition for entering certain businesses and public spheres. Entitlement for a Green Pass is granted to Israelis who have been vaccinated with two doses of COVID-19 vaccine, who have recovered from COVID-19, or who are participating in a clinical trial for vaccine development in Israel. The Green Pass is essential for retaining immune individuals' freedom of movement and for promoting the public interest in reopening the economic, educational, and cultural spheres of activity. Nonetheless, and as the Green Pass imposes restrictions on the movement of individuals who had not been vaccinated or who had not recovered, it is not consonant with solidarity and trust building. Implementing the Green Pass provision while advancing its effectiveness on the one hand, and safeguarding equality, proportionality, and fairness on the other hand may imbue this measure with ethical legitimacy despite involving a potential breach of trust and solidarity. © 2021. The Author(s).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
|34713912 |Vaccine development and technology for SARS-CoV-2: current insights.                                                                               |Journal of medical virology                                                                          |2021 Oct 29      |<AbstractText Label="BACKGROUND" NlmCategory="BACKGROUND">SARS-CoV-2 is associated to a severe respiratory disease in China, that rapidly spread across continents. Since the beginning of the pandemic, available data suggested the asymptomatic transmission and patients were treated with specific drugs with efficacy and safety data not always satisfactory. <AbstractText Label="OBJECTIVES" NlmCategory="OBJECTIVE">The aim of this review is to describe the vaccines developed by three companies, Pfizer-BioNTech, Moderna and University of Oxford/AstraZeneca, in terms of both technological and pharmaceutical formulation, safety, efficacy and immunogenicity. <AbstractText Label="METHODS" NlmCategory="METHODS">A critical analysis of phase 1, 2 and 3 clinical trial results available was conducted, comparing the three vaccine candidates, underlining their similarities and differences. <AbstractText Label="RESULTS AND CONCLUSIONS" NlmCategory="CONCLUSIONS">All candidates showed consistent efficacy and tolerability; although some differences can be noted, such as their technological formulation, temperature storage, which will be related to logistics and costs. Further studies will be necessary to evaluate long-term effects and to assess the vaccine safety and efficacy in the general population. This article is protected by copyright. All rights reserved. This article is protected by copyright. All rights reserved.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
|34711598 |BCG vaccination to reduce the impact of COVID-19 in healthcare workers: Protocol for a randomised controlled trial (BRACE trial).                  |BMJ open                                                                                             |2021 10 28       |<AbstractText Label="INTRODUCTION">BCG vaccination modulates immune responses to unrelated pathogens. This off-target effect could reduce the impact of emerging pathogens. As a readily available, inexpensive intervention that has a well-established safety profile, BCG is a good candidate for protecting healthcare workers (HCWs) and other vulnerable groups against COVID-19. <AbstractText Label="METHODS AND ANALYSIS">This international multicentre phase III randomised controlled trial aims to determine if BCG vaccination reduces the incidence of symptomatic and severe COVID-19 at 6 months (co-primary outcomes) compared with no BCG vaccination. We plan to randomise 10 078 HCWs from Australia, The Netherlands, Spain, the UK and Brazil in a 1:1 ratio to BCG vaccination or no BCG (control group). The participants will be followed for 1 year with questionnaires and collection of blood samples. For any episode of illness, clinical details will be collected daily, and the participant will be tested for SARS-CoV-2 infection. The secondary objectives are to determine if BCG vaccination reduces the rate, incidence, and severity of any febrile or respiratory illness (including SARS-CoV-2), as well as work absenteeism. The safety of BCG vaccination in HCWs will also be evaluated. Immunological analyses will assess changes in the immune system following vaccination, and identify factors associated with susceptibility to or protection against SARS-CoV-2 and other infections. <AbstractText Label="ETHICS AND DISSEMINATION">Ethical and governance approval will be obtained from participating sites. Results will be published in peer-reviewed open-access journals. The final cleaned and locked database will be deposited in a data sharing repository archiving system. <AbstractText Label="TRIAL REGISTRATION">ClinicalTrials.gov NCT04327206. © Author(s) (or their employer(s)) 2021. Re-use permitted under CC BY. Published by BMJ.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
|34704204 |COVID-19 Testing and Vaccine Acceptability Among Homeless-Experienced Adults: Qualitative Data from Two Samples.                                   |Journal of general internal medicine                                                                 |2021 Oct 26      |<AbstractText Label="BACKGROUND" NlmCategory="BACKGROUND">Homeless-experienced populations are at increased risk of exposure to SARS-CoV-2 due to their living environments and face an increased risk of severe COVID-19 disease due to underlying health conditions. Little is known about COVID-19 testing and vaccination acceptability among homeless-experienced populations. <AbstractText Label="OBJECTIVE" NlmCategory="OBJECTIVE">To understand the facilitators and barriers to COVID-19 testing and vaccine acceptability among homeless-experienced adults. <AbstractText Label="DESIGN" NlmCategory="METHODS">We conducted in-depth interviews with participants from July to October 2020. We purposively recruited participants from (1) a longitudinal cohort of homeless-experienced older adults in Oakland, CA (n=37) and (2) a convenience sample of people (n=57) during a mobile outreach COVID-19 testing event in San Francisco. <AbstractText Label="PARTICIPANTS" NlmCategory="METHODS">Adults with current or past experience of homelessness. <AbstractText Label="APPROACH" NlmCategory="METHODS">We asked participants about their experiences with and attitudes towards COVID-19 testing and their perceptions of COVID-19 vaccinations. We used participant observation techniques to document the interactions between testing teams and those approached for testing. We audio-recorded, transcribed, and content analyzed all interviews and identified major themes and subthemes. <AbstractText Label="KEY RESULTS" NlmCategory="RESULTS">Participants found incentivized COVID-19 testing administered in unsheltered settings and supported by community health outreach workers (CHOWs) to be acceptable. The majority of participants expressed a positive inclination toward vaccine acceptability, citing a desire to return to routine life and civic responsibility. Those who expressed hesitancy cited a desire to see trial data, concerns that vaccines included infectious materials, and mistrust of the government. <AbstractText Label="CONCLUSIONS" NlmCategory="CONCLUSIONS">Participants expressed positive evaluations of the incentivized, mobile COVID-19 testing supported by CHOWs in unsheltered settings. The majority of participants expressed a positive inclination toward vaccination. Vaccine hesitancy concerns must be addressed when designing vaccine delivery strategies that overcome access challenges. Based on the successful implementation of COVID-19 testing, we recommend mobile delivery of vaccines using trusted CHOWs to address concerns and facilitate wider access to and uptake of the COVID vaccine. © 2021. Society of General Internal Medicine. |
|34703690 |A Rare Variant of Guillain-Barre Syndrome Following Ad26.COV2.S Vaccination.                                                                       |Cureus                                                                                               |2021 Sep         |Efforts to combat the global pandemic caused by severe acute respiratory syndrome coronavirus 2 (SARS-CoV-2) range from adequate diagnostic testing and contract tracing to vaccination for the prevention of coronavirus disease 2019 (COVID-19). In the United States alone, three vaccinations have been authorized for emergency use (EUA) or approved to prevent COVID-19. The Ad26.COV2.S vaccine by Johnson and Johnson (New Brunswick, New Jersey) is the only adenovirus-based vaccine and deemed relatively effective and safe by the US Food and Drug Administration (FDA) following its clinical trial. Since its introduction, the US FDA has placed a warning on the vaccine adverse event reporting system (VAERS) after more than 100 cases of Guillain-Barre Syndrome (GBS) were reported. Herein, we outline the hospital course of a generally healthy 49-year-old female who experienced an axonal form of GBS nine days after receiving the Ad26.COV2.S vaccine. Copyright © 2021, Morehouse et al.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
|34702753 |Humoral immunogenicity of the seasonal influenza vaccine before and after CAR-T-cell therapy: a prospective observational study.                   |Journal for immunotherapy of cancer                                                                  |2021 10          |Recipients of chimeric antigen receptor-modified T (CAR-T) cell therapies for B cell malignancies have profound and prolonged immunodeficiencies and are at risk for serious infections, including respiratory virus infections. Vaccination may be important for infection prevention, but there are limited data on vaccine immunogenicity in this population. We conducted a prospective observational study of the humoral immunogenicity of commercially available 2019-2020 inactivated influenza vaccines in adults immediately prior to or while in durable remission after CD19-, CD20-, or B cell maturation antigen-targeted CAR-T-cell therapy, as well as controls. We tested for antibodies to all four vaccine strains using neutralization and hemagglutination inhibition (HAI) assays. Antibody responses were defined as at least fourfold titer increases from baseline. Seroprotection was defined as a HAI titer ≥40. Enrolled CAR-T-cell recipients were vaccinated 14-29 days prior to (n=5) or 13-57 months following therapy (n=13), and the majority had hypogammaglobulinemia and cellular immunodeficiencies prevaccination. Eight non-immunocompromised adults served as controls. Antibody responses to ≥1 vaccine strain occurred in 2 (40%) individuals before CAR-T-cell therapy and in 4 (31%) individuals vaccinated after CAR-T-cell therapy. An additional 1 (20%) and 6 (46%) individuals had at least twofold increases, respectively. One individual vaccinated prior to CAR-T-cell therapy maintained a response for &gt;3 months following therapy. Across all tested vaccine strains, seroprotection was less frequent in CAR-T-cell recipients than in controls. There was evidence of immunogenicity even among individuals with low immunoglobulin, CD19+ B cell, and CD4+ T-cell counts. These data support consideration for vaccination before and after CAR-T-cell therapy for influenza and other relevant pathogens such as SARS-CoV-2, irrespective of hypogammaglobulinemia or B cell aplasia. However, relatively impaired humoral vaccine immunogenicity indicates the need for additional infection-prevention strategies. Larger studies are needed to refine our understanding of potential correlates of vaccine immunogenicity, and durability of immune responses, in CAR-T-cell therapy recipients. © Author(s) (or their employer(s)) 2021. Re-use permitted under CC BY-NC. No commercial re-use. See rights and permissions. Published by BMJ.                                                                                                                                                                                                                                |
|34698827 |Measuring vaccine efficacy against infection and disease in clinical trials: sources and magnitude of bias in COVID-19 vaccine efficacy estimates. |Clinical infectious diseases : an official publication of the Infectious Diseases Society of America |2021 Oct 26      |<AbstractText Label="BACKGROUND" NlmCategory="BACKGROUND">Phase III trials have estimated COVID-19 vaccine efficacy (VE) against symptomatic and asymptomatic infection. We explore the direction and magnitude of potential biases in these estimates and their implications for vaccine protection against infection and against disease in breakthrough infections. <AbstractText Label="METHODS" NlmCategory="METHODS">We developed a mathematical model that accounts for natural and vaccine-induced immunity, changes in serostatus and imperfect sensitivity and specificity of tests for infection and antibodies. We estimated expected biases in VE against symptomatic, asymptomatic and any SARS͏CoV2 infections and against disease following infection for a range of vaccine characteristics and measurement approaches, and the likely overall biases for published trial results that included asymptomatic infections. <AbstractText Label="RESULTS" NlmCategory="RESULTS">VE against asymptomatic infection measured by PCR or serology is expected to be low or negative for vaccines that prevent disease but not infection. VE against any infection is overestimated when asymptomatic infections are less likely to be detected than symptomatic infections and the vaccine protects against symptom development. A competing bias towards underestimation arises for estimates based on tests with imperfect specificity, especially when testing is performed frequently. Our model indicates considerable uncertainty in Oxford-AstraZeneca ChAdOx1 and Janssen Ad26.COV2.S VE against any infection, with slightly higher than published, bias-adjusted values of 59.0% (95% uncertainty interval [UI] 38.4 to 77.1) and 70.9% (95% UI 49.8 to 80.7) respectively. <AbstractText Label="CONCLUSIONS" NlmCategory="CONCLUSIONS">Multiple biases are likely to influence COVID-19 VE estimates, potentially explaining the observed difference between ChAdOx1 and Ad26.COV2.S vaccines. These biases should be considered when interpreting both efficacy and effectiveness study results. © The Author(s) 2021. Published by Oxford University Press for the Infectious Diseases Society of America.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
|34697214 |Author Response: Guillain-Barré Syndrome in the Placebo and Active Arms of a COVID-19 Vaccine Clinical Trial.                                      |Neurology                                                                                            |2021 10 26       |NA                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |

# Text Mining


```r
input <- fread("https://github.com/USCbiostats/data-science-data/raw/master/03_pubmed/pubmed.csv")
```

## Part 1

### Tokenizing abstracts & plotting top 20 words
The majority of these top 20 words are stopwords. The words that stand out to me here are *covid*, *19*, *patients*, *cancer*, and *prostate*.


```r
input <- as_tibble(input)
input %>% unnest_tokens(token, abstract) %>% count(token, sort = TRUE) %>% top_n(20, n) %>% ggplot(aes(x = n, y = fct_reorder(token, n ))) + 
    geom_col(fill = 'lightslateblue') +
    labs(title = "Top 20 Most Frequent Words in Abstracts", y = "", x = "frequency")
```

![](README_files/figure-html/unnamed-chunk-1-1.png)<!-- -->

### Let's take out stopwords and see how the list changes.

The list of tokens has now changed -- the five most common stopwords now are *covid*, *19*, *patients*, *cancer*, and *prostate*. These words showed up on the previous list, but the other tokens in this list did not. *Covid* and *19* have similar frequencies, likely because they were commonly used together.


```r
input %>%
  unnest_tokens(token, abstract) %>%
  count(token, sort = TRUE) %>% 
  anti_join(stop_words, by = c("token" = "word")) %>% 
  top_n(20, n) %>% ggplot(aes(x = n, y = fct_reorder(token, n ))) + 
    geom_col(fill = 'lightslateblue') +
    labs(title = "Top 20 Most Frequent Words in Abstracts, without Stopwords", y = "", x = "frequency")
```

![](README_files/figure-html/remove-stopwords-1.png)<!-- -->

#### Let's group the the 5 most frequent tokens per search term

The corresponding tokens per search term are not surprising to me, as they are closely related. One thing that I thought was interesting was that *women* appeared as a top token for the search term *preeclampsia*, but *men* was not in the top 5 tokens for the search term *prostate cancer*.


```r
input %>%
  unnest_tokens(token, abstract) %>%
  anti_join(stop_words, by = c("token" = "word")) %>% 
  group_by(term)%>%
  count(token)%>%
  top_n(5, n) %>% knitr::kable(caption = "5 Most Common Tokens for each Search Term, After Removing Stopwords")
```



Table: 5 Most Common Tokens for each Search Term, After Removing Stopwords

|term            |token        |    n|
|:---------------|:------------|----:|
|covid           |19           | 7035|
|covid           |covid        | 7275|
|covid           |disease      |  943|
|covid           |pandemic     |  800|
|covid           |patients     | 2293|
|cystic fibrosis |cf           |  625|
|cystic fibrosis |cystic       |  862|
|cystic fibrosis |disease      |  400|
|cystic fibrosis |fibrosis     |  867|
|cystic fibrosis |patients     |  586|
|meningitis      |clinical     |  187|
|meningitis      |csf          |  206|
|meningitis      |meningeal    |  219|
|meningitis      |meningitis   |  429|
|meningitis      |patients     |  446|
|preeclampsia    |eclampsia    | 2005|
|preeclampsia    |pre          | 2038|
|preeclampsia    |preeclampsia | 1863|
|preeclampsia    |pregnancy    |  969|
|preeclampsia    |women        | 1196|
|prostate cancer |cancer       | 3840|
|prostate cancer |disease      |  652|
|prostate cancer |patients     |  934|
|prostate cancer |prostate     | 3832|
|prostate cancer |treatment    |  926|

## Part 2

### Top 10 Bi-grams with Stopwords

These top bi-grams include some common stop word phrases (*of the*, *in the*, *and the*, *to the*), as well as some that relate to the healthcare field and include the top tokens that were seen above (*covid 19*, *prostate cancer*, *pre eclampsia*). 


```r
input %>% unnest_ngrams(ngram, abstract, n = 2) %>% 
  count(ngram, sort = TRUE) %>%
  top_n(10, n) %>%
  ggplot(aes(x = n, y = fct_reorder(ngram, n))) +
  geom_col(fill = 'lightslateblue') +
  labs(title = "Top 10 Bi-grams, with Stopwords", y = "", x = "frequency")
```

![](README_files/figure-html/bigrams-1.png)<!-- -->

### Top 10 Bigrams, without Stopwords

After removing stopwords, these top 10 bigrams are now specific to the search terms that were shown above (*covid 19*, *prostate cancer*, *pre eclampsia*, *cystic fibrosis*). There is one search term missing: *meningitis*. Most of the top 10 is populated by bigrams related to covid 19. The *95 ci* bigram is interesting, and makes sense to me given these are research publications, so I would expect statistics to have been used for most of these studies. 


```r
bigrams <- input %>%
  unnest_ngrams(ngram, abstract, n = 2) %>% 
  separate(col=ngram, into=c("word1", "word2"), sep = " ") %>%
  select(word1, word2) %>%
  anti_join(stop_words, by = c("word1" = "word")) %>%
  anti_join(stop_words, by = c("word2" = "word")) %>%
  count(word1, word2, sort=TRUE) %>%
  top_n(10, n)
unite(bigrams, "ngram", c("word1", "word2"), sep = " ") %>%
  ggplot(aes(x = n, y = fct_reorder(ngram, n))) +
  geom_col(fill = 'lightslateblue') +
  labs(title = "Top 10 Bi-grams, without Stopwords", y = "", x = "frequency")
```

![](README_files/figure-html/bigrams-nostopwords-1.png)<!-- -->

## Part 3

### TF-IDF (Term Frequency * Inverse Document Frequency) for Search Term 

Compared to the previous table from part 1, there are some differences in the terms with the highest TF-IDF values. For example, the terms with the highest TF-IDF for the search term *prostate cancer* are *prostate*, *androgen*, *psa*, *prostatectomy*, and *castration*. Some of these words (*prostate*) appear on the previous list and are more generalized to the search term, whereas the others (*androgen*, *psa*, *prostatectomy*, *castration*) are unique to this list, and provide a little more detail and context about the text. To my understanding, terms that are less common across documents would have a higher inverse document frequency, which explains why there are differences in the words that appear on this list compared to the table from part 1, which showed the most frequent words overall.

The top 5 tokens from each search term with the highest TF-IDF values are:

* search term "covid": *covid*, *pandemic*, *coronavirus*, *sars*, *cov*

* search term "meningitis": *meningitis*, *meningeal*, *pachymeningitis*, *csf*, *meninges*

* search term "prostate cancer": *prostate*, *androgen*, *psa*, *prostatectomy*, *castration*

* search term "preeclampsia": *eclampsia*, *preeclampsia*, *pregnancy*, *maternal*, *gestational*

* search term "cystic fibrosis": *cf*, *fibrosis*, *cystic*, *cftr*, *sweat*


```r
input %>%
  unnest_tokens(token, abstract) %>%
  count(token, term) %>%
  bind_tf_idf(token, term, n) %>%
  group_by(term) %>%
  top_n(5, tf_idf) %>%
  arrange(desc(tf_idf), .by_group = TRUE) %>%
  select(term, token, n, tf_idf, tf, idf) %>% knitr::kable(caption="5 Tokens from each Search Term with Highest TF-IDF Value")
```



Table: 5 Tokens from each Search Term with Highest TF-IDF Value

|term            |token           |    n|    tf_idf|        tf|       idf|
|:---------------|:---------------|----:|---------:|---------:|---------:|
|covid           |covid           | 7275| 0.0597183| 0.0371050| 1.6094379|
|covid           |pandemic        |  800| 0.0065670| 0.0040803| 1.6094379|
|covid           |coronavirus     |  647| 0.0053110| 0.0032999| 1.6094379|
|covid           |sars            |  372| 0.0030536| 0.0018973| 1.6094379|
|covid           |cov             |  334| 0.0027417| 0.0017035| 1.6094379|
|cystic fibrosis |cf              |  625| 0.0116541| 0.0127188| 0.9162907|
|cystic fibrosis |fibrosis        |  867| 0.0090127| 0.0176435| 0.5108256|
|cystic fibrosis |cystic          |  862| 0.0089608| 0.0175417| 0.5108256|
|cystic fibrosis |cftr            |   86| 0.0028167| 0.0017501| 1.6094379|
|cystic fibrosis |sweat           |   83| 0.0027184| 0.0016891| 1.6094379|
|meningitis      |meningitis      |  429| 0.0147974| 0.0091942| 1.6094379|
|meningitis      |meningeal       |  219| 0.0075539| 0.0046935| 1.6094379|
|meningitis      |pachymeningitis |  149| 0.0051394| 0.0031933| 1.6094379|
|meningitis      |csf             |  206| 0.0040453| 0.0044149| 0.9162907|
|meningitis      |meninges        |  106| 0.0036562| 0.0022718| 1.6094379|
|preeclampsia    |eclampsia       | 2005| 0.0229802| 0.0142784| 1.6094379|
|preeclampsia    |preeclampsia    | 1863| 0.0213527| 0.0132672| 1.6094379|
|preeclampsia    |pregnancy       |  969| 0.0035250| 0.0069006| 0.5108256|
|preeclampsia    |maternal        |  797| 0.0028993| 0.0056757| 0.5108256|
|preeclampsia    |gestational     |  191| 0.0021891| 0.0013602| 1.6094379|
|prostate cancer |prostate        | 3832| 0.0501967| 0.0311890| 1.6094379|
|prostate cancer |androgen        |  305| 0.0039953| 0.0024824| 1.6094379|
|prostate cancer |psa             |  282| 0.0036940| 0.0022952| 1.6094379|
|prostate cancer |prostatectomy   |  215| 0.0028164| 0.0017499| 1.6094379|
|prostate cancer |castration      |  148| 0.0019387| 0.0012046| 1.6094379|


