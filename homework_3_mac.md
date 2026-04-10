~={red}(1point)=~ for Alpha Rarefaction Plot
Run Core Metrics ~={red}(1 point; .25pts per line)=~
Make alpha diversity plots ~={red}(3points)=~
~={red}10 points=~ for the questions 

~={red}15 points total=~
------------------------------------------------------------------

Due: 

**For complete credit for this assignment, you must answer all questions and include all commands in your obsidian upload.**

------------------------------------------------------------------
**Learning Objectives**
1. Practice recording commands and editing code to match your analysis.
2. Perform alpha rarefaction and determine an appropriate sequencing depth.
3. Run core metrics, generate plots for alpha and beta diversity
--------------------------------------------------

**Cow Site Data Workflow**, part 3

Load qiime2 in a terminal session after you go into the **cow** folder

```
# Insert the two commands to activate qiime2

module purge
module load qiime2/2024.10_amplicon
```

### Alpha Rarefaction Plot ~={red}(1 point)=~
- Chose the input sequencings depths (min and max) for generating the alpha rarefaction plot: 

```
#go to the cow directory

qiime diversity alpha-rarefaction \  
--i-table dada2/cow_table_dada2_filtered300.qza \  
--m-metadata-file metadata/cow_metadata.txt \  
--o-visualization alpha_rarefaction_curves_16S.qzv \  
--p-min-depth 1000 \  
--p-max-depth 25000
```


### Run Core Metrics ~={red}(1 point)=~

```
qiime diversity core-metrics-phylogenetic \  
--i-table dada2/cow_table_dada2_filtered300.qza \  
--i-phylogeny tree/tree_placements_gg2.qza \  
--m-metadata-file metadata/cow_metadata.txt \  
--p-sampling-depth 2000 \  
--output-dir core-metrics_filtered300_depth2000/5K
```


### Visualize alpha diversity plots
- generate a plot to visualize the observed features ~={red}(1 point)=~
```

qiime diversity alpha-group-significance \  
--i-alpha-diversity core-metrics_filtered300_depth2000/observed_features_vector.qza \  
--m-metadata-file metadata/cow_metadata.txt \  
--o-visualization core-metrics_filtered300_depth2000/observed_features_group_significance.qzv
```

- generate a plot to visualize faith's PD ~={red}(2 points)=~
```
## insert the entire code chunk for generating this visualization 

qiime diversity alpha-group-significance \  
--i-alpha-diversity core-metrics_filtered300_depth2000/faith_pd_vector.qza \  
--m-metadata-file metadata/cow_metadata.txt \  
--o-visualization core-metrics_filtered300_depth2000/faith_pd_group_significance.qzv
```



## Homework questions ~={red}(10 points)=~

1. what is the name of the file you needed to use to figure out what min and max depths to use to generate the alpha rarefaction plot? (Hint: which file contains the sequencing depths for each sample).                                                                                                                                 The file was `dada2/cow_table_dada2_filtered300.qzv`, which contains the sequencing depths for each sample and was used to choose the minimum and maximum depths for the alpha rarefaction plot.

2. what did you choose for the rarefaction depth (the input for core metrics -p-sampling-depth flag)? why?                                                                                                                                I chose a rarefaction depth of 2000 because that is the `--p-sampling-depth` value used in the core metrics step, and it is a reasonable depth that helps retain samples while still standardizing sequencing depth across samples for comparison. 4K-6K

3. Which cow body location had more observed features? Which has the lowest?                 Fecal had the highest observed features, and control had the lowest. The skin group was also high, but fecal appeared slightly higher overall in the boxplot. Nasal

4. What is the main difference between Faiths PD and Shannons alpha diversity metrics?            Faith’s PD accounts for the phylogenetic relationships among taxa, while Shannon’s alpha diversity measures species richness and evenness without considering phylogeny.

5. Which diversity metrics produced by the core-metrics pipeline require phylogenetic information?                                                                                                                               Faith’s PD, weighted UniFrac, and unweighted UniFrac.

6. Which two body sites have the highest Faiths PD alpha diversity?  Are the groups significantly different?                                                                                                                   The two body sites with the highest Faith’s PD alpha diversity were skin and fecal. Yes, the groups were significantly different, with a pairwise p-value of 1.038661e-04. Nasal and Oral

7. Does it seem like there are any groupings in the beta diversity? What are the groupings? Yes, there are groupings in the beta diversity. Fecal samples form a distinct cluster, skin and udder samples cluster close together, and oral and nasal samples are more spread out and partially overlap. Skin/udder

8. Why do you think these samples are grouping together?                                                     These samples are grouping together because samples collected from the same body site tend to have more similar microbial communities than samples from different body sites. Each body site provides a different local environment, which influences which microbes are able to grow there.

9. What test can you run to determine if the groups are significantly different?                        The test is PERMANOVA (Permutational Multivariate Analysis of Variance).

10. What command would you use to run that test?

```
#insert command for running the test you suggest from question 7

qiime diversity beta-group-significance \  
--i-distance-matrix core-metrics_filtered300_depth2000/weighted_unifrac_distance_matrix.qza \  
--m-metadata-file metadata/cow_metadata.txt \  
--m-metadata-column body_site \  
--p-method permanova \  
--o-visualization core-metrics_filtered300_depth2000/weighted_unifrac_body_site_permanova.qzv \  
--p-pairwise


```