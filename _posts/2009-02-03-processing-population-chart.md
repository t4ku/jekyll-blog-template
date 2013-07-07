--- 
layout: post
title: "Processingでグラフ"
tags: 
- Programming
- Technology
- processing
status: publish
type: post
published: true
meta: 
  _edit_lock: "1250756744"
  _edit_last: "1736684"
  image_url: http://www.assoc-amazon.jp/e/ir?t=firstminutesb-22&l=as2&o=9&a=4873113784
  image_md5: d41d8cd98f00b204e9800998ecf8427e
  image_tag: ""
  image_colors_bg: s:224:"a:11:{i:0;s:6:"000000";s:2:"+1";s:6:"262626";s:2:"+2";s:6:"404040";s:2:"+3";s:6:"808080";s:2:"+4";s:6:"bfbfbf";s:2:"+5";s:6:"e6e6e6";i:-1;s:6:"000000";i:-2;s:6:"000000";i:-3;s:6:"000000";i:-4;s:6:"000000";i:-5;s:6:"000000";}";
  image_colors_fg: s:224:"a:11:{i:0;s:6:"808080";s:2:"+1";s:6:"bfbfbf";s:2:"+2";s:6:"ffffff";s:2:"+3";s:6:"000000";s:2:"+4";s:6:"000000";s:2:"+5";s:6:"000000";i:-1;s:6:"808080";i:-2;s:6:"808080";i:-3;s:6:"808080";i:-4;s:6:"808080";i:-5;s:6:"808080";}";
---
前から少し気になっていたProcessingという言語について、ちょうど本屋におもしろそうなオライリー本が出ていたので買ってみました。

<a href="http://www.amazon.co.jp/gp/product/4873113784?ie=UTF8&amp;tag=firstminutesb-22&amp;linkCode=as2&amp;camp=247&amp;creative=1211&amp;creativeASIN=4873113784">ビジュアライジング・データ ―Processingによる情報視覚化手法</a><img style="border:none!important;margin:0!important;" src="http://www.assoc-amazon.jp/e/ir?t=firstminutesb-22&amp;l=as2&amp;o=9&amp;a=4873113784" border="0" alt="" width="1" height="1" />

簡単な例としてグラフの作成があったので、自分でも日本の人口統計を使ってグラフを書いてみました。

<img class="size-full wp-image-44" title="Population growth in Japan" src="/img/uploads/2009/02/japan_pl_51.jpg" alt="Population growth in Japan" width="600" height="400" />

{% prism clike %}

FloatTable data;
PFont plotFont;
float plotX1,plotY1;
float plotX2,plotY2;
float labelX,labelY;
float dataAreaWidth;
float dataAreaHeight;
final float dataOffsetY = 30;
float volumnInterval = 10000;
float dataCeil;

int []years;
int yearMin;
int yearMax;
int yearInterval;

float dataMin;
float dataMax;

void setup(){
  plotFont = createFont("SansSerif",20);
  textFont(plotFont);
  //load csv file
  data = new FloatTable("ppl1920-1990-grs-t.csv");
  years = int(data.getRowNames());
  yearMin = years[0];
  yearMax = years[years.length -1];
  yearInterval = 10;
  dataMin = data.getColumnMin(0);
  dataMax = data.getColumnMax(0);
  println(ceil(dataMax/volumnInterval));
  dataCeil = ceil(dataMax/volumnInterval) * volumnInterval;
  size(600,400);
  smooth();
  plotX1 = 160;
  plotY1 = 80;
  plotX2 = width - plotX1/2;
  plotY2 = height - plotY1;
  labelX = plotX1 -110;
  labelY = plotY2 +40;
  dataAreaWidth = plotX2 - plotX1;
  dataAreaHeight = plotY2- plotY1;
}
void draw(){
  background(022); // (1)background color
  fill(044);
  rectMode(CORNERS);
  noStroke();
  rect(plotX1,plotY1,plotX2,plotY2);
  strokeWeight(0.4);
  stroke(254);
  drawDataArea();
  stroke(112);
  strokeWeight(0.1);
  drawLabels();
}
void drawLabels(){
  drawTitleLabel();
  drawAxisLabels();
  drawYearLabels();
  drawYearGrid();
  drawVolumnLabels();
}
void drawTitleLabel( ) {
  fill(112);
  textSize(15);
  textAlign(CENTER,CENTER);
  text("Population growth in Japan\n 1920 to 2000",(plotX1+plotX2)/2,plotY1 - 40);
}
void drawAxisLabels( ) {
  fill(112);
  textSize(13);
  textLeading(15);
  textAlign(CENTER, CENTER);
  text("Total \npopulation\n(Thousand)", labelX, (plotY1+plotY2)/2);
  textAlign(CENTER);
  text("Year", (plotX1+plotX2)/2, labelY);
}
void drawYearLabels(){
  fill(112);
  textSize(12);
  textAlign(LEFT);
  int rowCount = data.getRowCount();
  for(int i=0;i < rowCount;i++){
    if(years[i] % yearInterval ==0){
      String yearText = data.getRowName(i);
      float x = map(int(yearText),yearMin,yearMax,plotX1,plotX2);
      text(yearText,x,plotY2+12);
    }
  }
}
void drawYearGrid(){
int rowCount = data.getRowCount();
for(int i=0;i < rowCount;i++){
if(years[i] % yearInterval ==0){
float x = float(data.getRowName(i));
x = map(x,yearMin,yearMax,plotX1,plotX2);
line(x,plotY2,x,plotY1);
}
}
}
void drawVolumnLabels(){
fill(112);
textSize(10);
textAlign(RIGHT);
float x = plotX1;
float y;
for(float v = floor(dataMin/volumnInterval)*volumnInterval;v <= dataCeil;v+= volumnInterval){
y = map(v,floor(dataMin/volumnInterval)*volumnInterval,dataCeil,plotY2,plotY1);
line(x-10,y,x,y);
//println(v);
text(floor(v),x-20,y+4);
}
}
void drawDataArea(){
beginShape();
int rowCount = data.getRowCount();
for(int i=0;i < rowCount;i++){
float value = data.getFloat(i,0);
float x = map(years[i],yearMin,yearMax,plotX1,plotX2);
float y = map(value,floor(dataMin/volumnInterval)*volumnInterval,dataCeil,plotY2,plotY1);
vertex(x,y);
}
endShape();
}
void keyPressed( ) {
saveFrame("japan_pl_##.png");
}
{% endprism %}

processing実行後に何かキーを押すと、連番で"japan_pl_01.png"という画像が保存されます。FloatTableというクラスは本のサンプルコードに含まれているものですが、<a href="http://benfry.com/writing/archives/3" target="_blank">http://benfry.com/writing/archives/3</a> からダウンロードできます。人口のCSVは<a href="http://www.stat.go.jp/data/guide/2.htm" target="_blank">統計局</a>からダウンロードしたものをタブ区切りのcsvにしてます。

こんな感じで
<div class="robots" style="font-family:monospace;color:#006;border:1px solid #d0d0d0;background-color:#f0f0f0;">
<ol>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;">Year    Total</div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1920</span> <span style="color:#cc66cc;">55963</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1921</span> <span style="color:#cc66cc;">56666</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1922</span> <span style="color:#cc66cc;">57390</span></div></li>
	<li style="vertical-align:top;font-weight:bold;color:#006060;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1923</span> <span style="color:#cc66cc;">58119</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1924</span> <span style="color:#cc66cc;">58876</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1925</span> <span style="color:#cc66cc;">59737</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1926</span> <span style="color:#cc66cc;">60740.9</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1927</span> <span style="color:#cc66cc;">61659.3</span></div></li>
	<li style="vertical-align:top;font-weight:bold;color:#006060;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1928</span> <span style="color:#cc66cc;">62595.3</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1929</span> <span style="color:#cc66cc;">63460.6</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1930</span> <span style="color:#cc66cc;">64450</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1931</span> <span style="color:#cc66cc;">65457</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1932</span> <span style="color:#cc66cc;">66434</span></div></li>
	<li style="vertical-align:top;font-weight:bold;color:#006060;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1933</span> <span style="color:#cc66cc;">67432</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1934</span> <span style="color:#cc66cc;">68309</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1935</span> <span style="color:#cc66cc;">69254</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1936</span> <span style="color:#cc66cc;">70114</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1937</span> <span style="color:#cc66cc;">70630</span></div></li>
	<li style="vertical-align:top;font-weight:bold;color:#006060;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1938</span> <span style="color:#cc66cc;">71013</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1939</span> <span style="color:#cc66cc;">71380</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1940</span> <span style="color:#cc66cc;">71933</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1941</span> <span style="color:#cc66cc;">71933</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1942</span> <span style="color:#cc66cc;">71933</span></div></li>
	<li style="vertical-align:top;font-weight:bold;color:#006060;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1943</span> <span style="color:#cc66cc;">71933</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1944</span> <span style="color:#cc66cc;">73064</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1945</span> <span style="color:#cc66cc;">71998</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1946</span> <span style="color:#cc66cc;">73114</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1947</span> <span style="color:#cc66cc;">78101</span></div></li>
	<li style="vertical-align:top;font-weight:bold;color:#006060;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1948</span> <span style="color:#cc66cc;">80002</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1949</span> <span style="color:#cc66cc;">81773</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1950</span> <span style="color:#cc66cc;">83200</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1951</span> <span style="color:#cc66cc;">84573</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1952</span> <span style="color:#cc66cc;">85852</span></div></li>
	<li style="vertical-align:top;font-weight:bold;color:#006060;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1953</span> <span style="color:#cc66cc;">87033</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1954</span> <span style="color:#cc66cc;">88293</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1955</span> <span style="color:#cc66cc;">89276</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1956</span> <span style="color:#cc66cc;">90259</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1957</span> <span style="color:#cc66cc;">91088</span></div></li>
	<li style="vertical-align:top;font-weight:bold;color:#006060;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1958</span> <span style="color:#cc66cc;">92010</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1959</span> <span style="color:#cc66cc;">92973</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1960</span> <span style="color:#cc66cc;">93419</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1961</span> <span style="color:#cc66cc;">94285</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1962</span> <span style="color:#cc66cc;">95178</span></div></li>
	<li style="vertical-align:top;font-weight:bold;color:#006060;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1963</span> <span style="color:#cc66cc;">96156</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1964</span> <span style="color:#cc66cc;">97186</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1965</span> <span style="color:#cc66cc;">98275</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1966</span> <span style="color:#cc66cc;">99054</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1967</span> <span style="color:#cc66cc;">100243</span></div></li>
	<li style="vertical-align:top;font-weight:bold;color:#006060;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1968</span> <span style="color:#cc66cc;">101408</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1969</span> <span style="color:#cc66cc;">102648</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1970</span> <span style="color:#cc66cc;">103720</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1971</span> <span style="color:#cc66cc;">105014</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1972</span> <span style="color:#cc66cc;">107332</span></div></li>
	<li style="vertical-align:top;font-weight:bold;color:#006060;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1973</span> <span style="color:#cc66cc;">108710</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1974</span> <span style="color:#cc66cc;">110049</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1975</span> <span style="color:#cc66cc;">111940</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1976</span> <span style="color:#cc66cc;">110089</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1977</span> <span style="color:#cc66cc;">114154</span></div></li>
	<li style="vertical-align:top;font-weight:bold;color:#006060;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1978</span> <span style="color:#cc66cc;">115174</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1979</span> <span style="color:#cc66cc;">116133</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1980</span> <span style="color:#cc66cc;">117060</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1981</span> <span style="color:#cc66cc;">117884</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1982</span> <span style="color:#cc66cc;">118693</span></div></li>
	<li style="vertical-align:top;font-weight:bold;color:#006060;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1983</span> <span style="color:#cc66cc;">119483</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1984</span> <span style="color:#cc66cc;">120235</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1985</span> <span style="color:#cc66cc;">121049</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1986</span> <span style="color:#cc66cc;">121672</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1987</span> <span style="color:#cc66cc;">122264</span></div></li>
	<li style="vertical-align:top;font-weight:bold;color:#006060;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1988</span> <span style="color:#cc66cc;">122783</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1989</span> <span style="color:#cc66cc;">123255</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1990</span> <span style="color:#cc66cc;">123611</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1991</span> <span style="color:#cc66cc;">124043</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1992</span> <span style="color:#cc66cc;">124452</span></div></li>
	<li style="vertical-align:top;font-weight:bold;color:#006060;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1993</span> <span style="color:#cc66cc;">124764</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1994</span> <span style="color:#cc66cc;">125034</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1995</span> <span style="color:#cc66cc;">125570</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1996</span> <span style="color:#cc66cc;">125864</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1997</span> <span style="color:#cc66cc;">126166</span></div></li>
	<li style="vertical-align:top;font-weight:bold;color:#006060;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1998</span> <span style="color:#cc66cc;">126486</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">1999</span> <span style="color:#cc66cc;">126686</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;">
<div style="font:normal normal 1em/1.2em monospace;background:none;vertical-align:top;color:#000020;margin:0;padding:0;"><span style="color:#cc66cc;">2000</span> <span style="color:#cc66cc;">126926</span></div></li>
	<li style="font-weight:normal;vertical-align:top;font:normal normal 100% 'Courier New', Courier, monospace;color:#003030;"></li>
</ol>
</div>
Processingを一言で説明しようとすると、Javaの描写機能を使用しやすいインタフェースで提供する言語環境（なのかな？）でしょうが、なんでProcessingに興味をもっていたかというと、これを使用した作品（っていうのがいいですね）に興味深いというか斬新なものが多かったからな訳です。

MusicBox: a truly powerful visualization of your music library:<a href="http://www.crunchgear.com/2008/12/15/musicbox-a-truly-powerful-visualization-of-your-music-library/">http://www.crunchgear.com/2008/12/15/musicbox-a-truly-powerful-visualization-of-your-music-library/</a>
音楽のライブラリを可視化するソフト。ファイルのメタデータ（作者やジャンルなど）のみではなく、実際の音声データをスキャンしてマッピングする。

Aligning humans and mammals:<a href="http://benfry.com/infoseed/">http://benfry.com/infoseed/</a>
人間とほ乳類の遺伝子配列をならべて描写した画像

Base26:<a href="http://toxi.co.uk/p5/base26/">http://toxi.co.uk/p5/base26/</a>
英語の言葉同士の慣例性を表現した図。

仕事ではほとんどCRUDなWEBアプリケーションしか見ないのですが、こういうデータの重要性もさることながらどう見せるかでどれだけ印象や理解度が変わるかという事を考えるとおもしろい。色々遊んでみようと思います。
