---
title: 'Intro R snippets'
date: 2023-07-24
permalink: /posts/2023/07/blog-post-2/
tags:
  - code
  - R
  - tutorial
  - intro
---

This is a collection of intro R code snippets that I found helpful to use as a cheat sheet when first starting out in my general data analysis work in R.

Code for getting a feel for the data and environment
=====
```R
# Get a feel for the data
dim(my_data) # output the dimensions rows by columns
class(my_data) # display type of variable
rownames(my_data) # view row names
colnames(my_data) # view column names
head(my_data,x) # view first x rows
tail(my_data,x) # view last x rows
is.na(my_data) # sets TRUE for NAs and FALSE elsewhere

# Get a feel for the environment
help(thingneedhelpwith) # get help with a function/package
setwd(/path/to/set/as/working/dir) # sets working directory
ls() # print current stored variables
dir() # prints files in working directory
getwd() # prints path location of working directory
```

Loading, modifying, and saving data
=======
```R
# Loading data
df <- read.table('file.txt') # read in a file of any type
df <- data.table::fread('file.txt',header=T) # faster read in file
write.table(df, 'file.txt') # write dataframe to a file of any type
load('file.RData') # load in R data file
save(df, file = 'file.Rdata') # save dataframe as an R data file
rm(list=ls()) # clears variables stored
INPUT_FILE=commandArgs(TRUE)[1] # example of accepting input from command line, change number based on how many inputs and which one referring to

# Loading key packages I use a lot
library(tidyr)
library(dplyr)
library(data.table)
library(RColorBrewer)
library(ggplot2)

# Changing data type
as.data.frame(my_data) # convert to a dataframe
as.data.table(my_data) # convert to a datatable
as.integer(my_data) # convert to integers
as.numeric(my_data) # convert to numbers
as.character(my_data) # convert to characters
cut(x, breaks = 4) #Turn a numeric vector into a factor but ‘cutting’ into sections

# Saving df as file
OUTPUT_FILE="/the/path/to/the/file/here/nameoffile.tsv"
write.table(df, OUTPUT_FILE, quote=F, sep="\t", row.names=F, col.names=T)
```

Plotting with ggplot
=====
```R
library(ggplot2)
# SCATTER PLOT: with each point labeled, manual color setting to correspond to metabolite group color key, changing axis label size and rotating x axis-labels only
# Changing labels for axes and color legend, showing all labels even if overlap and drawing line to point if label needs to be farther away
options(ggrepel.max.overlaps = Inf)
colors <- c("orange", "green", "red","purple", "blue", "pink")
mets <- c("Amino Acid","Fatty Acid", "Glycolysis Related Metabolite", "Ketone", "Lipid Particle", "Other")
ggplot(pathsOI,aes(x=pathway1.bed,y=pathway2.bed,color=Color_Group))+
	geom_point()+geom_label_repel(aes(label=metabolite), size=4, alpha=0.75, show.legend = FALSE,force=10,segment.size=0.25)+
	scale_color_manual(labels=mets, values = colors)+
	labs(x="Label for the x axis",y="Label for the y axis",color="Metabolite Group")+
	theme_classic()+ theme(axis.text=element_text(size=12),axis.title=element_text(size=14),axis.text.x = element_text(angle = 70, hjust = 1))
ggsave(paste0(PLOT_DIR,"nameofmyfigure.png"),height=12,width=12)

# VIOLIN PLOT: changing the violin width, transparancy and color + label individual points
ggplot(h_annot,aes(x=col1,y=col2,fill=Color_Group))+geom_violin(alpha=0.5)+ geom_boxplot(width=0.1)+
	geom_point()+geom_label_repel(aes(label=metabolite), size=4, alpha=0.75, show.legend = FALSE,force=10,segment.size=0.25)+
	scale_fill_manual(labels=mets, values = colors)+
	labs(x="Name for x axis",y="Name for y axis",fill="Metabolite Group")+
	theme_classic()+ theme(axis.text=element_text(size=12),axis.title=element_text(size=14),axis.text.x = element_text(angle = 70, hjust = 1))
ggsave(paste0(PLOT_DIR,"nameofmyfigure.png"),height=12,width=12)

# MATRIX CORRELATION-LIKE PLOT: plot cooccurance matrix visually
v[upper.tri(v,diag=FALSE)]<- NA # half to NAs
library(tidyr)
library(ggplot2)
cm <- as.data.frame(v) %>% mutate(Var1 = factor(row.names(.), levels=row.names(.))) %>%
  gather(key = Var2, value = value, -Var1, na.rm = TRUE, factor_key = TRUE)
cm <- cm %>% rename(Pathways1=Var1,Pathways2=Var2)
ggplot(data = cm) + geom_tile(aes(Pathways1, Pathways2, fill = value))+
      geom_text(aes(Pathways1, Pathways2, label = value))+theme_classic()+
theme(axis.text.x = element_text(angle = 70, hjust = 1))+ # text = element_text(size=20),
 scale_fill_gradient2(low = "red", high = "blue", mid = "white",
   midpoint = 0, name="Number of variants\nin both")+ # the "\n" adds a newline in the label
 coord_fixed()+labs(x="Pathways 1",y="Pathways 2")
ggsave("nameofmyfigure.png")
```
