# Thyroid

#-------REQUIRE
require(dplyr)
require(org.Hs.eg.db)
require(edgeR)
require(RColorBrewer)
require(tidyr)

#-------IMPORT & FILTERING
counts    <- read.delim("matriz4x4.txt.TXT", header=T, row.names=1)
RNAblood  <- read.table("Rnablood.csv", header=TRUE, sep=",") %>% 
             filter(nTPM >= mean(nTPM))

#-------ID Conversion
counts$symbol <- mapIds(org.Hs.eg.db, 
                        keys=row.names(counts),
                        keytype='REFSEQ', 
                        column='SYMBOL')

counts$Exists <- counts$symbol %in% RNAblood$Gene.name 
counts        <- counts %>% filter(Exists == FALSE) %>% drop_na()
counts$Exists <- NULL               

counts <- counts[!duplicated(counts$symbol), ]
counts <- data.frame(counts, row.names=9)

#-------HYPOTHESIS TESTING
group   <- c(rep('CTL',4),rep('HT',4))
group   <- as.factor(group)
design  <- model.matrix(~group)
norm    <- calcNormFactors(counts)
DGL     <- DGEList(counts, norm.factors=norm, group=group)
DGL     <- estimateCommonDisp(DGL)
Test    <- exactTest(DGL)
summary(decideTestsDGE(Test, p.value = 0.05))


#-------Data Extraction & Plot
top = topTags(Test, n=9999)
top = top@.Data[[1]]
DGE = subset(top, FDR<0.05)
plotSmear(DGL,de.tags = row.names(DGE), cex = 3,col=blues9)
