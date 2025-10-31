###
This script allows to visualize relative value of hybrid credit bonds in terms of SSS (Sub to Senior spread) both from an historical and peers perspectives. 
Both systems work on Bloomberg's BQUANT (using Bloomberg data).
They work by pulling the € curve of the issuer (selected in-app by user) and performing regression to interpolate over the maturity eactly equal to the Hybrid bond Next-Call-Date
Rergression is pulled from bloomberg, and is done over points that match user requirements (maturity, amount outstanding)
Interactive graphs are then plotted using plotly

Different model of regression (limited to the one amde available by bloomberg): Nelson-Siegel, Nelson-Siegel-Svensson, Linear, Linear-Log, Log-Log.
Polynomial and Quadratic models are not (yet) available on BQNT/BQL even if they are enabled in NIA<GO> 

#Monitor:

