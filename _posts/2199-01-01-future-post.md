---
title: 'Oddly Specific, Potentially Useful Code - General R snippets'
date: 2023-08-17
permalink: /posts/2012/08/blog-post-4/
tags:
  - code
  - R
  - tutorial
---

This is a collection of oddly specific, potential useful code snippets that I use frequently enough in my general data analysis work in `R` that I wanted to put them in one place as a reference. Some of these things can be a bit of a pain to figure out or look up, so I wanted to share in case it helps someone else.

Each of the below code sets is completely independent from one another if under a separate comment (#), except for the initial loading of the `dplyr` package.

Filtering rows and subseting columns
```R
library(dplyr)

# df keep row with lowest of a certain value for each group
newdf <- as.data.table(mydf)[ , .SD[which.min(P)], by = Gene]

# If have pairwise combinations of things in two separate columns and want to filter down to only a single row for each unique pair (where order doesn't matter)
mydf <- mydf[!duplicated(t(apply(mydf %>% select(ID,Item.1,Item.2), 1, sort))),]

# subset rows in a df based on having a certain number of columns that meet a criteria
sigthres = 5e-8
ncut = 1
pvals <- mydif %>% select(matches("^P_VALUE_")) # make a new df that only has the pvalue columns
newdf <- mydif[apply(pvals, 1, function(r) sum(r < sigthres) >= ncut),] # only keep rows where the pval is < sigthres for ncut or more columns

# select top n of a group
n=2
newdf <- as.data.frame(mydf %>% group_by(Col1,Col2) %>% arrange(Col1,Col2,desc(myvalue)) %>% slice(1:n) # filter to only keep top two values in myvalue per group

# select a subset of columns from a dataframe based on a substring or a custom list
newdf <- df %>% select(matches("BETA_"))
newdf <- df %>% select(all_of(mylistofcolumns)) # could also replace "all_of" with "any_of" if just want to keep any of the columns in mylist that are present in df

# Use a column name with special/annoying characters directly in filter/select
newdf <- filter(mydf,`P-VALUE`<5e-8)

# df group by a group but only keep one row from each group that meets a criteria (and breaks ties by picking first)
df <- as.data.frame(mydf %>% group_by(GENES) %>% filter(num_sig == max(num_sig)) %>% filter(1:n() == 1))
```

Munging the data
```R
library(dplyr)

# replace a single element in a df based on matching criteria
an <- as.data.frame(an)
an["Gene"][an["SNP"]=="rs1234"] <- "GENENAME1"

# turn rownames in df to a column
newdf <- as.data.frame((setDT(all, keep.rownames = TRUE)[]) %>% dplyr::rename(ID=rn)) # make rownames a column

# Separate a column that has a list (or any seperator) on the separator to split into either separate rows or separate columns
long <- tidyr::separate_rows(mydf,"Genes",sep=",") # split into multiple rows
mydf2 <- mydf2 %>% tidyr::separate(V2,sep=":",into=c("CHR","POS","A1","A2")) # split into multiple columns

# Collapse a group into one line each and comma separate all entries for each group for a given row
newdf <- mydf %>% group_by(SNP,BETA,SE,P_BOLT_LMM,MetaboliteName) %>% arrange(GENE) %>% summarize(GENES=paste(GENE,collapse=",")) # one row per variant, list genes

# Order a df by custom list (in this case "mypathwaylist") then subset to just first occurance
df_ordered <- mydf %>% mutate(Gene_Category = factor(Gene_Category, levels = mypathwaylist)) %>% arrange(Gene_Category) # custom order pathways
mydf <- as.data.frame(df_ordered %>% group_by(Gene) %>% filter(1:n() == 1)) # filter to only keep first occurance of each gene

# Switch wide matrix of SNP, beta / se / pvalue for each metabolite to long form
mydf_long <- data.table::melt(mydf, measure.vars = patterns("^BETA_", "^SE_", "^P_BOLT_LMM"),variable.name=c("Metabolite"), value.name=c("BETA", "SE", "P_BOLT_LMM"))
mydf_long$Metabolite <- rep(gsub("BETA_","",colnames(mylistofcolumns)),each=nmets)

# sort numerically a SINGLE list
l <- stringr::str_sort(l, numeric = TRUE)
```

Working with dynamic strings and variables
```R
library(dplyr)

# Have R (such as during a for loop) treat your string saved as a variable as the column name for filter/select etc
for (col in 1:(ncol(mydf)-1)){
group_num=paste0("V",col,sep="")
group <- filter(mydf,!!sym(group_num) > 1) # filter to keep rows > 1 in the specified column
}

# Add a column with a dynamic (custum based on other variable) column name
i=1
varname <- paste0("cluster",i)
df[, varname] <- "test" # adds column called cluster1 to the dataframe

# Treat variable name as a string and/or turn a string into a variable
deparse(substitute(my_variable)) # variable name to string
eval(parse(text = my_string)) # string to variable name
```

Reformating Data
```R
library(dplyr)

# Replace the last occurance of something with whatever
gsub("l([^l]*)$", "hello", replacement="\\1") # replace "l" in this case with nothing
gsub("_([^_]*)$", p$V1, replacement=" \\1") # replace last underscore with a space

# Remove "chr" before chromosome numbers in the entire dataset (e.g. chr1 --> 1) and then add it back
df$CHROM <- as.integer(gsub("chr","",as.character(df$CHROM))) # remove "chr" before each
df$CHROM <- paste0("chr",df$CHROM) # add "chr" before each... (or could use gsub("^","chr",df$CHROM))

# Delete a symbol and all characters after it for all elements in a df column
mydf$genes <- gsub("\\..*","",mydf$genes)
```

Miscellaneous
```R
library(dplyr)

# Use summarize to calculate mean and se for each group
newdf <- mydf %>% group_by(Pathway) %>% summarize(absvalue=abs(myvalues),avgvalue=mean(myvalues),sevalue=sd(myvalues)/sqrt(n()))

# Loop through a bunch of files and bind them all together with a new column that has a modified
# name indicating which file that row came from
my.path <- "/my/path/to/these/file/of/interest/" # set the directory path
my.list <- paste0(my.path, list.files(my.path, pattern = "_results.txt")) # create list of file names, assumes all files in this folder follow this pattern 
my.joined <- data.table::rbindlist(lapply(my.list, function(h) data.table::fread(h) %>% mutate(MetaboliteName=gsub(".*interest/*", "", gsub("_results.*", "", h))))) # Read in all files, bind them together and add a column with the name of the trait the file came from

# Labeling points in a plot
library(ggrepel)
library(ggplot2)
ggplot(mydf,aes(x=PA1,y=PA2,color=ColorColumn),size=6)+geom_point()+labs(x="Factor 1",y="Factor 2")+
	geom_label_repel(aes(label=Metabolites), size=4, show.legend = FALSE,force=10,segment.size=0.25)+
  scale_color_brewer(palette = "Dark2") + theme_classic()
ggsave(paste0(PLOT_DIR,prep,"_scores12.pdf"),height=15,width=20)

# globally for entire session set to always show all labels, regardless of too many overlaps, for ggrepel
options(ggrepel.max.overlaps = Inf)

# Quickly load a very large dataset
mydf <- data.table::fread("mylargedataset.tsv")
```
