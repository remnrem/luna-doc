
# R visualization code for the CHEP vignette

The following snippets of R code were used to generate the figures in [this vignette](chep.md).

## Raw data PSD

```
library(luna)
d <- ldb("ss_psd1.db")

# check the outputs
lx(d)

# extract PSD for frequency bins
psd = d$PSD$CH_F
psd$PSD <- 10*log10(psd$PSD)

# plot PSD for all channels
psd$CH <- toupper(psd$CH)
topo <- ldefault.xy()
chs <-  topo$CH[ topo$CH %in% unique(psd$CH)]
pal <- rainbow(length(chs))
plot( range(psd$F), range((psd$PSD)),  type='n',
      ylab="10xlog10(PSD)", xlab="Frequency", cex.lab=1.3)
for (ind in seq_along(chs))
 {lines(psd$F[psd$CH == chs[ind]], psd$PSD[psd$CH == chs[ind]],  col=pal[ind])}

par(new=TRUE, oma=c(1,4,5,1), mfcol=c(2,2), mfg=c(1,2), mar=c(3,5,1,1))
plot(c(0, 1), c(0, 1), type = "n", axes = F, xlab = "", ylab = "",
     xaxt = "n", yaxt = "n")
for (ind in seq_along(chs)) {
 px <- topo$X[topo$CH == chs[ind]] + 0.5
  py <- topo$Y[topo$CH == chs[ind]] + 0.5
 points(px, py, pch = 21, cex = 1.5, col = pal[ind], bg= pal[ind])
}
```


## Raw data Hjorth parameters

```
ss = d$SIGSTATS$CH_E
X11(width=12, height=3)
par(mfrow=c(1,3) , mar=c(3,3,3,1) )
for (h in names(ss[4:6])) {
 if (h == "H1") {H <-  log(ss[,h])} else {H  <-  ss[,h]}
 lheatmap( ss$E , ss$CH , H ) 
 title(main=h, cex.main=2, xlab="epochs",ylab="channels",line=0.2,cex.lab=2)
} 
```
						  



## Bad channel detection

```
library(luna)
c <- ldb("chep_bchs.db")

# check the outputs
lx(c)

#plot CHEP mask
chep <- c$CHEP$CH_E
X11(width=10, height=8)
lheatmap( chep$E , chep$CH , chep$CHEP, col = c("lightgray","brown"))
ind <- order(chep$E, chep$CH)
axis(2,at=seq(0,1,length.out=length(unique(chep$CH))),
     labels=unique(chep$CH[ind]), las=2 , cex.axis = 0.6) 
title(main="CHEP mask", cex.main=1.5, xlab="epochs",line=0.5, cex.lab=1.5)
```

## Interpolating bad channels

```
library(luna)
d_cln <- ldb("ss_psd2.db") 

X11(width=12, height=6)
par(mfrow=c(1,2) , mar=c(4,4,3,1) )

# plot chep mask
chep <- d_cln$CHEP$CH_E
lheatmap( chep$E , chep$CH , chep$CHEP, col = c("lightgray","brown"))
ind <- order(chep$E, chep$CH)
axis(2,at=seq(0,1,length.out=length(unique(chep$CH))),
     labels=unique(chep$CH[ind]), las=2 , cex.axis = 0.6) 
title(main="CHEP mask", cex.main=2, xlab="epochs",line=0.5, cex.lab=2)
bad_chs <- toupper(unique(chep$CH[chep$CHEP > 0]))

# extract PSD for frequency bins cleaned data
psd_cln = d_cln$PSD$CH_F
psd_cln$PSD <- 10*log10(psd_cln$PSD)
psd_cln$CH <- toupper(psd_cln$CH)

# extract PSD for frequency bins uncleaned data
d <- ldb("ss_psd1.db") 
psd = d$PSD$CH_F
psd$PSD <- 10*log10(psd$PSD)
psd$CH <- toupper(psd$CH)

#before after bad channels on top of other
topo <- ldefault.xy()
chs <-  topo$CH[ topo$CH %in% unique(psd$CH)]
plot(range(psd$F), range((psd$PSD)),  type='n',
     ylab="10log10(PSD)", xlab="Frequency")
for (ind in seq_along(chs)) 
 lines(psd_cln$F[psd_cln$CH == chs[ind]],
       psd_cln$PSD[psd_cln$CH == chs[ind]],
       col="grey")
for (b_ch in bad_chs) {
 ind = which(chs==b_ch)
 lines(psd$F[psd$CH == chs[ind]],
       psd$PSD[psd$CH == chs[ind]],
       col="maroon",lwd = 2 )
 lines(psd_cln$F[psd_cln$CH == chs[ind]],
       psd_cln$PSD[psd_cln$CH == chs[ind]],
       col="blue", lwd = 2)
}

legend(10, 35, c("Before interpolation", "After interpolation"), 
       col=c("maroon", "blue"), lty=1, box.lty=0)
title (paste0("Bad channels: ", (paste(bad_chs, collapse=","))))
```

## Epoch-level interpolation 

```
library(luna)
d_cln <- ldb("ss_psd3.db") 

X11(width=12, height=6)
par(mfrow=c(1,2) , mar=c(4,4,3,1) )

# plot chep mask
chep <- d_cln$CHEP$CH_E
lheatmap( chep$E , chep$CH , chep$CHEP, col = c("lightgray","brown"))
ind <- order(chep$E, chep$CH)
axis(2,at=seq(0,1,length.out=length(unique(chep$CH))),
     labels=unique(chep$CH[ind]), las=2 , cex.axis = 0.6) 
title(main="CHEP mask", cex.main=2, xlab="epochs",line=0.5, cex.lab=2)

# extract PSD for frequency bins cleaned data
psd_cln = d_cln$PSD$CH_F
psd_cln$PSD <- 10*log10(psd_cln$PSD)
psd_cln$CH <- toupper(psd_cln$CH)

# extract PSD for frequency bins uncleaned data
d <- ldb("ss_psd1.db") 
psd = d$PSD$CH_F
psd$PSD <- 10*log10(psd$PSD)
psd$CH <- toupper(psd$CH)

#before after interpolation
topo <- ldefault.xy()
chs <-  topo$CH[ topo$CH %in% unique(psd$CH)]
plot(range(psd$F), range((psd$PSD)),  type='n',
     ylab="10log10(PSD)", xlab="Frequency")
for (ind in seq_along(chs))
  lines(psd_cln$F[psd$CH == chs[ind]],
        psd$PSD[psd$CH == chs[ind]],  col="maroon")
for (ind in seq_along(chs)) 
  lines(psd_cln$F[psd_cln$CH == chs[ind]],
        psd_cln$PSD[psd_cln$CH == chs[ind]],  col="blue")
legend(10, 35, c("Before interpolation", "After interpolation"), 
       col=c("maroon", "blue"), lty=1, box.lty=0)
```
	


## Masking remaining outlier epochs

```
library(luna)
d_cln <- ldb("ss_psd4.db") 

X11(width=12, height=6)
par(mfrow=c(1,2) , mar=c(4,4,3,1) )

# plot chep mask
chep <- d_cln$CHEP$CH_E
lheatmap( chep$E , chep$CH , chep$CHEP, col = c("lightgray","brown"))
ind <- order(chep$E, chep$CH)
axis(2,at=seq(0,1,length.out=length(unique(chep$CH))),
     labels=unique(chep$CH[ind]), las=2 , cex.axis = 0.6) 
title(main="CHEP mask", cex.main=2, xlab="epochs",line=0.5, cex.lab=2)

# extract PSD for frequency bins cleaned data
psd_cln = d_cln$PSD$CH_F
psd_cln$PSD <- 10*log10(psd_cln$PSD)
psd_cln$CH <- toupper(psd_cln$CH)

# extract PSD for frequency bins uncleaned data
d <- ldb("ss_psd1.db") 
psd = d$PSD$CH_F
psd$PSD <- 10*log10(psd$PSD)
psd$CH <- toupper(psd$CH)

#before after bad channels on top of others
topo <- ldefault.xy()
chs <-  topo$CH[ topo$CH %in% unique(psd$CH)]
plot(range(psd$F), range((psd$PSD)),  type='n',
     ylab="10log10(PSD)", xlab="Frequency")
for (ind in seq_along(chs))
  lines(psd_cln$F[psd$CH == chs[ind]],
        psd$PSD[psd$CH == chs[ind]],
	col="maroon")
for (ind in seq_along(chs))
  lines(psd_cln$F[psd_cln$CH == chs[ind]],
        psd_cln$PSD[psd_cln$CH == chs[ind]],
	col="blue")
legend(10, 35, c("Before interpolation", "After interpolation"), 
       col=c("maroon", "blue"), lty=1, box.lty=0)
```
	
