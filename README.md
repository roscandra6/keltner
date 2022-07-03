// This source code is subject to the terms of the Mozilla Public License 2.0 at https://mozilla.org/MPL/2.0/
// © eykpunter

//@version=5
indicator("Keltner Center Of Gravity Channel Mutations Indicator", shorttitle="KCGMut")

//calculate lookback
//script automatically adepts lookback to timeframe
per=14                               //initialize per
setper= 14                           //initialize setper
tf=timeframe.period
setper:= tf=="M"? 14: tf=="W"? 14: tf=="D"? 21: tf=="240"? 28: tf=="180"? 28: tf=="120"? 35: tf=="60"? 35: tf=="45"? 35: tf=="30"? 42: tf=="15"? 42: tf=="5"? 49: tf=="3"? 49: tf=="1"? 56: 10
per:= setper

//KeltCOG RELEVANT CODE
//Calculate channel
//Calculate COG
donhigh=ta.highest(high,per)           //COG calculated using values of Donchian channel
donlow=ta.lowest(low,per)
cent=(donhigh+donlow)/2             //center line not plotted, used calculation of widthpercent
donrange=donhigh-donlow
coglow=donlow+donrange*0.382        //Actually a Center Low fibonacci line in my donchian fibonacci channel
coghigh=donlow+donrange*0.618       //center high fibonacci

//Calculate borders
//calculate width (i.e. COG to border)
//script adepts width to lookback period
varwidth=2.00                           //initialize variable width
formula=math.round(2 + per/25 - 6/per, 1)    //Script uses this formula to adapt Width (i.e COG to border) to look back period
varwidth:= formula
dis=ta.atr(per)                        //Keltner channels have lines spaced by average true range
horizontal=false                    //Initialize horizontal boolean. New values only calculated when the COG (Center of Gravity) changes 

outerhigh=coghigh             //initialize Outer Keltner line above COG
outerlow=coglow               //initialize outer Kelner below COG
horizontal:= coglow==coglow[1]?true: false                //set horizontal COG changes result in Keltner changes, otherwize Keltner is horizontal
outerhigh:= horizontal?outerhigh[1]: coghigh+varwidth*dis
outerlow:= horizontal?outerlow[1]: coglow-varwidth*dis

//CALCULATE VALUES FOR INDICATOR
//define direction
trendup=close>coghigh
trenddown=close<coglow
trendside=close<=coghigh and close>=coglow

//define width percent of channel
widchan=outerhigh-outerlow
perchan = trendside? math.round(widchan/cent*25, 1): math.round(widchan/cent*50, 1) //indicator uses half of width, whisch is a little more than atr-percent
negperchan=-perchan

//define width percent of cog
widcog=coghigh-coglow
percog = trendside? math.round(widcog/cent*50, 1) :math.round(widcog/cent*100, 1)
negpercog = -percog

//CALCULATE VOLUME EXPANSION EVENTS
//Function finding the usual value of a series with a pick and choose statistical procedure, inspects 8 periods but averages the 'middlest' 4.
usual(src) =>                                            
    pick = math.sum(src,3) -ta.highest(src,3) -ta.lowest(src,3)    //pick the middle
    (math.sum(pick,6) -ta.highest(pick,6) -ta.lowest(pick,6))/4    //choose the mediocre out of the picks
// end of function

//finding volume events
usuvol= usual(volume) //find something to compare present volume with
relv= volume>usuvol? 100*(volume-usuvol)/volume :0     //only rises reported
eventbig=relv>50
event=relv>20 and relv<=50
evline=eventbig? widchan/cent*60 :event? widchan/cent*45: 0
negevline=eventbig? -widchan/cent*60 :event? -widchan/cent*35: 0
volcol = eventbig? color.blue: color.maroon

//firstplots
plot(evline,    title="volume event pos", color=volcol, style=plot.style_histogram, linewidth=2 )
plot(negevline, title="volume event neg", color=volcol, style=plot.style_histogram, linewidth=2 )

//PRAPARE AND EXECUTE PLOTS

//define swelling situations
chanswellup = widchan>widchan[1] and trendup
chanswelldown = widchan>widchan[1] and trenddown
chanswellside = widchan>widchan[1] and trendside
cogswellup = widcog>widcog[1] and trendup
cogswelldown = widcog>widcog[1] and trenddown
cogswellside = widcog>widcog[1] and trendside

//define colors
chancol = chanswellup? color.rgb(236, 177, 38, 00) : chanswelldown? color.maroon: chanswellside? color.rgb(245,124,00,00): trenddown? color.rgb(210,140,185,0): color.rgb(130,148,164,0)
cogcol = cogswellup ?  color.lime: cogswelldown? color.red: cogswellside? color.orange: color.black

//second plots
plot(trendup or trendside? perchan :na, title="channelwidth as percent of middle line", style=plot.style_columns, color=chancol)
plot(trendup or trendside? percog :na, title="COGwidth as percent of middle line", style=plot.style_columns, color=cogcol )
plot(trenddown or trendside? negperchan :na, title="channelwidth as percent negative", style=plot.style_columns, color=chancol )
plot(trenddown or trendside? negpercog :na, title="COGwidth as percent negative", style=plot.style_columns, color=cogcol )




