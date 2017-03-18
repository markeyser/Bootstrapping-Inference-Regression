Title: Bootstrapping Inference in Regreesion
Date: 2015-03-08
Category: Blog

#### Summary
The purpose of this notebook is illustrate how apply bootstrap a confidence interval for the slope of the `MATH` independent variable in a multivariable regression environment using the `GENMOD` procedure in SAS.

This technique is useful in an inference problem where the interpretation of the Beta parameter is critical for the business problem and we are not really sure if our model follows all the GLM assumptions. 

This technique only is practicable with models that fit very fast.

#### Introduction
Apply bootstrapping to check the inference (p-value) of the coefficient estimates of the one or all explanatory variables. In this expample only the `MATH` Beta parameter. The aim of this consists in determine if the beta estimator of the one or all explanatory variables are statistically significant. Therefore, if the assumptions of our GLM model are correct the p-value we get in the output of the GLM model should be correct and our work finish here. But, if one or several assumptions of the GLM are not correct the inference is not valid. The bootstrap approach does not rely on any of these assumptions, and so it is likely giving a more accurate estimate of the coefficient estimates (and of the standard errors). I think it worth checking if the bootstrap confidence interval are the same of the GLM Wald confidence interval. 

Let's build up a SAS datasets to illustrate the bootstrapping technique:


```sas
options nosource;
```




<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN"
   "http://www.w3.org/TR/html4/strict.dtd">

<html>
<head>
  <title></title>
  <meta http-equiv="content-type" content="text/html; charset=None">
  <style type="text/css">
td.linenos { background-color: #f0f0f0; padding-right: 10px; }
span.lineno { background-color: #f0f0f0; padding: 0 5px 0 5px; }
pre { line-height: 125%; }
body .hll { background-color: #ffffcc }
body  { background: #ffffff; }
body .c { color: #0000FF } /* Comment */
body .k { color: #ff0000; font-weight: bold } /* Keyword */
body .n { color: #008000 } /* Name */
body .ch { color: #0000FF } /* Comment.Hashbang */
body .cm { color: #0000FF } /* Comment.Multiline */
body .cp { color: #0000FF } /* Comment.Preproc */
body .cpf { color: #0000FF } /* Comment.PreprocFile */
body .c1 { color: #0000FF } /* Comment.Single */
body .cs { color: #0000FF } /* Comment.Special */
body .kc { color: #ff0000; font-weight: bold } /* Keyword.Constant */
body .kd { color: #ff0000; font-weight: bold } /* Keyword.Declaration */
body .kn { color: #ff0000; font-weight: bold } /* Keyword.Namespace */
body .kp { color: #ff0000; font-weight: bold } /* Keyword.Pseudo */
body .kr { color: #ff0000; font-weight: bold } /* Keyword.Reserved */
body .kt { color: #ff0000; font-weight: bold } /* Keyword.Type */
body .s { color: #111111 } /* Literal.String */
body .na { color: #008000 } /* Name.Attribute */
body .nb { color: #008000 } /* Name.Builtin */
body .nc { color: #008000 } /* Name.Class */
body .no { color: #008000 } /* Name.Constant */
body .nd { color: #008000 } /* Name.Decorator */
body .ni { color: #008000 } /* Name.Entity */
body .ne { color: #008000 } /* Name.Exception */
body .nf { color: #008000 } /* Name.Function */
body .nl { color: #008000 } /* Name.Label */
body .nn { color: #008000 } /* Name.Namespace */
body .nx { color: #008000 } /* Name.Other */
body .py { color: #008000 } /* Name.Property */
body .nt { color: #008000 } /* Name.Tag */
body .nv { color: #008000 } /* Name.Variable */
body .sb { color: #111111 } /* Literal.String.Backtick */
body .sc { color: #111111 } /* Literal.String.Char */
body .sd { color: #111111 } /* Literal.String.Doc */
body .s2 { color: #111111 } /* Literal.String.Double */
body .se { color: #111111 } /* Literal.String.Escape */
body .sh { color: #111111 } /* Literal.String.Heredoc */
body .si { color: #111111 } /* Literal.String.Interpol */
body .sx { color: #111111 } /* Literal.String.Other */
body .sr { color: #111111 } /* Literal.String.Regex */
body .s1 { color: #111111 } /* Literal.String.Single */
body .ss { color: #111111 } /* Literal.String.Symbol */
body .bp { color: #008000 } /* Name.Builtin.Pseudo */
body .vc { color: #008000 } /* Name.Variable.Class */
body .vg { color: #008000 } /* Name.Variable.Global */
body .vi { color: #008000 } /* Name.Variable.Instance */

  </style>
</head>
<body>
<h2></h2>

<div class="highlight"><pre><span></span><span class="s">504  ods listing close;ods html5 file=stdout options(bitmap_mode=&#39;inline&#39;) device=png; ods graphics on / outputfmt=png;</span><br><span class="cm">NOTE: Writing HTML5 Body file: STDOUT</span><br><span class="s">505  </span><br><span class="s">506  options nosource;</span><br></pre></div>
</body>
</html>





```sas
data HSB2 ;
	infile datalines dsd
	;
input
	id
	female
	race
	ses
	schtyp
	prog
	read
	write
	math
	science
	socst
;
datalines;
70,0,4,1,1,1,57,52,41,47,57
121,1,4,2,1,3,68,59,53,63,61
86,0,4,3,1,1,44,33,54,58,31
141,0,4,3,1,3,63,44,47,53,56
172,0,4,2,1,2,47,52,57,53,61
113,0,4,2,1,2,44,52,51,63,61
50,0,3,2,1,1,50,59,42,53,61
11,0,1,2,1,2,34,46,45,39,36
84,0,4,2,1,1,63,57,54,58,51
48,0,3,2,1,2,57,55,52,50,51
75,0,4,2,1,3,60,46,51,53,61
60,0,4,2,1,2,57,65,51,63,61
95,0,4,3,1,2,73,60,71,61,71
104,0,4,3,1,2,54,63,57,55,46
38,0,3,1,1,2,45,57,50,31,56
115,0,4,1,1,1,42,49,43,50,56
76,0,4,3,1,2,47,52,51,50,56
195,0,4,2,2,1,57,57,60,58,56
114,0,4,3,1,2,68,65,62,55,61
85,0,4,2,1,1,55,39,57,53,46
167,0,4,2,1,1,63,49,35,66,41
143,0,4,2,1,3,63,63,75,72,66
41,0,3,2,1,2,50,40,45,55,56
20,0,1,3,1,2,60,52,57,61,61
12,0,1,2,1,3,37,44,45,39,46
53,0,3,2,1,3,34,37,46,39,31
154,0,4,3,1,2,65,65,66,61,66
178,0,4,2,2,3,47,57,57,58,46
196,0,4,3,2,2,44,38,49,39,46
29,0,2,1,1,1,52,44,49,55,41
126,0,4,2,1,1,42,31,57,47,51
103,0,4,3,1,2,76,52,64,64,61
192,0,4,3,2,2,65,67,63,66,71
150,0,4,2,1,3,42,41,57,72,31
199,0,4,3,2,2,52,59,50,61,61
144,0,4,3,1,1,60,65,58,61,66
200,0,4,2,2,2,68,54,75,66,66
80,0,4,3,1,2,65,62,68,66,66
16,0,1,1,1,3,47,31,44,36,36
153,0,4,2,1,3,39,31,40,39,51
176,0,4,2,2,2,47,47,41,42,51
177,0,4,2,2,2,55,59,62,58,51
168,0,4,2,1,2,52,54,57,55,51
40,0,3,1,1,1,42,41,43,50,41
62,0,4,3,1,1,65,65,48,63,66
169,0,4,1,1,1,55,59,63,69,46
49,0,3,3,1,3,50,40,39,49,47
136,0,4,2,1,2,65,59,70,63,51
189,0,4,2,2,2,47,59,63,53,46
7,0,1,2,1,2,57,54,59,47,51
27,0,2,2,1,2,53,61,61,57,56
128,0,4,3,1,2,39,33,38,47,41
21,0,1,2,1,1,44,44,61,50,46
183,0,4,2,2,2,63,59,49,55,71
132,0,4,2,1,2,73,62,73,69,66
15,0,1,3,1,3,39,39,44,26,42
67,0,4,1,1,3,37,37,42,33,32
22,0,1,2,1,3,42,39,39,56,46
185,0,4,2,2,2,63,57,55,58,41
9,0,1,2,1,3,48,49,52,44,51
181,0,4,2,2,2,50,46,45,58,61
170,0,4,3,1,2,47,62,61,69,66
134,0,4,1,1,1,44,44,39,34,46
108,0,4,2,1,1,34,33,41,36,36
197,0,4,3,2,2,50,42,50,36,61
140,0,4,2,1,3,44,41,40,50,26
171,0,4,2,1,2,60,54,60,55,66
107,0,4,1,1,3,47,39,47,42,26
81,0,4,1,1,2,63,43,59,65,44
18,0,1,2,1,3,50,33,49,44,36
155,0,4,2,1,1,44,44,46,39,51
97,0,4,3,1,2,60,54,58,58,61
68,0,4,2,1,2,73,67,71,63,66
157,0,4,2,1,1,68,59,58,74,66
56,0,4,2,1,3,55,45,46,58,51
5,0,1,1,1,2,47,40,43,45,31
159,0,4,3,1,2,55,61,54,49,61
123,0,4,3,1,1,68,59,56,63,66
164,0,4,2,1,3,31,36,46,39,46
14,0,1,3,1,2,47,41,54,42,56
127,0,4,3,1,2,63,59,57,55,56
165,0,4,1,1,3,36,49,54,61,36
174,0,4,2,2,2,68,59,71,66,56
3,0,1,1,1,2,63,65,48,63,56
58,0,4,2,1,3,55,41,40,44,41
146,0,4,3,1,2,55,62,64,63,66
102,0,4,3,1,2,52,41,51,53,56
117,0,4,3,1,3,34,49,39,42,56
133,0,4,2,1,3,50,31,40,34,31
94,0,4,3,1,2,55,49,61,61,56
24,0,2,2,1,2,52,62,66,47,46
149,0,4,1,1,1,63,49,49,66,46
82,1,4,3,1,2,68,62,65,69,61
8,1,1,1,1,2,39,44,52,44,48
129,1,4,1,1,1,44,44,46,47,51
173,1,4,1,1,1,50,62,61,63,51
57,1,4,2,1,2,71,65,72,66,56
100,1,4,3,1,2,63,65,71,69,71
1,1,1,1,1,3,34,44,40,39,41
194,1,4,3,2,2,63,63,69,61,61
88,1,4,3,1,2,68,60,64,69,66
99,1,4,3,1,1,47,59,56,66,61
47,1,3,1,1,2,47,46,49,33,41
120,1,4,3,1,2,63,52,54,50,51
166,1,4,2,1,2,52,59,53,61,51
65,1,4,2,1,2,55,54,66,42,56
101,1,4,3,1,2,60,62,67,50,56
89,1,4,1,1,3,35,35,40,51,33
54,1,3,1,2,1,47,54,46,50,56
180,1,4,3,2,2,71,65,69,58,71
162,1,4,2,1,3,57,52,40,61,56
4,1,1,1,1,2,44,50,41,39,51
131,1,4,3,1,2,65,59,57,46,66
125,1,4,1,1,2,68,65,58,59,56
34,1,1,3,2,2,73,61,57,55,66
106,1,4,2,1,3,36,44,37,42,41
130,1,4,3,1,1,43,54,55,55,46
93,1,4,3,1,2,73,67,62,58,66
163,1,4,1,1,2,52,57,64,58,56
37,1,3,1,1,3,41,47,40,39,51
35,1,1,1,2,1,60,54,50,50,51
87,1,4,2,1,1,50,52,46,50,56
73,1,4,2,1,2,50,52,53,39,56
151,1,4,2,1,3,47,46,52,48,46
44,1,3,1,1,3,47,62,45,34,46
152,1,4,3,1,2,55,57,56,58,61
105,1,4,2,1,2,50,41,45,44,56
28,1,2,2,1,1,39,53,54,50,41
91,1,4,3,1,3,50,49,56,47,46
45,1,3,1,1,3,34,35,41,29,26
116,1,4,2,1,2,57,59,54,50,56
33,1,2,1,1,2,57,65,72,54,56
66,1,4,2,1,3,68,62,56,50,51
72,1,4,2,1,3,42,54,47,47,46
77,1,4,1,1,2,61,59,49,44,66
61,1,4,3,1,2,76,63,60,67,66
190,1,4,2,2,2,47,59,54,58,46
42,1,3,2,1,3,46,52,55,44,56
2,1,1,2,1,3,39,41,33,42,41
55,1,3,2,2,2,52,49,49,44,61
19,1,1,1,1,1,28,46,43,44,51
90,1,4,3,1,2,42,54,50,50,52
142,1,4,2,1,3,47,42,52,39,51
17,1,1,2,1,2,47,57,48,44,41
122,1,4,2,1,2,52,59,58,53,66
191,1,4,3,2,2,47,52,43,48,61
83,1,4,2,1,3,50,62,41,55,31
182,1,4,2,2,2,44,52,43,44,51
6,1,1,1,1,2,47,41,46,40,41
46,1,3,1,1,2,45,55,44,34,41
43,1,3,1,1,2,47,37,43,42,46
96,1,4,3,1,2,65,54,61,58,56
138,1,4,2,1,3,43,57,40,50,51
10,1,1,2,1,1,47,54,49,53,61
71,1,4,2,1,1,57,62,56,58,66
139,1,4,2,1,2,68,59,61,55,71
110,1,4,2,1,3,52,55,50,54,61
148,1,4,2,1,3,42,57,51,47,61
109,1,4,2,1,1,42,39,42,42,41
39,1,3,3,1,2,66,67,67,61,66
147,1,4,1,1,2,47,62,53,53,61
74,1,4,2,1,2,57,50,50,51,58
198,1,4,3,2,2,47,61,51,63,31
161,1,4,1,1,2,57,62,72,61,61
112,1,4,2,1,2,52,59,48,55,61
69,1,4,1,1,3,44,44,40,40,31
156,1,4,2,1,2,50,59,53,61,61
111,1,4,1,1,1,39,54,39,47,36
186,1,4,2,2,2,57,62,63,55,41
98,1,4,1,1,3,57,60,51,53,37
119,1,4,1,1,1,42,57,45,50,43
13,1,1,2,1,3,47,46,39,47,61
51,1,3,3,1,1,42,36,42,31,39
26,1,2,3,1,2,60,59,62,61,51
36,1,3,1,1,1,44,49,44,35,51
135,1,4,1,1,2,63,60,65,54,66
59,1,4,2,1,2,65,67,63,55,71
78,1,4,2,1,2,39,54,54,53,41
64,1,4,3,1,3,50,52,45,58,36
63,1,4,1,1,1,52,65,60,56,51
79,1,4,2,1,2,60,62,49,50,51
193,1,4,2,2,2,44,49,48,39,51
92,1,4,3,1,1,52,67,57,63,61
160,1,4,2,1,2,55,65,55,50,61
32,1,2,3,1,3,50,67,66,66,56
23,1,2,1,1,2,65,65,64,58,71
158,1,4,2,1,1,52,54,55,53,51
25,1,2,2,1,1,47,44,42,42,36
188,1,4,3,2,2,63,62,56,55,61
52,1,3,1,1,2,50,46,53,53,66
124,1,4,1,1,3,42,54,41,42,41
175,1,4,3,2,1,36,57,42,50,41
;
```




<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN"
   "http://www.w3.org/TR/html4/strict.dtd">

<html>
<head>
  <title></title>
  <meta http-equiv="content-type" content="text/html; charset=None">
  <style type="text/css">
td.linenos { background-color: #f0f0f0; padding-right: 10px; }
span.lineno { background-color: #f0f0f0; padding: 0 5px 0 5px; }
pre { line-height: 125%; }
body .hll { background-color: #ffffcc }
body  { background: #ffffff; }
body .c { color: #0000FF } /* Comment */
body .k { color: #ff0000; font-weight: bold } /* Keyword */
body .n { color: #008000 } /* Name */
body .ch { color: #0000FF } /* Comment.Hashbang */
body .cm { color: #0000FF } /* Comment.Multiline */
body .cp { color: #0000FF } /* Comment.Preproc */
body .cpf { color: #0000FF } /* Comment.PreprocFile */
body .c1 { color: #0000FF } /* Comment.Single */
body .cs { color: #0000FF } /* Comment.Special */
body .kc { color: #ff0000; font-weight: bold } /* Keyword.Constant */
body .kd { color: #ff0000; font-weight: bold } /* Keyword.Declaration */
body .kn { color: #ff0000; font-weight: bold } /* Keyword.Namespace */
body .kp { color: #ff0000; font-weight: bold } /* Keyword.Pseudo */
body .kr { color: #ff0000; font-weight: bold } /* Keyword.Reserved */
body .kt { color: #ff0000; font-weight: bold } /* Keyword.Type */
body .s { color: #111111 } /* Literal.String */
body .na { color: #008000 } /* Name.Attribute */
body .nb { color: #008000 } /* Name.Builtin */
body .nc { color: #008000 } /* Name.Class */
body .no { color: #008000 } /* Name.Constant */
body .nd { color: #008000 } /* Name.Decorator */
body .ni { color: #008000 } /* Name.Entity */
body .ne { color: #008000 } /* Name.Exception */
body .nf { color: #008000 } /* Name.Function */
body .nl { color: #008000 } /* Name.Label */
body .nn { color: #008000 } /* Name.Namespace */
body .nx { color: #008000 } /* Name.Other */
body .py { color: #008000 } /* Name.Property */
body .nt { color: #008000 } /* Name.Tag */
body .nv { color: #008000 } /* Name.Variable */
body .sb { color: #111111 } /* Literal.String.Backtick */
body .sc { color: #111111 } /* Literal.String.Char */
body .sd { color: #111111 } /* Literal.String.Doc */
body .s2 { color: #111111 } /* Literal.String.Double */
body .se { color: #111111 } /* Literal.String.Escape */
body .sh { color: #111111 } /* Literal.String.Heredoc */
body .si { color: #111111 } /* Literal.String.Interpol */
body .sx { color: #111111 } /* Literal.String.Other */
body .sr { color: #111111 } /* Literal.String.Regex */
body .s1 { color: #111111 } /* Literal.String.Single */
body .ss { color: #111111 } /* Literal.String.Symbol */
body .bp { color: #008000 } /* Name.Builtin.Pseudo */
body .vc { color: #008000 } /* Name.Variable.Class */
body .vg { color: #008000 } /* Name.Variable.Global */
body .vi { color: #008000 } /* Name.Variable.Instance */

  </style>
</head>
<body>
<h2></h2>

<div class="highlight"><pre><span></span><span class="cm">NOTE: Writing HTML5 Body file: STDOUT</span><br><span class="cm">NOTE: The data set WORK.HSB2 has 192 observations and 11 variables.</span><br><span class="cm">NOTE: DATA statement used (Total process time):</span><br><span class="cm">      real time           0.02 seconds</span><br><span class="cm">      cpu time            0.00 seconds</span><br><span class="cm">      </span><br></pre></div>
</body>
</html>




We want to check if the coefficient estimate of the `MATH` explanatory variable really doesn’t contains the 0 value; that is if the `MATH` coefficient is statistically significant (p-value less than .05). 

### How can I bootstrap estimates in SAS?

Bootstrapping allows for estimation of statistics through the repeated resampling of data. In this page, we will demonstrate several methods of bootstrapping a confidence interval about the slope of the `MATH` explanatory variable in SAS. We will be using the `hsb2` dataset that
can be found here. We will begin by running an OLS regression, predicting `read` with `female`, `math`, `write`, and `ses`, and saving the slope of `math` value in a dataset called `t0`. The estimated `MATH` paramter value in this regression is 0.4333.


```sas
proc genmod data = HSB2 ;
  model read = female math write ses;
  ods output parameterestimates = t0;
run;
```




<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta charset="utf-8"/>
<meta content="SAS 9.4" name="generator"/>
<title>SAS Output</title>
<style>
/*<![CDATA[*/
.body.c section > table, .body.c section > pre, .body.c div > table,
.body.c div > pre, .body.c article > table, .body.c article > pre,
.body.j section > table, .body.j section > pre, .body.j div > table,
.body.j div > pre, .body.j article > table, .body.j article > pre,
.body.c p.note, .body.c p.warning, .body.c p.error, .body.c p.fatal,
.body.j p.note, .body.j p.warning, .body.j p.error, .body.j p.fatal,
.body.c > table.layoutcontainer, .body.j > table.layoutcontainer { margin-left: auto; margin-right: auto }
.layoutregion.l table, .layoutregion.l pre, .layoutregion.l p.note,
.layoutregion.l p.warning, .layoutregion.l p.error, .layoutregion.l p.fatal { margin-left: 0 }
.layoutregion.c table, .layoutregion.c pre, .layoutregion.c p.note,
.layoutregion.c p.warning, .layoutregion.c p.error, .layoutregion.c p.fatal { margin-left: auto; margin-right: auto }
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r p.warning, .layoutregion.r p.error, .layoutregion.r p.fatal { margin-right: 0 }
article, aside, details, figcaption, figure, footer, header, hgroup, nav, section { display: block }
html{ font-size: 100% }
.body { margin: 1em; font-size: 13px; line-height: 1.231 }
sup { position: relative; vertical-align: baseline; bottom: 0.25em; font-size: 0.8em }
sub { position: relative; vertical-align: baseline; top: 0.25em; font-size: 0.8em }
ul, ol { margin: 1em 0; padding: 0 0 0 40px }
dd { margin: 0 0 0 40px }
nav ul, nav ol { list-style: none; list-style-image: none; margin: 0; padding: 0 }
img { border: 0; vertical-align: middle }
svg:not(:root) { overflow: hidden }
figure { margin: 0 }
table { border-collapse: collapse; border-spacing: 0 }
.layoutcontainer { border-collapse: separate; border-spacing: 0 }
p { margin-top: 0; text-align: left }
span { text-align: left }
table { margin-bottom: 1em }
td, th { text-align: left; padding: 3px 6px; vertical-align: top }
td[class$="fixed"], th[class$="fixed"] { white-space: pre }
section, article { padding-top: 1px; padding-bottom: 8px }
hr.pagebreak { height: 0px; border: 0; border-bottom: 1px solid #c0c0c0; margin: 1em 0 }
.stacked-value { text-align: left; display: block }
.stacked-cell > .stacked-value, td.data > td.data, th.data > td.data, th.data > th.data, td.data > th.data, th.header > th.header { border: 0 }
.stacked-cell > div.data { border-width: 0 }
.systitleandfootercontainer { white-space: nowrap; margin-bottom: 1em }
.systitleandfootercontainer > p { margin: 0 }
.systitleandfootercontainer > p > span { display: inline-block; width: 100%; white-space: normal }
.batch { display: table }
.toc { display: none }
.proc_note_group, .proc_title_group { margin-bottom: 1em }
p.proctitle { margin: 0 }
p.note, p.warning, p.error, p.fatal { display: table }
.notebanner, .warnbanner, .errorbanner, .fatalbanner,
.notecontent, .warncontent, .errorcontent, .fatalcontent { display: table-cell; padding: 0.5em }
.notebanner, .warnbanner, .errorbanner, .fatalbanner { padding-right: 0 }
.body > div > ol li { text-align: left }
.c { text-align: center }
.r { text-align: right }
.l { text-align: left }
.j { text-align: justify }
.d { text-align: right }
.b { vertical-align: bottom }
.m { vertical-align: middle }
.t { vertical-align: top }
.aftercaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    padding-top: 4pt;
}
.batch > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.batch > tbody, .batch > thead, .batch > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.batch { border: hidden; }
.batch {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: 'SAS Monospace', 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    padding: 7px;
    }
.beforecaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.body {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    margin-left: 8px;
    margin-right: 8px;
}
.bodydate {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: right;
    vertical-align: top;
    width: 100%;
}
.bycontentfolder {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.byline {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.bylinecontainer > col, .bylinecontainer > colgroup > col, .bylinecontainer > colgroup, .bylinecontainer > tr, .bylinecontainer > * > tr, .bylinecontainer > thead, .bylinecontainer > tbody, .bylinecontainer > tfoot { border: none; }
.bylinecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.caption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.cell, .container {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.contentfolder, .contentitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.contentproclabel, .contentprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.contents {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.contentsdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.contenttitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.continued {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    width: 100%;
}
.data, .dataemphasis {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.dataemphasisfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.dataempty {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datafixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datastrong {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.datastrongfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.date {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.document {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.errorcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.extendedpage {
    background-color: #fafbfe;
    border-style: solid;
    border-width: 1pt;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
    text-align: center;
}
.fatalbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.fatalcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.fatalcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.folderaction {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.footer {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footeremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footeremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.footerstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.frame {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.graph > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.graph > tbody, .graph > thead, .graph > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.graph { border: hidden; }
.graph {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.header {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headeremphasis {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headeremphasisfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.headersandfooters {
    background-color: #edf2f9;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrong {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrongfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.index {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.indexaction, .indexitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.indexprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.indextitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.layoutcontainer, .layoutregion {
    border-width: 0;
    border-spacing: 30px;
}
.linecontent {
    background-color: #fafbfe;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.list {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.list10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.list2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.list3, .list4, .list5, .list6, .list7, .list8, .list9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.listitem10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.listitem3, .listitem4, .listitem5, .listitem6, .listitem7, .listitem8, .listitem9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.note {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notebanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.notecontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notecontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.output > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.output > tbody, .output > thead, .output > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.output { border: hidden; }
.output {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.pageno {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    text-align: right;
    vertical-align: top;
}
.pages {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.pagesdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.pagesitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.pagesproclabel, .pagesprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.pagestitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.paragraph {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.parskip > col, .parskip > colgroup > col, .parskip > colgroup, .parskip > tr, .parskip > * > tr, .parskip > thead, .parskip > tbody, .parskip > tfoot { border: none; }
.parskip {
    border: none;
    border-spacing: 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    }
.prepage {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: left;
}
.proctitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.proctitlefixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooter {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooteremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooteremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooterempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowfooterstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheader {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderemphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderemphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowheaderstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.systemfooter, .systemfooter10, .systemfooter2, .systemfooter3, .systemfooter4, .systemfooter5, .systemfooter6, .systemfooter7, .systemfooter8, .systemfooter9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.systemtitle, .systemtitle10, .systemtitle2, .systemtitle3, .systemtitle4, .systemtitle5, .systemtitle6, .systemtitle7, .systemtitle8, .systemtitle9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size: small;
    font-style: normal;
    font-weight: bold;
}
.systitleandfootercontainer > col, .systitleandfootercontainer > colgroup > col, .systitleandfootercontainer > colgroup, .systitleandfootercontainer > tr, .systitleandfootercontainer > * > tr, .systitleandfootercontainer > thead, .systitleandfootercontainer > tbody, .systitleandfootercontainer > tfoot { border: none; }
.systitleandfootercontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.table > col, .table > colgroup > col {
    border-left: 1px solid #c1c1c1;
    border-right: 0 solid #c1c1c1;
}
.table > tr, .table > * > tr {
    border-top: 1px solid #c1c1c1;
    border-bottom: 0 solid #c1c1c1;
}
.table { border: hidden; }
.table {
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 1px 0 0 1px;
    border-collapse: collapse;
    border-spacing: 0;
    }
.titleandnotecontainer > col, .titleandnotecontainer > colgroup > col, .titleandnotecontainer > colgroup, .titleandnotecontainer > tr, .titleandnotecontainer > * > tr, .titleandnotecontainer > thead, .titleandnotecontainer > tbody, .titleandnotecontainer > tfoot { border: none; }
.titleandnotecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.titlesandfooters {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.usertext {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warnbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.warncontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warncontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
/*]]>*/
</style>
</head>
<body class="l body">
<h1 class="body toc">SAS Output</h1>
<section data-name="Genmod" data-sec-type="proc">
<div id="IDX" class="systitleandfootercontainer" style="border-spacing: 1px">
<p><span class="c systemtitle">The SAS System</span> </p>
</div>
<div class="proc_title_group">
<p class="c proctitle">The GENMOD Procedure</p>
</div>
<h1 class="contentprocname toc">The GENMOD Procedure</h1>
<article>
<h1 class="contentitem toc">Model Information</h1>
<table class="table" style="border-spacing: 0">
<colgroup><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="2" scope="colgroup">Model Information</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">Data Set</th>
<td class="r data">WORK.HSB2</td>
</tr>
<tr>
<th class="rowheader" scope="row">Distribution</th>
<td class="r data">Normal</td>
</tr>
<tr>
<th class="rowheader" scope="row">Link Function</th>
<td class="r data">Identity</td>
</tr>
<tr>
<th class="rowheader" scope="row">Dependent Variable</th>
<td class="r data">read</td>
</tr>
</tbody>
</table>
</article>
<article id="IDX1">
<h1 class="contentitem toc">Number of Observations</h1>
<table class="table" style="border-spacing: 0">
<colgroup><col/><col/></colgroup>
<tbody>
<tr>
<th class="rowheader" scope="row">Number of Observations Read</th>
<td class="r data">192</td>
</tr>
<tr>
<th class="rowheader" scope="row">Number of Observations Used</th>
<td class="r data">192</td>
</tr>
</tbody>
</table>
</article>
<article id="IDX2">
<h1 class="contentitem toc">Criteria For Assessing Goodness Of Fit</h1>
<table class="table" style="border-spacing: 0">
<colgroup><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="4" scope="colgroup">Criteria For Assessing Goodness Of Fit</th>
</tr>
<tr>
<th class="b header" scope="col">Criterion</th>
<th class="r b header" scope="col">DF</th>
<th class="r b header" scope="col">Value</th>
<th class="r b header" scope="col">Value/DF</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">Deviance</th>
<td class="r data">187</td>
<td class="r data">9749.9643</td>
<td class="r data">52.1388</td>
</tr>
<tr>
<th class="rowheader" scope="row">Scaled Deviance</th>
<td class="r data">187</td>
<td class="r data">192.0000</td>
<td class="r data">1.0267</td>
</tr>
<tr>
<th class="rowheader" scope="row">Pearson Chi-Square</th>
<td class="r data">187</td>
<td class="r data">9749.9643</td>
<td class="r data">52.1388</td>
</tr>
<tr>
<th class="rowheader" scope="row">Scaled Pearson X2</th>
<td class="r data">187</td>
<td class="r data">192.0000</td>
<td class="r data">1.0267</td>
</tr>
<tr>
<th class="rowheader" scope="row">Log Likelihood</th>
<td class="r data">&#160;</td>
<td class="r data" style="white-space: nowrap">-649.4785</td>
<td class="r data">&#160;</td>
</tr>
<tr>
<th class="rowheader" scope="row">Full Log Likelihood</th>
<td class="r data">&#160;</td>
<td class="r data" style="white-space: nowrap">-649.4785</td>
<td class="r data">&#160;</td>
</tr>
<tr>
<th class="rowheader" scope="row">AIC (smaller is better)</th>
<td class="r data">&#160;</td>
<td class="r data">1310.9569</td>
<td class="r data">&#160;</td>
</tr>
<tr>
<th class="rowheader" scope="row">AICC (smaller is better)</th>
<td class="r data">&#160;</td>
<td class="r data">1311.4110</td>
<td class="r data">&#160;</td>
</tr>
<tr>
<th class="rowheader" scope="row">BIC (smaller is better)</th>
<td class="r data">&#160;</td>
<td class="r data">1330.5019</td>
<td class="r data">&#160;</td>
</tr>
</tbody>
</table>
</article>
<article id="IDX3">
<h1 class="contentitem toc">Convergence Status</h1>
<table class="table" style="border-spacing: 0">
<colgroup><col/></colgroup>
<tbody>
<tr>
<td class="data">Algorithm converged.</td>
</tr>
</tbody>
</table>
</article>
<article id="IDX4">
<h1 class="contentitem toc">Analysis Of Parameter Estimates</h1>
<table class="table" style="border-spacing: 0">
<colgroup><col/></colgroup><colgroup><col/><col/><col/><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="c b header" colspan="8" scope="colgroup">Analysis Of Maximum Likelihood Parameter Estimates</th>
</tr>
<tr>
<th class="b header" scope="col">Parameter</th>
<th class="r b header" scope="col">DF</th>
<th class="r b header" scope="col">Estimate</th>
<th class="r b header" scope="col">Standard<br/>Error</th>
<th class="c b header" colspan="2" scope="colgroup">Wald 95% Confidence Limits</th>
<th class="r b header" scope="col">Wald Chi-Square</th>
<th class="r b header" scope="col">Pr&#160;&gt;&#160;ChiSq</th>
</tr>
</thead>
<tbody>
<tr>
<th class="rowheader" scope="row">Intercept</th>
<td class="r data">1</td>
<td class="r data">6.3343</td>
<td class="r data">3.2981</td>
<td class="r data" style="white-space: nowrap">-0.1299</td>
<td class="r data">12.7985</td>
<td class="r data">3.69</td>
<td class="r data">0.0548</td>
</tr>
<tr>
<th class="rowheader" scope="row">female</th>
<td class="r data">1</td>
<td class="r data" style="white-space: nowrap">-2.4704</td>
<td class="r data">1.1133</td>
<td class="r data" style="white-space: nowrap">-4.6525</td>
<td class="r data" style="white-space: nowrap">-0.2883</td>
<td class="r data">4.92</td>
<td class="r data">0.0265</td>
</tr>
<tr>
<th class="rowheader" scope="row">math</th>
<td class="r data">1</td>
<td class="r data">0.4333</td>
<td class="r data">0.0734</td>
<td class="r data">0.2894</td>
<td class="r data">0.5773</td>
<td class="r data">34.82</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<th class="rowheader" scope="row">write</th>
<td class="r data">1</td>
<td class="r data">0.4106</td>
<td class="r data">0.0744</td>
<td class="r data">0.2647</td>
<td class="r data">0.5564</td>
<td class="r data">30.43</td>
<td class="r data">&lt;.0001</td>
</tr>
<tr>
<th class="rowheader" scope="row">ses</th>
<td class="r data">1</td>
<td class="r data">1.3787</td>
<td class="r data">0.7423</td>
<td class="r data" style="white-space: nowrap">-0.0761</td>
<td class="r data">2.8336</td>
<td class="r data">3.45</td>
<td class="r data">0.0633</td>
</tr>
<tr>
<th class="rowheader" scope="row">Scale</th>
<td class="r data">1</td>
<td class="r data">7.1261</td>
<td class="r data">0.3637</td>
<td class="r data">6.4478</td>
<td class="r data">7.8757</td>
<td class="r data">&#160;</td>
<td class="r data">&#160;</td>
</tr>
</tbody>
</table>
<div class="proc_note_group">
<p class="c note"><span class="c notebanner">Note:</span><span class="c notecontent">The scale parameter was estimated by maximum likelihood.</span></p>
</div>
</article>
</section>
</body>
</html>




Store the estimated `MATH` parameter:


```sas
data _null_;
    set t0;
    if parameter =  "math" then call symput('est_bar', estimate);
run;

%put &est_bar;
```




<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN"
   "http://www.w3.org/TR/html4/strict.dtd">

<html>
<head>
  <title></title>
  <meta http-equiv="content-type" content="text/html; charset=None">
  <style type="text/css">
td.linenos { background-color: #f0f0f0; padding-right: 10px; }
span.lineno { background-color: #f0f0f0; padding: 0 5px 0 5px; }
pre { line-height: 125%; }
body .hll { background-color: #ffffcc }
body  { background: #ffffff; }
body .c { color: #0000FF } /* Comment */
body .k { color: #ff0000; font-weight: bold } /* Keyword */
body .n { color: #008000 } /* Name */
body .ch { color: #0000FF } /* Comment.Hashbang */
body .cm { color: #0000FF } /* Comment.Multiline */
body .cp { color: #0000FF } /* Comment.Preproc */
body .cpf { color: #0000FF } /* Comment.PreprocFile */
body .c1 { color: #0000FF } /* Comment.Single */
body .cs { color: #0000FF } /* Comment.Special */
body .kc { color: #ff0000; font-weight: bold } /* Keyword.Constant */
body .kd { color: #ff0000; font-weight: bold } /* Keyword.Declaration */
body .kn { color: #ff0000; font-weight: bold } /* Keyword.Namespace */
body .kp { color: #ff0000; font-weight: bold } /* Keyword.Pseudo */
body .kr { color: #ff0000; font-weight: bold } /* Keyword.Reserved */
body .kt { color: #ff0000; font-weight: bold } /* Keyword.Type */
body .s { color: #111111 } /* Literal.String */
body .na { color: #008000 } /* Name.Attribute */
body .nb { color: #008000 } /* Name.Builtin */
body .nc { color: #008000 } /* Name.Class */
body .no { color: #008000 } /* Name.Constant */
body .nd { color: #008000 } /* Name.Decorator */
body .ni { color: #008000 } /* Name.Entity */
body .ne { color: #008000 } /* Name.Exception */
body .nf { color: #008000 } /* Name.Function */
body .nl { color: #008000 } /* Name.Label */
body .nn { color: #008000 } /* Name.Namespace */
body .nx { color: #008000 } /* Name.Other */
body .py { color: #008000 } /* Name.Property */
body .nt { color: #008000 } /* Name.Tag */
body .nv { color: #008000 } /* Name.Variable */
body .sb { color: #111111 } /* Literal.String.Backtick */
body .sc { color: #111111 } /* Literal.String.Char */
body .sd { color: #111111 } /* Literal.String.Doc */
body .s2 { color: #111111 } /* Literal.String.Double */
body .se { color: #111111 } /* Literal.String.Escape */
body .sh { color: #111111 } /* Literal.String.Heredoc */
body .si { color: #111111 } /* Literal.String.Interpol */
body .sx { color: #111111 } /* Literal.String.Other */
body .sr { color: #111111 } /* Literal.String.Regex */
body .s1 { color: #111111 } /* Literal.String.Single */
body .ss { color: #111111 } /* Literal.String.Symbol */
body .bp { color: #008000 } /* Name.Builtin.Pseudo */
body .vc { color: #008000 } /* Name.Variable.Class */
body .vg { color: #008000 } /* Name.Variable.Global */
body .vi { color: #008000 } /* Name.Variable.Instance */

  </style>
</head>
<body>
<h2></h2>

<div class="highlight"><pre><span></span><span class="cm">NOTE: Writing HTML5 Body file: STDOUT</span><br><span class="cm">NOTE: Numeric values have been converted to character values at the places given by: (Line):(Column).</span><br><span class="cm">      738:56   </span><br><span class="cm">NOTE: There were 6 observations read from the data set WORK.T0.</span><br><span class="cm">NOTE: DATA statement used (Total process time):</span><br><span class="cm">      real time           0.00 seconds</span><br><span class="cm">      cpu time            0.00 seconds</span><br><span class="cm">      </span><br><span class="s">0.4333340545</span><br></pre></div>
</body>
</html>




To bootstrap a confidence interval about this `MATH` beta value, we will first need to resample.  This step involves sampling with replacement from our original dataset
to generate a new dataset the same size as our original dataset.  For each of these
samples, we will be running the same regression as above and saving the `MATH` beta parameter
value.  `proc surveyselect` allows us to do this resampling in one step.

Before carrying out this step, let's outline the assumptions we are making about our
data when we use this method. We are assuming that the observations in our dataset
are independent. We are also assuming that the statistic we are estimating is
asymptotically normally distributed.

We indicate an output dataset, a seed, a sampling method, and the number of
replicates.  The sampling method indicated, `urs`, is unrestricted random sampling,
or sampling with replacement.  The samprate indicates how large each sample should
be relative to the input dataset.  A `samprate` of one means that the sampled datasets
should be of the same size as the input dataset.  So in this example, we will generate
500 datasets of 200, so our output dataset `bootsample` will have 100,000 observations.


```sas
%let rep = 100000;
proc surveyselect data= HSB2 out=bootsample 
     seed = 1347 method = urs
	 samprate = 1 outhits rep = &rep;
run;
```




<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta charset="utf-8"/>
<meta content="SAS 9.4" name="generator"/>
<title>SAS Output</title>
<style>
/*<![CDATA[*/
.body.c section > table, .body.c section > pre, .body.c div > table,
.body.c div > pre, .body.c article > table, .body.c article > pre,
.body.j section > table, .body.j section > pre, .body.j div > table,
.body.j div > pre, .body.j article > table, .body.j article > pre,
.body.c p.note, .body.c p.warning, .body.c p.error, .body.c p.fatal,
.body.j p.note, .body.j p.warning, .body.j p.error, .body.j p.fatal,
.body.c > table.layoutcontainer, .body.j > table.layoutcontainer { margin-left: auto; margin-right: auto }
.layoutregion.l table, .layoutregion.l pre, .layoutregion.l p.note,
.layoutregion.l p.warning, .layoutregion.l p.error, .layoutregion.l p.fatal { margin-left: 0 }
.layoutregion.c table, .layoutregion.c pre, .layoutregion.c p.note,
.layoutregion.c p.warning, .layoutregion.c p.error, .layoutregion.c p.fatal { margin-left: auto; margin-right: auto }
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r p.warning, .layoutregion.r p.error, .layoutregion.r p.fatal { margin-right: 0 }
article, aside, details, figcaption, figure, footer, header, hgroup, nav, section { display: block }
html{ font-size: 100% }
.body { margin: 1em; font-size: 13px; line-height: 1.231 }
sup { position: relative; vertical-align: baseline; bottom: 0.25em; font-size: 0.8em }
sub { position: relative; vertical-align: baseline; top: 0.25em; font-size: 0.8em }
ul, ol { margin: 1em 0; padding: 0 0 0 40px }
dd { margin: 0 0 0 40px }
nav ul, nav ol { list-style: none; list-style-image: none; margin: 0; padding: 0 }
img { border: 0; vertical-align: middle }
svg:not(:root) { overflow: hidden }
figure { margin: 0 }
table { border-collapse: collapse; border-spacing: 0 }
.layoutcontainer { border-collapse: separate; border-spacing: 0 }
p { margin-top: 0; text-align: left }
span { text-align: left }
table { margin-bottom: 1em }
td, th { text-align: left; padding: 3px 6px; vertical-align: top }
td[class$="fixed"], th[class$="fixed"] { white-space: pre }
section, article { padding-top: 1px; padding-bottom: 8px }
hr.pagebreak { height: 0px; border: 0; border-bottom: 1px solid #c0c0c0; margin: 1em 0 }
.stacked-value { text-align: left; display: block }
.stacked-cell > .stacked-value, td.data > td.data, th.data > td.data, th.data > th.data, td.data > th.data, th.header > th.header { border: 0 }
.stacked-cell > div.data { border-width: 0 }
.systitleandfootercontainer { white-space: nowrap; margin-bottom: 1em }
.systitleandfootercontainer > p { margin: 0 }
.systitleandfootercontainer > p > span { display: inline-block; width: 100%; white-space: normal }
.batch { display: table }
.toc { display: none }
.proc_note_group, .proc_title_group { margin-bottom: 1em }
p.proctitle { margin: 0 }
p.note, p.warning, p.error, p.fatal { display: table }
.notebanner, .warnbanner, .errorbanner, .fatalbanner,
.notecontent, .warncontent, .errorcontent, .fatalcontent { display: table-cell; padding: 0.5em }
.notebanner, .warnbanner, .errorbanner, .fatalbanner { padding-right: 0 }
.body > div > ol li { text-align: left }
.c { text-align: center }
.r { text-align: right }
.l { text-align: left }
.j { text-align: justify }
.d { text-align: right }
.b { vertical-align: bottom }
.m { vertical-align: middle }
.t { vertical-align: top }
.aftercaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    padding-top: 4pt;
}
.batch > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.batch > tbody, .batch > thead, .batch > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.batch { border: hidden; }
.batch {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: 'SAS Monospace', 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    padding: 7px;
    }
.beforecaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.body {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    margin-left: 8px;
    margin-right: 8px;
}
.bodydate {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: right;
    vertical-align: top;
    width: 100%;
}
.bycontentfolder {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.byline {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.bylinecontainer > col, .bylinecontainer > colgroup > col, .bylinecontainer > colgroup, .bylinecontainer > tr, .bylinecontainer > * > tr, .bylinecontainer > thead, .bylinecontainer > tbody, .bylinecontainer > tfoot { border: none; }
.bylinecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.caption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.cell, .container {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.contentfolder, .contentitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.contentproclabel, .contentprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.contents {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.contentsdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.contenttitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.continued {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    width: 100%;
}
.data, .dataemphasis {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.dataemphasisfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.dataempty {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datafixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datastrong {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.datastrongfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.date {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.document {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.errorcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.extendedpage {
    background-color: #fafbfe;
    border-style: solid;
    border-width: 1pt;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
    text-align: center;
}
.fatalbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.fatalcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.fatalcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.folderaction {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.footer {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footeremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footeremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.footerstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.frame {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.graph > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.graph > tbody, .graph > thead, .graph > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.graph { border: hidden; }
.graph {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.header {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headeremphasis {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headeremphasisfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.headersandfooters {
    background-color: #edf2f9;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrong {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrongfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.index {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.indexaction, .indexitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.indexprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.indextitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.layoutcontainer, .layoutregion {
    border-width: 0;
    border-spacing: 30px;
}
.linecontent {
    background-color: #fafbfe;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.list {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.list10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.list2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.list3, .list4, .list5, .list6, .list7, .list8, .list9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.listitem10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.listitem3, .listitem4, .listitem5, .listitem6, .listitem7, .listitem8, .listitem9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.note {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notebanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.notecontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notecontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.output > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.output > tbody, .output > thead, .output > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.output { border: hidden; }
.output {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.pageno {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    text-align: right;
    vertical-align: top;
}
.pages {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.pagesdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.pagesitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.pagesproclabel, .pagesprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.pagestitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.paragraph {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.parskip > col, .parskip > colgroup > col, .parskip > colgroup, .parskip > tr, .parskip > * > tr, .parskip > thead, .parskip > tbody, .parskip > tfoot { border: none; }
.parskip {
    border: none;
    border-spacing: 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    }
.prepage {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: left;
}
.proctitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.proctitlefixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooter {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooteremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooteremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooterempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowfooterstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheader {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderemphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderemphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowheaderstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.systemfooter, .systemfooter10, .systemfooter2, .systemfooter3, .systemfooter4, .systemfooter5, .systemfooter6, .systemfooter7, .systemfooter8, .systemfooter9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.systemtitle, .systemtitle10, .systemtitle2, .systemtitle3, .systemtitle4, .systemtitle5, .systemtitle6, .systemtitle7, .systemtitle8, .systemtitle9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size: small;
    font-style: normal;
    font-weight: bold;
}
.systitleandfootercontainer > col, .systitleandfootercontainer > colgroup > col, .systitleandfootercontainer > colgroup, .systitleandfootercontainer > tr, .systitleandfootercontainer > * > tr, .systitleandfootercontainer > thead, .systitleandfootercontainer > tbody, .systitleandfootercontainer > tfoot { border: none; }
.systitleandfootercontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.table > col, .table > colgroup > col {
    border-left: 1px solid #c1c1c1;
    border-right: 0 solid #c1c1c1;
}
.table > tr, .table > * > tr {
    border-top: 1px solid #c1c1c1;
    border-bottom: 0 solid #c1c1c1;
}
.table { border: hidden; }
.table {
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 1px 0 0 1px;
    border-collapse: collapse;
    border-spacing: 0;
    }
.titleandnotecontainer > col, .titleandnotecontainer > colgroup > col, .titleandnotecontainer > colgroup, .titleandnotecontainer > tr, .titleandnotecontainer > * > tr, .titleandnotecontainer > thead, .titleandnotecontainer > tbody, .titleandnotecontainer > tfoot { border: none; }
.titleandnotecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.titlesandfooters {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.usertext {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warnbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.warncontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warncontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
/*]]>*/
</style>
</head>
<body class="l body">
<h1 class="body toc">SAS Output</h1>
<section data-name="Surveyselect" data-sec-type="proc">
<div id="IDX" class="systitleandfootercontainer" style="border-spacing: 1px">
<p><span class="c systemtitle">The SAS System</span> </p>
</div>
<div class="proc_title_group">
<p class="c proctitle">The SURVEYSELECT Procedure</p>
</div>
<h1 class="contentprocname toc">The SURVEYSELECT Procedure</h1>
<article>
<h1 class="contentitem toc">Sample Selection Method</h1>
<table class="table" style="border-spacing: 0">
<colgroup><col/><col/></colgroup>
<tbody>
<tr>
<th class="rowheader" scope="row">Selection Method</th>
<td class="r data">Unrestricted Random Sampling</td>
</tr>
</tbody>
</table>
</article>
<article id="IDX1">
<h1 class="contentitem toc">Sample Selection Summary</h1>
<table class="table" style="border-spacing: 0">
<colgroup><col/><col/></colgroup>
<tbody>
<tr>
<th class="rowheader" scope="row">Input Data Set</th>
<td class="r data">HSB2</td>
</tr>
<tr>
<th class="rowheader" scope="row">Random Number Seed</th>
<td class="r data">1347</td>
</tr>
<tr>
<th class="rowheader" scope="row">Sampling Rate</th>
<td class="r data">1</td>
</tr>
<tr>
<th class="rowheader" scope="row">Sample Size</th>
<td class="r data">192</td>
</tr>
<tr>
<th class="rowheader" scope="row">Expected Number of Hits</th>
<td class="r data">1</td>
</tr>
<tr>
<th class="rowheader" scope="row">Sampling Weight</th>
<td class="r data">1</td>
</tr>
<tr>
<th class="rowheader" scope="row">Number of Replicates</th>
<td class="r data">100000</td>
</tr>
<tr>
<th class="rowheader" scope="row">Total Sample Size</th>
<td class="r data">19200000</td>
</tr>
<tr>
<th class="rowheader" scope="row">Output Data Set</th>
<td class="r data">BOOTSAMPLE</td>
</tr>
</tbody>
</table>
</article>
</section>
</body>
</html>




With this dataset, we will now run our regression model, specifying by replicate so
that the model will be run separately for each of the 100000 sample datasets. After that,
we use a data step to convert the `MATH` beta paramter values to numeric.


```sas
ods select none;
ods output parameterestimates = tdata (where = (parameter =  "math"));
proc genmod data = bootsample;
  by replicate;
  model read = female math write ses;
  ods output parameterestimates = t (where = (parameter =  "math"));
run;
quit;
```




<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta charset="utf-8"/>
<meta content="SAS 9.4" name="generator"/>
<title>SAS Output</title>
<style>
/*<![CDATA[*/
.body.c section > table, .body.c section > pre, .body.c div > table,
.body.c div > pre, .body.c article > table, .body.c article > pre,
.body.j section > table, .body.j section > pre, .body.j div > table,
.body.j div > pre, .body.j article > table, .body.j article > pre,
.body.c p.note, .body.c p.warning, .body.c p.error, .body.c p.fatal,
.body.j p.note, .body.j p.warning, .body.j p.error, .body.j p.fatal,
.body.c > table.layoutcontainer, .body.j > table.layoutcontainer { margin-left: auto; margin-right: auto }
.layoutregion.l table, .layoutregion.l pre, .layoutregion.l p.note,
.layoutregion.l p.warning, .layoutregion.l p.error, .layoutregion.l p.fatal { margin-left: 0 }
.layoutregion.c table, .layoutregion.c pre, .layoutregion.c p.note,
.layoutregion.c p.warning, .layoutregion.c p.error, .layoutregion.c p.fatal { margin-left: auto; margin-right: auto }
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r p.warning, .layoutregion.r p.error, .layoutregion.r p.fatal { margin-right: 0 }
article, aside, details, figcaption, figure, footer, header, hgroup, nav, section { display: block }
html{ font-size: 100% }
.body { margin: 1em; font-size: 13px; line-height: 1.231 }
sup { position: relative; vertical-align: baseline; bottom: 0.25em; font-size: 0.8em }
sub { position: relative; vertical-align: baseline; top: 0.25em; font-size: 0.8em }
ul, ol { margin: 1em 0; padding: 0 0 0 40px }
dd { margin: 0 0 0 40px }
nav ul, nav ol { list-style: none; list-style-image: none; margin: 0; padding: 0 }
img { border: 0; vertical-align: middle }
svg:not(:root) { overflow: hidden }
figure { margin: 0 }
table { border-collapse: collapse; border-spacing: 0 }
.layoutcontainer { border-collapse: separate; border-spacing: 0 }
p { margin-top: 0; text-align: left }
span { text-align: left }
table { margin-bottom: 1em }
td, th { text-align: left; padding: 3px 6px; vertical-align: top }
td[class$="fixed"], th[class$="fixed"] { white-space: pre }
section, article { padding-top: 1px; padding-bottom: 8px }
hr.pagebreak { height: 0px; border: 0; border-bottom: 1px solid #c0c0c0; margin: 1em 0 }
.stacked-value { text-align: left; display: block }
.stacked-cell > .stacked-value, td.data > td.data, th.data > td.data, th.data > th.data, td.data > th.data, th.header > th.header { border: 0 }
.stacked-cell > div.data { border-width: 0 }
.systitleandfootercontainer { white-space: nowrap; margin-bottom: 1em }
.systitleandfootercontainer > p { margin: 0 }
.systitleandfootercontainer > p > span { display: inline-block; width: 100%; white-space: normal }
.batch { display: table }
.toc { display: none }
.proc_note_group, .proc_title_group { margin-bottom: 1em }
p.proctitle { margin: 0 }
p.note, p.warning, p.error, p.fatal { display: table }
.notebanner, .warnbanner, .errorbanner, .fatalbanner,
.notecontent, .warncontent, .errorcontent, .fatalcontent { display: table-cell; padding: 0.5em }
.notebanner, .warnbanner, .errorbanner, .fatalbanner { padding-right: 0 }
.body > div > ol li { text-align: left }
.c { text-align: center }
.r { text-align: right }
.l { text-align: left }
.j { text-align: justify }
.d { text-align: right }
.b { vertical-align: bottom }
.m { vertical-align: middle }
.t { vertical-align: top }
.aftercaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    padding-top: 4pt;
}
.batch > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.batch > tbody, .batch > thead, .batch > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.batch { border: hidden; }
.batch {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: 'SAS Monospace', 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    padding: 7px;
    }
.beforecaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.body {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    margin-left: 8px;
    margin-right: 8px;
}
.bodydate {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: right;
    vertical-align: top;
    width: 100%;
}
.bycontentfolder {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.byline {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.bylinecontainer > col, .bylinecontainer > colgroup > col, .bylinecontainer > colgroup, .bylinecontainer > tr, .bylinecontainer > * > tr, .bylinecontainer > thead, .bylinecontainer > tbody, .bylinecontainer > tfoot { border: none; }
.bylinecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.caption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.cell, .container {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.contentfolder, .contentitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.contentproclabel, .contentprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.contents {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.contentsdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.contenttitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.continued {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    width: 100%;
}
.data, .dataemphasis {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.dataemphasisfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.dataempty {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datafixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datastrong {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.datastrongfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.date {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.document {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.errorcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.extendedpage {
    background-color: #fafbfe;
    border-style: solid;
    border-width: 1pt;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
    text-align: center;
}
.fatalbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.fatalcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.fatalcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.folderaction {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.footer {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footeremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footeremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.footerstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.frame {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.graph > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.graph > tbody, .graph > thead, .graph > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.graph { border: hidden; }
.graph {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.header {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headeremphasis {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headeremphasisfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.headersandfooters {
    background-color: #edf2f9;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrong {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrongfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.index {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.indexaction, .indexitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.indexprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.indextitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.layoutcontainer, .layoutregion {
    border-width: 0;
    border-spacing: 30px;
}
.linecontent {
    background-color: #fafbfe;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.list {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.list10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.list2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.list3, .list4, .list5, .list6, .list7, .list8, .list9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.listitem10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.listitem3, .listitem4, .listitem5, .listitem6, .listitem7, .listitem8, .listitem9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.note {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notebanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.notecontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notecontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.output > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.output > tbody, .output > thead, .output > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.output { border: hidden; }
.output {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.pageno {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    text-align: right;
    vertical-align: top;
}
.pages {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.pagesdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.pagesitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.pagesproclabel, .pagesprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.pagestitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.paragraph {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.parskip > col, .parskip > colgroup > col, .parskip > colgroup, .parskip > tr, .parskip > * > tr, .parskip > thead, .parskip > tbody, .parskip > tfoot { border: none; }
.parskip {
    border: none;
    border-spacing: 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    }
.prepage {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: left;
}
.proctitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.proctitlefixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooter {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooteremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooteremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooterempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowfooterstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheader {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderemphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderemphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowheaderstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.systemfooter, .systemfooter10, .systemfooter2, .systemfooter3, .systemfooter4, .systemfooter5, .systemfooter6, .systemfooter7, .systemfooter8, .systemfooter9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.systemtitle, .systemtitle10, .systemtitle2, .systemtitle3, .systemtitle4, .systemtitle5, .systemtitle6, .systemtitle7, .systemtitle8, .systemtitle9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size: small;
    font-style: normal;
    font-weight: bold;
}
.systitleandfootercontainer > col, .systitleandfootercontainer > colgroup > col, .systitleandfootercontainer > colgroup, .systitleandfootercontainer > tr, .systitleandfootercontainer > * > tr, .systitleandfootercontainer > thead, .systitleandfootercontainer > tbody, .systitleandfootercontainer > tfoot { border: none; }
.systitleandfootercontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.table > col, .table > colgroup > col {
    border-left: 1px solid #c1c1c1;
    border-right: 0 solid #c1c1c1;
}
.table > tr, .table > * > tr {
    border-top: 1px solid #c1c1c1;
    border-bottom: 0 solid #c1c1c1;
}
.table { border: hidden; }
.table {
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 1px 0 0 1px;
    border-collapse: collapse;
    border-spacing: 0;
    }
.titleandnotecontainer > col, .titleandnotecontainer > colgroup > col, .titleandnotecontainer > colgroup, .titleandnotecontainer > tr, .titleandnotecontainer > * > tr, .titleandnotecontainer > thead, .titleandnotecontainer > tbody, .titleandnotecontainer > tfoot { border: none; }
.titleandnotecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.titlesandfooters {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.usertext {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warnbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.warncontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warncontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
/*]]>*/
</style>
</head>
<body class="l body">
<h1 class="body toc">SAS Output</h1>
<section data-name="Genmod" data-sec-type="proc">
</section>
</body>
</html>




NOTE: The `ods select none` suppresses displayed output in SAS `GENMOD`. The `noprint` options deosn't work in this procedure. More about the use of the `ODS` system in this example see [this](http://documentation.sas.com/?docsetId=statug&docsetVersion=14.2&docsetTarget=statug_ods_examples04.htm&locale=en)


```sas
ods select all;
```




<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01//EN"
   "http://www.w3.org/TR/html4/strict.dtd">

<html>
<head>
  <title></title>
  <meta http-equiv="content-type" content="text/html; charset=None">
  <style type="text/css">
td.linenos { background-color: #f0f0f0; padding-right: 10px; }
span.lineno { background-color: #f0f0f0; padding: 0 5px 0 5px; }
pre { line-height: 125%; }
body .hll { background-color: #ffffcc }
body  { background: #ffffff; }
body .c { color: #0000FF } /* Comment */
body .k { color: #ff0000; font-weight: bold } /* Keyword */
body .n { color: #008000 } /* Name */
body .ch { color: #0000FF } /* Comment.Hashbang */
body .cm { color: #0000FF } /* Comment.Multiline */
body .cp { color: #0000FF } /* Comment.Preproc */
body .cpf { color: #0000FF } /* Comment.PreprocFile */
body .c1 { color: #0000FF } /* Comment.Single */
body .cs { color: #0000FF } /* Comment.Special */
body .kc { color: #ff0000; font-weight: bold } /* Keyword.Constant */
body .kd { color: #ff0000; font-weight: bold } /* Keyword.Declaration */
body .kn { color: #ff0000; font-weight: bold } /* Keyword.Namespace */
body .kp { color: #ff0000; font-weight: bold } /* Keyword.Pseudo */
body .kr { color: #ff0000; font-weight: bold } /* Keyword.Reserved */
body .kt { color: #ff0000; font-weight: bold } /* Keyword.Type */
body .s { color: #111111 } /* Literal.String */
body .na { color: #008000 } /* Name.Attribute */
body .nb { color: #008000 } /* Name.Builtin */
body .nc { color: #008000 } /* Name.Class */
body .no { color: #008000 } /* Name.Constant */
body .nd { color: #008000 } /* Name.Decorator */
body .ni { color: #008000 } /* Name.Entity */
body .ne { color: #008000 } /* Name.Exception */
body .nf { color: #008000 } /* Name.Function */
body .nl { color: #008000 } /* Name.Label */
body .nn { color: #008000 } /* Name.Namespace */
body .nx { color: #008000 } /* Name.Other */
body .py { color: #008000 } /* Name.Property */
body .nt { color: #008000 } /* Name.Tag */
body .nv { color: #008000 } /* Name.Variable */
body .sb { color: #111111 } /* Literal.String.Backtick */
body .sc { color: #111111 } /* Literal.String.Char */
body .sd { color: #111111 } /* Literal.String.Doc */
body .s2 { color: #111111 } /* Literal.String.Double */
body .se { color: #111111 } /* Literal.String.Escape */
body .sh { color: #111111 } /* Literal.String.Heredoc */
body .si { color: #111111 } /* Literal.String.Interpol */
body .sx { color: #111111 } /* Literal.String.Other */
body .sr { color: #111111 } /* Literal.String.Regex */
body .s1 { color: #111111 } /* Literal.String.Single */
body .ss { color: #111111 } /* Literal.String.Symbol */
body .bp { color: #008000 } /* Name.Builtin.Pseudo */
body .vc { color: #008000 } /* Name.Variable.Class */
body .vg { color: #008000 } /* Name.Variable.Global */
body .vi { color: #008000 } /* Name.Variable.Instance */

  </style>
</head>
<body>
<h2></h2>

<div class="highlight"><pre><span></span><span class="cm">NOTE: Writing HTML5 Body file: STDOUT</span><br></pre></div>
</body>
</html>




The following histogram shows the distributions of the bootstrapped `MATH` beta estimate parameter for the 100000 samples follows a Normal distribution:


```sas
proc univariate data=tdata noprint;
    histogram Estimate / kernel (noprint) normal (noprint);
run;
```




<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta charset="utf-8"/>
<meta content="SAS 9.4" name="generator"/>
<title>SAS Output</title>
<style>
/*<![CDATA[*/
.body.c section > table, .body.c section > pre, .body.c div > table,
.body.c div > pre, .body.c article > table, .body.c article > pre,
.body.j section > table, .body.j section > pre, .body.j div > table,
.body.j div > pre, .body.j article > table, .body.j article > pre,
.body.c p.note, .body.c p.warning, .body.c p.error, .body.c p.fatal,
.body.j p.note, .body.j p.warning, .body.j p.error, .body.j p.fatal,
.body.c > table.layoutcontainer, .body.j > table.layoutcontainer { margin-left: auto; margin-right: auto }
.layoutregion.l table, .layoutregion.l pre, .layoutregion.l p.note,
.layoutregion.l p.warning, .layoutregion.l p.error, .layoutregion.l p.fatal { margin-left: 0 }
.layoutregion.c table, .layoutregion.c pre, .layoutregion.c p.note,
.layoutregion.c p.warning, .layoutregion.c p.error, .layoutregion.c p.fatal { margin-left: auto; margin-right: auto }
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r p.warning, .layoutregion.r p.error, .layoutregion.r p.fatal { margin-right: 0 }
article, aside, details, figcaption, figure, footer, header, hgroup, nav, section { display: block }
html{ font-size: 100% }
.body { margin: 1em; font-size: 13px; line-height: 1.231 }
sup { position: relative; vertical-align: baseline; bottom: 0.25em; font-size: 0.8em }
sub { position: relative; vertical-align: baseline; top: 0.25em; font-size: 0.8em }
ul, ol { margin: 1em 0; padding: 0 0 0 40px }
dd { margin: 0 0 0 40px }
nav ul, nav ol { list-style: none; list-style-image: none; margin: 0; padding: 0 }
img { border: 0; vertical-align: middle }
svg:not(:root) { overflow: hidden }
figure { margin: 0 }
table { border-collapse: collapse; border-spacing: 0 }
.layoutcontainer { border-collapse: separate; border-spacing: 0 }
p { margin-top: 0; text-align: left }
span { text-align: left }
table { margin-bottom: 1em }
td, th { text-align: left; padding: 3px 6px; vertical-align: top }
td[class$="fixed"], th[class$="fixed"] { white-space: pre }
section, article { padding-top: 1px; padding-bottom: 8px }
hr.pagebreak { height: 0px; border: 0; border-bottom: 1px solid #c0c0c0; margin: 1em 0 }
.stacked-value { text-align: left; display: block }
.stacked-cell > .stacked-value, td.data > td.data, th.data > td.data, th.data > th.data, td.data > th.data, th.header > th.header { border: 0 }
.stacked-cell > div.data { border-width: 0 }
.systitleandfootercontainer { white-space: nowrap; margin-bottom: 1em }
.systitleandfootercontainer > p { margin: 0 }
.systitleandfootercontainer > p > span { display: inline-block; width: 100%; white-space: normal }
.batch { display: table }
.toc { display: none }
.proc_note_group, .proc_title_group { margin-bottom: 1em }
p.proctitle { margin: 0 }
p.note, p.warning, p.error, p.fatal { display: table }
.notebanner, .warnbanner, .errorbanner, .fatalbanner,
.notecontent, .warncontent, .errorcontent, .fatalcontent { display: table-cell; padding: 0.5em }
.notebanner, .warnbanner, .errorbanner, .fatalbanner { padding-right: 0 }
.body > div > ol li { text-align: left }
.c { text-align: center }
.r { text-align: right }
.l { text-align: left }
.j { text-align: justify }
.d { text-align: right }
.b { vertical-align: bottom }
.m { vertical-align: middle }
.t { vertical-align: top }
.aftercaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    padding-top: 4pt;
}
.batch > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.batch > tbody, .batch > thead, .batch > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.batch { border: hidden; }
.batch {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: 'SAS Monospace', 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    padding: 7px;
    }
.beforecaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.body {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    margin-left: 8px;
    margin-right: 8px;
}
.bodydate {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: right;
    vertical-align: top;
    width: 100%;
}
.bycontentfolder {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.byline {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.bylinecontainer > col, .bylinecontainer > colgroup > col, .bylinecontainer > colgroup, .bylinecontainer > tr, .bylinecontainer > * > tr, .bylinecontainer > thead, .bylinecontainer > tbody, .bylinecontainer > tfoot { border: none; }
.bylinecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.caption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.cell, .container {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.contentfolder, .contentitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.contentproclabel, .contentprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.contents {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.contentsdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.contenttitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.continued {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    width: 100%;
}
.data, .dataemphasis {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.dataemphasisfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.dataempty {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datafixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datastrong {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.datastrongfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.date {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.document {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.errorcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.extendedpage {
    background-color: #fafbfe;
    border-style: solid;
    border-width: 1pt;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
    text-align: center;
}
.fatalbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.fatalcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.fatalcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.folderaction {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.footer {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footeremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footeremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.footerstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.frame {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.graph > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.graph > tbody, .graph > thead, .graph > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.graph { border: hidden; }
.graph {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.header {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headeremphasis {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headeremphasisfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.headersandfooters {
    background-color: #edf2f9;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrong {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrongfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.index {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.indexaction, .indexitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.indexprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.indextitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.layoutcontainer, .layoutregion {
    border-width: 0;
    border-spacing: 30px;
}
.linecontent {
    background-color: #fafbfe;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.list {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.list10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.list2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.list3, .list4, .list5, .list6, .list7, .list8, .list9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.listitem10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.listitem3, .listitem4, .listitem5, .listitem6, .listitem7, .listitem8, .listitem9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.note {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notebanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.notecontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notecontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.output > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.output > tbody, .output > thead, .output > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.output { border: hidden; }
.output {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.pageno {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    text-align: right;
    vertical-align: top;
}
.pages {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.pagesdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.pagesitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.pagesproclabel, .pagesprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.pagestitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.paragraph {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.parskip > col, .parskip > colgroup > col, .parskip > colgroup, .parskip > tr, .parskip > * > tr, .parskip > thead, .parskip > tbody, .parskip > tfoot { border: none; }
.parskip {
    border: none;
    border-spacing: 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    }
.prepage {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: left;
}
.proctitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.proctitlefixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooter {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooteremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooteremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooterempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowfooterstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheader {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderemphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderemphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowheaderstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.systemfooter, .systemfooter10, .systemfooter2, .systemfooter3, .systemfooter4, .systemfooter5, .systemfooter6, .systemfooter7, .systemfooter8, .systemfooter9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.systemtitle, .systemtitle10, .systemtitle2, .systemtitle3, .systemtitle4, .systemtitle5, .systemtitle6, .systemtitle7, .systemtitle8, .systemtitle9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size: small;
    font-style: normal;
    font-weight: bold;
}
.systitleandfootercontainer > col, .systitleandfootercontainer > colgroup > col, .systitleandfootercontainer > colgroup, .systitleandfootercontainer > tr, .systitleandfootercontainer > * > tr, .systitleandfootercontainer > thead, .systitleandfootercontainer > tbody, .systitleandfootercontainer > tfoot { border: none; }
.systitleandfootercontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.table > col, .table > colgroup > col {
    border-left: 1px solid #c1c1c1;
    border-right: 0 solid #c1c1c1;
}
.table > tr, .table > * > tr {
    border-top: 1px solid #c1c1c1;
    border-bottom: 0 solid #c1c1c1;
}
.table { border: hidden; }
.table {
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 1px 0 0 1px;
    border-collapse: collapse;
    border-spacing: 0;
    }
.titleandnotecontainer > col, .titleandnotecontainer > colgroup > col, .titleandnotecontainer > colgroup, .titleandnotecontainer > tr, .titleandnotecontainer > * > tr, .titleandnotecontainer > thead, .titleandnotecontainer > tbody, .titleandnotecontainer > tfoot { border: none; }
.titleandnotecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.titlesandfooters {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.usertext {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warnbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.warncontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warncontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
/*]]>*/
</style>
</head>
<body class="l body">
<h1 class="body toc">SAS Output</h1>
<section data-name="Univariate" data-sec-type="proc">
<div id="IDX" class="systitleandfootercontainer" style="border-spacing: 1px">
<p><span class="c systemtitle">The SAS System</span> </p>
</div>
<div class="proc_title_group">
<p class="c proctitle">The UNIVARIATE Procedure</p>
</div>
<h1 class="contentprocname toc">The UNIVARIATE Procedure</h1>
<section>
<h1 class="contentfolder toc">Estimate</h1>
<section>
<h1 class="contentfolder toc">Histogram 1</h1>
<article>
<h1 class="contentitem toc">Panel 1</h1>
<div class="c">
<img style="height: 480px; width: 640px" alt="Histogram for Estimate" src="data:image/png;base64,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I/qZo8+m6SNohmHMZnNCQkJCQsK+ffvYQr1e726vxWLJyspSKpWpqansw0zBQt8iEtSzj9dEnkagIkqaI/+TWUBy9CGopGmy7Wg0mvLy8rGxsYCkOdWIwZmuzLPzJwjTlRsxOOPo0k4QxtFrOgHs1ZaWFr1er9Ppamtrg5CjYESpczQYDP39/V4PkTSi/zn6xmaz4c8RAgAABBv+HCEAAIA8sAADAADIAAswAACADLAAAwAAyAALMAAAgAywAAMAAMgACzAAAIAMsAADAADIAAswAACADLAAAwAAyAALMAAAgAywAAMAAMgACzDATKIQ4rk+u7148WI/Q/tzOAC4CJP7BABgahihv+Qoxh9//BHYMwEAf+AOGCBEnDp1auHChWq1euHChUePHqX+u2dl75LZW1iFQlFXVxcTExMTE9PR0bF///45c+bExMQcO3aMVDh27NiSJUsiIiISExNPnTrFb+rBgwf5+flqtfq999578OCBHOkCzHhYgAFCxJo1aw4ePPjkyROz2dzR0UH9d69M/vq3S+Xr16///fffBw8eLC0tPX/+vN1u/+mnn7Zu3Ur2tre3f/XVV0+fPt27d++mTZv4TX355ZcFBQVPnjyprKz8+uuvg5kmQMhQ2Gy2RYsWyX0aACAK/4tYdnFdsGDBhg0bSktL582bx63PVmC3BQspigoLC5uYmHBpny3k1oyJiblx40ZERMTk5GRcXNzt27cDmCPA82BwcBB3wAAzDPMstvz06dP3799fvnz5kiVLzp4960PLNE2TjStXrqxZs2bevHlarZYt5LLb7SqVSqFQKJXKe/fu+ZYIwHMOCzBAiIiLi/vuu++uXLnyzTffmEwmf5pauXLlm2++efny5X/++UewQnR0NE3T5BMA/6YZAMTAAgwQItavX09emIqKipqcnCSFKSkpPjwfvnPnTkFBAUVRX3zxBVvIbeqDDz5oaGigKGpwcHDDhg3+nzzAcwgLMMAM4+7fAefn53/66adhYWHr168/dOgQKezs7Fy+fPnbb789pRBmszk5OTklJSUpKUmn05H3nLlN7d69+9SpU2FhYQUFBUajMaD5ATwv8BIWAABAsOElLAAAAHlgAQYAAJABFmAAAAAZYAEGAACQARZgAAAAGWABBgAAkAEWYAAAABlgAQYAAJABFmAAAAAZYAEGAACQARZgAAAAGWAB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"/>
</div>
</article>
</section>
</section>
</section>
</body>
</html>




### Method 1: Normal Distribution Confidence Interval

We will first create a confidence interval using the normal distribution theory. This
assumes that the Beta values follow a t distribution, so we can generate a 95%
confidence interval by about the mean of the Betea values based on quantiles from
a t-distribution with 99000 degrees of freedom.  We find the critical t values for our
confidence interval and multiply these by the standard deviation of the Beta values
that arose in our 100000 replications.  Our confidence interval using this method is
symmetric about the Beta value we saw in our original regression.  We can see that
the 95% confidence interval using this method is (0.286893, 0.579775).  We have also
calculated the bias in our original value of Beta as the difference between that
value and the mean of the 100000 Beatas in our bootstrap sample.


```sas
%let alphalev = .05;
ods listing;
proc sql;
  select  &est_bar as estimate,
          mean(estimate) - &est_bar as bias,
		  std(estimate) as std_err,
          &est_bar - tinv(1-&alphalev/2, &rep-1)*std(estimate) as lb,
          &est_bar + tinv(1-&alphalev/2, &rep-1)*std(estimate) as hb
  from t;
quit;
```




<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta charset="utf-8"/>
<meta content="SAS 9.4" name="generator"/>
<title>SAS Output</title>
<style>
/*<![CDATA[*/
.body.c section > table, .body.c section > pre, .body.c div > table,
.body.c div > pre, .body.c article > table, .body.c article > pre,
.body.j section > table, .body.j section > pre, .body.j div > table,
.body.j div > pre, .body.j article > table, .body.j article > pre,
.body.c p.note, .body.c p.warning, .body.c p.error, .body.c p.fatal,
.body.j p.note, .body.j p.warning, .body.j p.error, .body.j p.fatal,
.body.c > table.layoutcontainer, .body.j > table.layoutcontainer { margin-left: auto; margin-right: auto }
.layoutregion.l table, .layoutregion.l pre, .layoutregion.l p.note,
.layoutregion.l p.warning, .layoutregion.l p.error, .layoutregion.l p.fatal { margin-left: 0 }
.layoutregion.c table, .layoutregion.c pre, .layoutregion.c p.note,
.layoutregion.c p.warning, .layoutregion.c p.error, .layoutregion.c p.fatal { margin-left: auto; margin-right: auto }
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r p.warning, .layoutregion.r p.error, .layoutregion.r p.fatal { margin-right: 0 }
article, aside, details, figcaption, figure, footer, header, hgroup, nav, section { display: block }
html{ font-size: 100% }
.body { margin: 1em; font-size: 13px; line-height: 1.231 }
sup { position: relative; vertical-align: baseline; bottom: 0.25em; font-size: 0.8em }
sub { position: relative; vertical-align: baseline; top: 0.25em; font-size: 0.8em }
ul, ol { margin: 1em 0; padding: 0 0 0 40px }
dd { margin: 0 0 0 40px }
nav ul, nav ol { list-style: none; list-style-image: none; margin: 0; padding: 0 }
img { border: 0; vertical-align: middle }
svg:not(:root) { overflow: hidden }
figure { margin: 0 }
table { border-collapse: collapse; border-spacing: 0 }
.layoutcontainer { border-collapse: separate; border-spacing: 0 }
p { margin-top: 0; text-align: left }
span { text-align: left }
table { margin-bottom: 1em }
td, th { text-align: left; padding: 3px 6px; vertical-align: top }
td[class$="fixed"], th[class$="fixed"] { white-space: pre }
section, article { padding-top: 1px; padding-bottom: 8px }
hr.pagebreak { height: 0px; border: 0; border-bottom: 1px solid #c0c0c0; margin: 1em 0 }
.stacked-value { text-align: left; display: block }
.stacked-cell > .stacked-value, td.data > td.data, th.data > td.data, th.data > th.data, td.data > th.data, th.header > th.header { border: 0 }
.stacked-cell > div.data { border-width: 0 }
.systitleandfootercontainer { white-space: nowrap; margin-bottom: 1em }
.systitleandfootercontainer > p { margin: 0 }
.systitleandfootercontainer > p > span { display: inline-block; width: 100%; white-space: normal }
.batch { display: table }
.toc { display: none }
.proc_note_group, .proc_title_group { margin-bottom: 1em }
p.proctitle { margin: 0 }
p.note, p.warning, p.error, p.fatal { display: table }
.notebanner, .warnbanner, .errorbanner, .fatalbanner,
.notecontent, .warncontent, .errorcontent, .fatalcontent { display: table-cell; padding: 0.5em }
.notebanner, .warnbanner, .errorbanner, .fatalbanner { padding-right: 0 }
.body > div > ol li { text-align: left }
.c { text-align: center }
.r { text-align: right }
.l { text-align: left }
.j { text-align: justify }
.d { text-align: right }
.b { vertical-align: bottom }
.m { vertical-align: middle }
.t { vertical-align: top }
.aftercaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    padding-top: 4pt;
}
.batch > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.batch > tbody, .batch > thead, .batch > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.batch { border: hidden; }
.batch {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: 'SAS Monospace', 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    padding: 7px;
    }
.beforecaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.body {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    margin-left: 8px;
    margin-right: 8px;
}
.bodydate {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: right;
    vertical-align: top;
    width: 100%;
}
.bycontentfolder {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.byline {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.bylinecontainer > col, .bylinecontainer > colgroup > col, .bylinecontainer > colgroup, .bylinecontainer > tr, .bylinecontainer > * > tr, .bylinecontainer > thead, .bylinecontainer > tbody, .bylinecontainer > tfoot { border: none; }
.bylinecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.caption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.cell, .container {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.contentfolder, .contentitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.contentproclabel, .contentprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.contents {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.contentsdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.contenttitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.continued {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    width: 100%;
}
.data, .dataemphasis {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.dataemphasisfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.dataempty {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datafixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datastrong {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.datastrongfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.date {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.document {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.errorcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.extendedpage {
    background-color: #fafbfe;
    border-style: solid;
    border-width: 1pt;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
    text-align: center;
}
.fatalbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.fatalcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.fatalcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.folderaction {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.footer {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footeremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footeremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.footerstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.frame {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.graph > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.graph > tbody, .graph > thead, .graph > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.graph { border: hidden; }
.graph {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.header {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headeremphasis {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headeremphasisfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.headersandfooters {
    background-color: #edf2f9;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrong {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrongfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.index {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.indexaction, .indexitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.indexprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.indextitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.layoutcontainer, .layoutregion {
    border-width: 0;
    border-spacing: 30px;
}
.linecontent {
    background-color: #fafbfe;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.list {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.list10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.list2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.list3, .list4, .list5, .list6, .list7, .list8, .list9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.listitem10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.listitem3, .listitem4, .listitem5, .listitem6, .listitem7, .listitem8, .listitem9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.note {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notebanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.notecontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notecontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.output > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.output > tbody, .output > thead, .output > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.output { border: hidden; }
.output {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.pageno {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    text-align: right;
    vertical-align: top;
}
.pages {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.pagesdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.pagesitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.pagesproclabel, .pagesprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.pagestitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.paragraph {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.parskip > col, .parskip > colgroup > col, .parskip > colgroup, .parskip > tr, .parskip > * > tr, .parskip > thead, .parskip > tbody, .parskip > tfoot { border: none; }
.parskip {
    border: none;
    border-spacing: 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    }
.prepage {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: left;
}
.proctitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.proctitlefixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooter {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooteremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooteremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooterempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowfooterstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheader {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderemphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderemphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowheaderstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.systemfooter, .systemfooter10, .systemfooter2, .systemfooter3, .systemfooter4, .systemfooter5, .systemfooter6, .systemfooter7, .systemfooter8, .systemfooter9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.systemtitle, .systemtitle10, .systemtitle2, .systemtitle3, .systemtitle4, .systemtitle5, .systemtitle6, .systemtitle7, .systemtitle8, .systemtitle9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size: small;
    font-style: normal;
    font-weight: bold;
}
.systitleandfootercontainer > col, .systitleandfootercontainer > colgroup > col, .systitleandfootercontainer > colgroup, .systitleandfootercontainer > tr, .systitleandfootercontainer > * > tr, .systitleandfootercontainer > thead, .systitleandfootercontainer > tbody, .systitleandfootercontainer > tfoot { border: none; }
.systitleandfootercontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.table > col, .table > colgroup > col {
    border-left: 1px solid #c1c1c1;
    border-right: 0 solid #c1c1c1;
}
.table > tr, .table > * > tr {
    border-top: 1px solid #c1c1c1;
    border-bottom: 0 solid #c1c1c1;
}
.table { border: hidden; }
.table {
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 1px 0 0 1px;
    border-collapse: collapse;
    border-spacing: 0;
    }
.titleandnotecontainer > col, .titleandnotecontainer > colgroup > col, .titleandnotecontainer > colgroup, .titleandnotecontainer > tr, .titleandnotecontainer > * > tr, .titleandnotecontainer > thead, .titleandnotecontainer > tbody, .titleandnotecontainer > tfoot { border: none; }
.titleandnotecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.titlesandfooters {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.usertext {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warnbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.warncontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warncontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
                                                           The SAS System                         10:43 Saturday, March 18, 2017   1

                                          estimate      bias   std_err        lb        hb
                                          ------------------------------------------------
                                          0.433334  0.000637  0.074715  0.286893  0.579775/*]]>*/
</style>
</head>
<body class="l body">
<h1 class="body toc">SAS Output</h1>
<section data-name="SQL" data-sec-type="proc">
<div id="IDX" class="systitleandfootercontainer" style="border-spacing: 1px">
<p><span class="c systemtitle">The SAS System</span> </p>
</div>
<h1 class="contentprocname toc">The SQL Procedure</h1>
<article>
<h1 class="contentitem toc">Query Results</h1>
<table class="table" style="border-spacing: 0">
<colgroup><col/><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="r b header" scope="col">estimate</th>
<th class="r b header" scope="col">bias</th>
<th class="r b header" scope="col">std_err</th>
<th class="r b header" scope="col">lb</th>
<th class="r b header" scope="col">hb</th>
</tr>
</thead>
<tbody>
<tr>
<td class="r data">0.433334</td>
<td class="r data">0.000637</td>
<td class="r data">0.074715</td>
<td class="r data">0.286893</td>
<td class="r data">0.579775</td>
</tr>
</tbody>
</table>
</article>
</section>
</body>
</html>




### Method 2: Percentile Confidence Interval
Another way to generate a bootstrap 95% confidence interval from the sample of 100000
Betas for `MATH` values is to look at the 2.5th and 97.5th percentiles in this distribution.
This approach to the confidence interval has some advantages over the normal approximation
used above.  This interval is not symmetric about the original estimate of the Beta
and this method is unaffected by monotonic transformations on the estimated statistic.
The first advantage is relevant because our original estimate is subject to bias.
The second advantage is less relevant in this example than in an instance where the
estimate might be subject to a transformation.  The bootstrap estimates that form the
bounds of the interval can be transformed in the same way to create the bootstrap interval
of the transformed estimate.

We can easily generate a percentile confidence interval in SAS using `PROC UNIVARIATE` after
creating some macro variables for the percentiles of interest and using them in the output
statement. We can see that the confidence interval from this method is (0.28637, 0.57973).
Since we have put the information of interest into a new dataset, pmethod, we have omitted
the standard output from the proc univariate.


```sas
%let alphalev = .05;
%let a1 = %sysevalf(&alphalev/2*100);
%let a2 = %sysevalf((1 - &alphalev/2)*100);
* creating confidence interval, percentile method;
proc univariate data = t alpha = .05 noprint;
  var estimate;
  output out=pmethod mean = estimate_hat pctlpts=&a1 &a2 pctlpre = p pctlname = _lb _ub ;
run;

data t2;
  set pmethod;
  bias = estimate_hat - &est_bar;
  estimate = &est_bar;
run;
ods listing;
proc print data  = t2;
  var estimate bias p_lb p_ub;
run;
```




<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta charset="utf-8"/>
<meta content="SAS 9.4" name="generator"/>
<title>SAS Output</title>
<style>
/*<![CDATA[*/
.body.c section > table, .body.c section > pre, .body.c div > table,
.body.c div > pre, .body.c article > table, .body.c article > pre,
.body.j section > table, .body.j section > pre, .body.j div > table,
.body.j div > pre, .body.j article > table, .body.j article > pre,
.body.c p.note, .body.c p.warning, .body.c p.error, .body.c p.fatal,
.body.j p.note, .body.j p.warning, .body.j p.error, .body.j p.fatal,
.body.c > table.layoutcontainer, .body.j > table.layoutcontainer { margin-left: auto; margin-right: auto }
.layoutregion.l table, .layoutregion.l pre, .layoutregion.l p.note,
.layoutregion.l p.warning, .layoutregion.l p.error, .layoutregion.l p.fatal { margin-left: 0 }
.layoutregion.c table, .layoutregion.c pre, .layoutregion.c p.note,
.layoutregion.c p.warning, .layoutregion.c p.error, .layoutregion.c p.fatal { margin-left: auto; margin-right: auto }
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r p.warning, .layoutregion.r p.error, .layoutregion.r p.fatal { margin-right: 0 }
article, aside, details, figcaption, figure, footer, header, hgroup, nav, section { display: block }
html{ font-size: 100% }
.body { margin: 1em; font-size: 13px; line-height: 1.231 }
sup { position: relative; vertical-align: baseline; bottom: 0.25em; font-size: 0.8em }
sub { position: relative; vertical-align: baseline; top: 0.25em; font-size: 0.8em }
ul, ol { margin: 1em 0; padding: 0 0 0 40px }
dd { margin: 0 0 0 40px }
nav ul, nav ol { list-style: none; list-style-image: none; margin: 0; padding: 0 }
img { border: 0; vertical-align: middle }
svg:not(:root) { overflow: hidden }
figure { margin: 0 }
table { border-collapse: collapse; border-spacing: 0 }
.layoutcontainer { border-collapse: separate; border-spacing: 0 }
p { margin-top: 0; text-align: left }
span { text-align: left }
table { margin-bottom: 1em }
td, th { text-align: left; padding: 3px 6px; vertical-align: top }
td[class$="fixed"], th[class$="fixed"] { white-space: pre }
section, article { padding-top: 1px; padding-bottom: 8px }
hr.pagebreak { height: 0px; border: 0; border-bottom: 1px solid #c0c0c0; margin: 1em 0 }
.stacked-value { text-align: left; display: block }
.stacked-cell > .stacked-value, td.data > td.data, th.data > td.data, th.data > th.data, td.data > th.data, th.header > th.header { border: 0 }
.stacked-cell > div.data { border-width: 0 }
.systitleandfootercontainer { white-space: nowrap; margin-bottom: 1em }
.systitleandfootercontainer > p { margin: 0 }
.systitleandfootercontainer > p > span { display: inline-block; width: 100%; white-space: normal }
.batch { display: table }
.toc { display: none }
.proc_note_group, .proc_title_group { margin-bottom: 1em }
p.proctitle { margin: 0 }
p.note, p.warning, p.error, p.fatal { display: table }
.notebanner, .warnbanner, .errorbanner, .fatalbanner,
.notecontent, .warncontent, .errorcontent, .fatalcontent { display: table-cell; padding: 0.5em }
.notebanner, .warnbanner, .errorbanner, .fatalbanner { padding-right: 0 }
.body > div > ol li { text-align: left }
.c { text-align: center }
.r { text-align: right }
.l { text-align: left }
.j { text-align: justify }
.d { text-align: right }
.b { vertical-align: bottom }
.m { vertical-align: middle }
.t { vertical-align: top }
.aftercaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    padding-top: 4pt;
}
.batch > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.batch > tbody, .batch > thead, .batch > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.batch { border: hidden; }
.batch {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: 'SAS Monospace', 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    padding: 7px;
    }
.beforecaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.body {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    margin-left: 8px;
    margin-right: 8px;
}
.bodydate {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: right;
    vertical-align: top;
    width: 100%;
}
.bycontentfolder {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.byline {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.bylinecontainer > col, .bylinecontainer > colgroup > col, .bylinecontainer > colgroup, .bylinecontainer > tr, .bylinecontainer > * > tr, .bylinecontainer > thead, .bylinecontainer > tbody, .bylinecontainer > tfoot { border: none; }
.bylinecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.caption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.cell, .container {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.contentfolder, .contentitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.contentproclabel, .contentprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.contents {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.contentsdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.contenttitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.continued {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    width: 100%;
}
.data, .dataemphasis {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.dataemphasisfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.dataempty {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datafixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datastrong {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.datastrongfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.date {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.document {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.errorcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.extendedpage {
    background-color: #fafbfe;
    border-style: solid;
    border-width: 1pt;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
    text-align: center;
}
.fatalbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.fatalcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.fatalcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.folderaction {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.footer {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footeremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footeremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.footerstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.frame {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.graph > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.graph > tbody, .graph > thead, .graph > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.graph { border: hidden; }
.graph {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.header {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headeremphasis {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headeremphasisfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.headersandfooters {
    background-color: #edf2f9;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrong {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrongfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.index {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.indexaction, .indexitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.indexprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.indextitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.layoutcontainer, .layoutregion {
    border-width: 0;
    border-spacing: 30px;
}
.linecontent {
    background-color: #fafbfe;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.list {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.list10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.list2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.list3, .list4, .list5, .list6, .list7, .list8, .list9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.listitem10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.listitem3, .listitem4, .listitem5, .listitem6, .listitem7, .listitem8, .listitem9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.note {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notebanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.notecontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notecontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.output > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.output > tbody, .output > thead, .output > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.output { border: hidden; }
.output {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.pageno {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    text-align: right;
    vertical-align: top;
}
.pages {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.pagesdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.pagesitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.pagesproclabel, .pagesprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.pagestitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.paragraph {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.parskip > col, .parskip > colgroup > col, .parskip > colgroup, .parskip > tr, .parskip > * > tr, .parskip > thead, .parskip > tbody, .parskip > tfoot { border: none; }
.parskip {
    border: none;
    border-spacing: 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    }
.prepage {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: left;
}
.proctitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.proctitlefixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooter {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooteremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooteremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooterempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowfooterstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheader {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderemphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderemphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowheaderstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.systemfooter, .systemfooter10, .systemfooter2, .systemfooter3, .systemfooter4, .systemfooter5, .systemfooter6, .systemfooter7, .systemfooter8, .systemfooter9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.systemtitle, .systemtitle10, .systemtitle2, .systemtitle3, .systemtitle4, .systemtitle5, .systemtitle6, .systemtitle7, .systemtitle8, .systemtitle9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size: small;
    font-style: normal;
    font-weight: bold;
}
.systitleandfootercontainer > col, .systitleandfootercontainer > colgroup > col, .systitleandfootercontainer > colgroup, .systitleandfootercontainer > tr, .systitleandfootercontainer > * > tr, .systitleandfootercontainer > thead, .systitleandfootercontainer > tbody, .systitleandfootercontainer > tfoot { border: none; }
.systitleandfootercontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.table > col, .table > colgroup > col {
    border-left: 1px solid #c1c1c1;
    border-right: 0 solid #c1c1c1;
}
.table > tr, .table > * > tr {
    border-top: 1px solid #c1c1c1;
    border-bottom: 0 solid #c1c1c1;
}
.table { border: hidden; }
.table {
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 1px 0 0 1px;
    border-collapse: collapse;
    border-spacing: 0;
    }
.titleandnotecontainer > col, .titleandnotecontainer > colgroup > col, .titleandnotecontainer > colgroup, .titleandnotecontainer > tr, .titleandnotecontainer > * > tr, .titleandnotecontainer > thead, .titleandnotecontainer > tbody, .titleandnotecontainer > tfoot { border: none; }
.titleandnotecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.titlesandfooters {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.usertext {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warnbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.warncontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warncontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}


                                                           The SAS System                         10:43 Saturday, March 18, 2017   3

                                        Obs    estimate       bias         p_lb       p_ub

                                         1      0.43333    .000636566    0.28637    0.57973/*]]>*/
</style>
</head>
<body class="l body">
<h1 class="body toc">SAS Output</h1>
<section data-name="Print" data-sec-type="proc">
<div id="IDX" class="systitleandfootercontainer" style="border-spacing: 1px">
<p><span class="c systemtitle">The SAS System</span> </p>
</div>
<h1 class="contentprocname toc">The PRINT Procedure</h1>
<article>
<h1 class="contentitem toc">Data Set WORK.T2</h1>
<table class="table" style="border-spacing: 0">
<colgroup><col/></colgroup><colgroup><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="r header" scope="col">Obs</th>
<th class="r header" scope="col">estimate</th>
<th class="r header" scope="col">bias</th>
<th class="r header" scope="col">p_lb</th>
<th class="r header" scope="col">p_ub</th>
</tr>
</thead>
<tbody>
<tr>
<th class="r rowheader" scope="row">1</th>
<td class="r data">0.43333</td>
<td class="r data">.000636566</td>
<td class="r data">0.28637</td>
<td class="r data">0.57973</td>
</tr>
</tbody>
</table>
</article>
</section>
</body>
</html>




### Method 3: Bias-Corrected Confidence Interval

We can also correct for bias in calculating our confidence interval. We have calculated bias
in the previous method as the difference between the Beta we observed in our initial
regression and the mean of the 100000 Beta values from the bootstrap samples.  The Beta
estimate from the initial regression is assumed to be an unbiased estimate of the true parameter.
If we wish to correct for the bias in calculating our confidence interval, we can go through the
steps below.  These are described by Cameron and Trivedi in Microeconomics Using Stata.

We first calculate the proportion of the bootstrap Beta that are less than our original
value.  We will adjust the percentiles used to define our confidence interval based on how
this proportion differs from 0.5.  We then find the probit of this proportion (z0) and the
proportion associated with our alpha level (zalpha). Next, we calculate the two percentiles
that will be used to find our confidence interval, p1 and p2, from these values.  We then
calculate our interval with proc univariate.  From this method, our interval is (0.27954, 0.57413).


```sas
%let alphalev = .05;
%let alpha1 = %sysevalf(1 - &alphalev/2);
%put &alpha1;
proc sql;
  select sum(estimate<=&est_bar)/count(estimate) into :z0bar
  from t;
quit;

data _null_;
  z0 = probit(&z0bar);
  zalpha = probit(&alpha1);
  p1 = put(probnorm(2*z0 - zalpha)*100, 3.0);
  p2 = put(probnorm(2*z0 + zalpha)*100, 3.0);
  output;
  call symput('a1', p1);
  call symput('a2', p2);
run;
```




<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta charset="utf-8"/>
<meta content="SAS 9.4" name="generator"/>
<title>SAS Output</title>
<style>
/*<![CDATA[*/
.body.c section > table, .body.c section > pre, .body.c div > table,
.body.c div > pre, .body.c article > table, .body.c article > pre,
.body.j section > table, .body.j section > pre, .body.j div > table,
.body.j div > pre, .body.j article > table, .body.j article > pre,
.body.c p.note, .body.c p.warning, .body.c p.error, .body.c p.fatal,
.body.j p.note, .body.j p.warning, .body.j p.error, .body.j p.fatal,
.body.c > table.layoutcontainer, .body.j > table.layoutcontainer { margin-left: auto; margin-right: auto }
.layoutregion.l table, .layoutregion.l pre, .layoutregion.l p.note,
.layoutregion.l p.warning, .layoutregion.l p.error, .layoutregion.l p.fatal { margin-left: 0 }
.layoutregion.c table, .layoutregion.c pre, .layoutregion.c p.note,
.layoutregion.c p.warning, .layoutregion.c p.error, .layoutregion.c p.fatal { margin-left: auto; margin-right: auto }
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r p.warning, .layoutregion.r p.error, .layoutregion.r p.fatal { margin-right: 0 }
article, aside, details, figcaption, figure, footer, header, hgroup, nav, section { display: block }
html{ font-size: 100% }
.body { margin: 1em; font-size: 13px; line-height: 1.231 }
sup { position: relative; vertical-align: baseline; bottom: 0.25em; font-size: 0.8em }
sub { position: relative; vertical-align: baseline; top: 0.25em; font-size: 0.8em }
ul, ol { margin: 1em 0; padding: 0 0 0 40px }
dd { margin: 0 0 0 40px }
nav ul, nav ol { list-style: none; list-style-image: none; margin: 0; padding: 0 }
img { border: 0; vertical-align: middle }
svg:not(:root) { overflow: hidden }
figure { margin: 0 }
table { border-collapse: collapse; border-spacing: 0 }
.layoutcontainer { border-collapse: separate; border-spacing: 0 }
p { margin-top: 0; text-align: left }
span { text-align: left }
table { margin-bottom: 1em }
td, th { text-align: left; padding: 3px 6px; vertical-align: top }
td[class$="fixed"], th[class$="fixed"] { white-space: pre }
section, article { padding-top: 1px; padding-bottom: 8px }
hr.pagebreak { height: 0px; border: 0; border-bottom: 1px solid #c0c0c0; margin: 1em 0 }
.stacked-value { text-align: left; display: block }
.stacked-cell > .stacked-value, td.data > td.data, th.data > td.data, th.data > th.data, td.data > th.data, th.header > th.header { border: 0 }
.stacked-cell > div.data { border-width: 0 }
.systitleandfootercontainer { white-space: nowrap; margin-bottom: 1em }
.systitleandfootercontainer > p { margin: 0 }
.systitleandfootercontainer > p > span { display: inline-block; width: 100%; white-space: normal }
.batch { display: table }
.toc { display: none }
.proc_note_group, .proc_title_group { margin-bottom: 1em }
p.proctitle { margin: 0 }
p.note, p.warning, p.error, p.fatal { display: table }
.notebanner, .warnbanner, .errorbanner, .fatalbanner,
.notecontent, .warncontent, .errorcontent, .fatalcontent { display: table-cell; padding: 0.5em }
.notebanner, .warnbanner, .errorbanner, .fatalbanner { padding-right: 0 }
.body > div > ol li { text-align: left }
.c { text-align: center }
.r { text-align: right }
.l { text-align: left }
.j { text-align: justify }
.d { text-align: right }
.b { vertical-align: bottom }
.m { vertical-align: middle }
.t { vertical-align: top }
.aftercaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    padding-top: 4pt;
}
.batch > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.batch > tbody, .batch > thead, .batch > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.batch { border: hidden; }
.batch {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: 'SAS Monospace', 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    padding: 7px;
    }
.beforecaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.body {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    margin-left: 8px;
    margin-right: 8px;
}
.bodydate {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: right;
    vertical-align: top;
    width: 100%;
}
.bycontentfolder {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.byline {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.bylinecontainer > col, .bylinecontainer > colgroup > col, .bylinecontainer > colgroup, .bylinecontainer > tr, .bylinecontainer > * > tr, .bylinecontainer > thead, .bylinecontainer > tbody, .bylinecontainer > tfoot { border: none; }
.bylinecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.caption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.cell, .container {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.contentfolder, .contentitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.contentproclabel, .contentprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.contents {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.contentsdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.contenttitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.continued {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    width: 100%;
}
.data, .dataemphasis {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.dataemphasisfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.dataempty {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datafixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datastrong {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.datastrongfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.date {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.document {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.errorcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.extendedpage {
    background-color: #fafbfe;
    border-style: solid;
    border-width: 1pt;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
    text-align: center;
}
.fatalbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.fatalcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.fatalcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.folderaction {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.footer {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footeremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footeremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.footerstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.frame {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.graph > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.graph > tbody, .graph > thead, .graph > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.graph { border: hidden; }
.graph {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.header {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headeremphasis {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headeremphasisfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.headersandfooters {
    background-color: #edf2f9;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrong {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrongfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.index {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.indexaction, .indexitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.indexprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.indextitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.layoutcontainer, .layoutregion {
    border-width: 0;
    border-spacing: 30px;
}
.linecontent {
    background-color: #fafbfe;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.list {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.list10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.list2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.list3, .list4, .list5, .list6, .list7, .list8, .list9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.listitem10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.listitem3, .listitem4, .listitem5, .listitem6, .listitem7, .listitem8, .listitem9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.note {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notebanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.notecontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notecontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.output > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.output > tbody, .output > thead, .output > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.output { border: hidden; }
.output {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.pageno {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    text-align: right;
    vertical-align: top;
}
.pages {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.pagesdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.pagesitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.pagesproclabel, .pagesprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.pagestitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.paragraph {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.parskip > col, .parskip > colgroup > col, .parskip > colgroup, .parskip > tr, .parskip > * > tr, .parskip > thead, .parskip > tbody, .parskip > tfoot { border: none; }
.parskip {
    border: none;
    border-spacing: 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    }
.prepage {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: left;
}
.proctitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.proctitlefixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooter {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooteremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooteremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooterempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowfooterstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheader {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderemphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderemphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowheaderstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.systemfooter, .systemfooter10, .systemfooter2, .systemfooter3, .systemfooter4, .systemfooter5, .systemfooter6, .systemfooter7, .systemfooter8, .systemfooter9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.systemtitle, .systemtitle10, .systemtitle2, .systemtitle3, .systemtitle4, .systemtitle5, .systemtitle6, .systemtitle7, .systemtitle8, .systemtitle9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size: small;
    font-style: normal;
    font-weight: bold;
}
.systitleandfootercontainer > col, .systitleandfootercontainer > colgroup > col, .systitleandfootercontainer > colgroup, .systitleandfootercontainer > tr, .systitleandfootercontainer > * > tr, .systitleandfootercontainer > thead, .systitleandfootercontainer > tbody, .systitleandfootercontainer > tfoot { border: none; }
.systitleandfootercontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.table > col, .table > colgroup > col {
    border-left: 1px solid #c1c1c1;
    border-right: 0 solid #c1c1c1;
}
.table > tr, .table > * > tr {
    border-top: 1px solid #c1c1c1;
    border-bottom: 0 solid #c1c1c1;
}
.table { border: hidden; }
.table {
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 1px 0 0 1px;
    border-collapse: collapse;
    border-spacing: 0;
    }
.titleandnotecontainer > col, .titleandnotecontainer > colgroup > col, .titleandnotecontainer > colgroup, .titleandnotecontainer > tr, .titleandnotecontainer > * > tr, .titleandnotecontainer > thead, .titleandnotecontainer > tbody, .titleandnotecontainer > tfoot { border: none; }
.titleandnotecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.titlesandfooters {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.usertext {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warnbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.warncontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warncontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
/*]]>*/
</style>
</head>
<body class="l body">
<h1 class="body toc">SAS Output</h1>
<section data-name="SQL" data-sec-type="proc">
<div id="IDX" class="systitleandfootercontainer" style="border-spacing: 1px">
<p><span class="c systemtitle">The SAS System</span> </p>
</div>
<h1 class="contentprocname toc">The SQL Procedure</h1>
<article>
<h1 class="contentitem toc">Query Results</h1>
<table class="table" style="border-spacing: 0">
<colgroup><col/></colgroup>
<thead>
<tr>
<th class="r b header" scope="col"> &#160;</th>
</tr>
</thead>
<tbody>
<tr>
<td class="r data">0.4952</td>
</tr>
</tbody>
</table>
</article>
</section>
</body>
</html>




Creating confidence interval, bias-corrected method


```sas
proc univariate data = t alpha = .05 noprint;
  var estimate;
  output out=pmethod mean = estimate_hat pctlpts=&a1 &a2 pctlpre = p pctlname = _lb _ub ;
run;

data t2;
  set pmethod;
  bias = estimate_hat - &est_bar;
  estimate = &est_bar;
run;

ods listing;

proc print data  = t2;
  var estimate bias p_lb p_ub;
run;
```




<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
<meta charset="utf-8"/>
<meta content="SAS 9.4" name="generator"/>
<title>SAS Output</title>
<style>
/*<![CDATA[*/
.body.c section > table, .body.c section > pre, .body.c div > table,
.body.c div > pre, .body.c article > table, .body.c article > pre,
.body.j section > table, .body.j section > pre, .body.j div > table,
.body.j div > pre, .body.j article > table, .body.j article > pre,
.body.c p.note, .body.c p.warning, .body.c p.error, .body.c p.fatal,
.body.j p.note, .body.j p.warning, .body.j p.error, .body.j p.fatal,
.body.c > table.layoutcontainer, .body.j > table.layoutcontainer { margin-left: auto; margin-right: auto }
.layoutregion.l table, .layoutregion.l pre, .layoutregion.l p.note,
.layoutregion.l p.warning, .layoutregion.l p.error, .layoutregion.l p.fatal { margin-left: 0 }
.layoutregion.c table, .layoutregion.c pre, .layoutregion.c p.note,
.layoutregion.c p.warning, .layoutregion.c p.error, .layoutregion.c p.fatal { margin-left: auto; margin-right: auto }
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r table, .layoutregion.r pre, .layoutregion.r p.note,
.layoutregion.r p.warning, .layoutregion.r p.error, .layoutregion.r p.fatal { margin-right: 0 }
article, aside, details, figcaption, figure, footer, header, hgroup, nav, section { display: block }
html{ font-size: 100% }
.body { margin: 1em; font-size: 13px; line-height: 1.231 }
sup { position: relative; vertical-align: baseline; bottom: 0.25em; font-size: 0.8em }
sub { position: relative; vertical-align: baseline; top: 0.25em; font-size: 0.8em }
ul, ol { margin: 1em 0; padding: 0 0 0 40px }
dd { margin: 0 0 0 40px }
nav ul, nav ol { list-style: none; list-style-image: none; margin: 0; padding: 0 }
img { border: 0; vertical-align: middle }
svg:not(:root) { overflow: hidden }
figure { margin: 0 }
table { border-collapse: collapse; border-spacing: 0 }
.layoutcontainer { border-collapse: separate; border-spacing: 0 }
p { margin-top: 0; text-align: left }
span { text-align: left }
table { margin-bottom: 1em }
td, th { text-align: left; padding: 3px 6px; vertical-align: top }
td[class$="fixed"], th[class$="fixed"] { white-space: pre }
section, article { padding-top: 1px; padding-bottom: 8px }
hr.pagebreak { height: 0px; border: 0; border-bottom: 1px solid #c0c0c0; margin: 1em 0 }
.stacked-value { text-align: left; display: block }
.stacked-cell > .stacked-value, td.data > td.data, th.data > td.data, th.data > th.data, td.data > th.data, th.header > th.header { border: 0 }
.stacked-cell > div.data { border-width: 0 }
.systitleandfootercontainer { white-space: nowrap; margin-bottom: 1em }
.systitleandfootercontainer > p { margin: 0 }
.systitleandfootercontainer > p > span { display: inline-block; width: 100%; white-space: normal }
.batch { display: table }
.toc { display: none }
.proc_note_group, .proc_title_group { margin-bottom: 1em }
p.proctitle { margin: 0 }
p.note, p.warning, p.error, p.fatal { display: table }
.notebanner, .warnbanner, .errorbanner, .fatalbanner,
.notecontent, .warncontent, .errorcontent, .fatalcontent { display: table-cell; padding: 0.5em }
.notebanner, .warnbanner, .errorbanner, .fatalbanner { padding-right: 0 }
.body > div > ol li { text-align: left }
.c { text-align: center }
.r { text-align: right }
.l { text-align: left }
.j { text-align: justify }
.d { text-align: right }
.b { vertical-align: bottom }
.m { vertical-align: middle }
.t { vertical-align: top }
.aftercaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    padding-top: 4pt;
}
.batch > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.batch > tbody, .batch > thead, .batch > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.batch { border: hidden; }
.batch {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: 'SAS Monospace', 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    padding: 7px;
    }
.beforecaption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.body {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    margin-left: 8px;
    margin-right: 8px;
}
.bodydate {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: right;
    vertical-align: top;
    width: 100%;
}
.bycontentfolder {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.byline {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.bylinecontainer > col, .bylinecontainer > colgroup > col, .bylinecontainer > colgroup, .bylinecontainer > tr, .bylinecontainer > * > tr, .bylinecontainer > thead, .bylinecontainer > tbody, .bylinecontainer > tfoot { border: none; }
.bylinecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.caption {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.cell, .container {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.contentfolder, .contentitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.contentproclabel, .contentprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.contents {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.contentsdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.contenttitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.continued {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    width: 100%;
}
.data, .dataemphasis {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.dataemphasisfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.dataempty {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datafixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.datastrong {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.datastrongfixed {
    background-color: #ffffff;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.date {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.document {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.errorcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.errorcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.extendedpage {
    background-color: #fafbfe;
    border-style: solid;
    border-width: 1pt;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
    text-align: center;
}
.fatalbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.fatalcontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.fatalcontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.folderaction {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.footer {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footeremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footeremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.footerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.footerstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.footerstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.frame {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.graph > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.graph > tbody, .graph > thead, .graph > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.graph { border: hidden; }
.graph {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.header {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headeremphasis {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headeremphasisfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.headerempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.headersandfooters {
    background-color: #edf2f9;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrong {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.headerstrongfixed {
    background-color: #d8dbd3;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #000000;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.index {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.indexaction, .indexitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.indexprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.indextitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.layoutcontainer, .layoutregion {
    border-width: 0;
    border-spacing: 30px;
}
.linecontent {
    background-color: #fafbfe;
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.list {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.list10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.list2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.list3, .list4, .list5, .list6, .list7, .list8, .list9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: disc;
}
.listitem10 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.listitem2 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: circle;
}
.listitem3, .listitem4, .listitem5, .listitem6, .listitem7, .listitem8, .listitem9 {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: square;
}
.note {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notebanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.notecontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.notecontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.output > colgroup {
    border-left: 1px solid #c1c1c1;
    border-right: 1px solid #c1c1c1;
}
.output > tbody, .output > thead, .output > tfoot {
    border-top: 1px solid #c1c1c1;
    border-bottom: 1px solid #c1c1c1;
}
.output { border: hidden; }
.output {
    background-color: #fafbfe;
    border: 1px solid #c1c1c1;
    border-collapse: separate;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    }
.pageno {
    background-color: #fafbfe;
    border-spacing: 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    text-align: right;
    vertical-align: top;
}
.pages {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: decimal;
    margin-left: 8px;
    margin-right: 8px;
}
.pagesdate {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.pagesitem {
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    list-style-type: none;
    margin-left: 6pt;
}
.pagesproclabel, .pagesprocname {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.pagestitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: bold;
}
.paragraph {
    background-color: #fafbfe;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.parskip > col, .parskip > colgroup > col, .parskip > colgroup, .parskip > tr, .parskip > * > tr, .parskip > thead, .parskip > tbody, .parskip > tfoot { border: none; }
.parskip {
    border: none;
    border-spacing: 0;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
    }
.prepage {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    text-align: left;
}
.proctitle {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.proctitlefixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooter {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooteremphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooteremphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowfooterempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowfooterstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowfooterstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheader {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderemphasis {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderemphasisfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: italic;
    font-weight: normal;
}
.rowheaderempty {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.rowheaderstrong {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.rowheaderstrongfixed {
    background-color: #edf2f9;
    border-color: #b0b7bb;
    border-style: solid;
    border-width: 0 1px 1px 0;
    color: #112277;
    font-family: 'Courier New', Courier, monospace;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.systemfooter, .systemfooter10, .systemfooter2, .systemfooter3, .systemfooter4, .systemfooter5, .systemfooter6, .systemfooter7, .systemfooter8, .systemfooter9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.systemtitle, .systemtitle10, .systemtitle2, .systemtitle3, .systemtitle4, .systemtitle5, .systemtitle6, .systemtitle7, .systemtitle8, .systemtitle9 {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size: small;
    font-style: normal;
    font-weight: bold;
}
.systitleandfootercontainer > col, .systitleandfootercontainer > colgroup > col, .systitleandfootercontainer > colgroup, .systitleandfootercontainer > tr, .systitleandfootercontainer > * > tr, .systitleandfootercontainer > thead, .systitleandfootercontainer > tbody, .systitleandfootercontainer > tfoot { border: none; }
.systitleandfootercontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.table > col, .table > colgroup > col {
    border-left: 1px solid #c1c1c1;
    border-right: 0 solid #c1c1c1;
}
.table > tr, .table > * > tr {
    border-top: 1px solid #c1c1c1;
    border-bottom: 0 solid #c1c1c1;
}
.table { border: hidden; }
.table {
    border-color: #c1c1c1;
    border-style: solid;
    border-width: 1px 0 0 1px;
    border-collapse: collapse;
    border-spacing: 0;
    }
.titleandnotecontainer > col, .titleandnotecontainer > colgroup > col, .titleandnotecontainer > colgroup, .titleandnotecontainer > tr, .titleandnotecontainer > * > tr, .titleandnotecontainer > thead, .titleandnotecontainer > tbody, .titleandnotecontainer > tfoot { border: none; }
.titleandnotecontainer {
    background-color: #fafbfe;
    border: none;
    border-spacing: 1px;
    color: #000000;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
    width: 100%;
}
.titlesandfooters {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.usertext {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warnbanner {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: bold;
}
.warncontent {
    background-color: #fafbfe;
    color: #112277;
    font-family: Arial, 'Albany AMT', Helvetica, Helv;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}
.warncontentfixed {
    background-color: #fafbfe;
    color: #112277;
    font-family: 'Courier New', Courier;
    font-size:  normal;
    font-style: normal;
    font-weight: normal;
}


                                                           The SAS System                         10:43 Saturday, March 18, 2017   5

                                        Obs    estimate       bias         p_lb       p_ub

                                         1      0.43333    .000636566    0.27954    0.57413/*]]>*/
</style>
</head>
<body class="l body">
<h1 class="body toc">SAS Output</h1>
<section data-name="Print" data-sec-type="proc">
<div id="IDX" class="systitleandfootercontainer" style="border-spacing: 1px">
<p><span class="c systemtitle">The SAS System</span> </p>
</div>
<h1 class="contentprocname toc">The PRINT Procedure</h1>
<article>
<h1 class="contentitem toc">Data Set WORK.T2</h1>
<table class="table" style="border-spacing: 0">
<colgroup><col/></colgroup><colgroup><col/><col/><col/><col/></colgroup>
<thead>
<tr>
<th class="r header" scope="col">Obs</th>
<th class="r header" scope="col">estimate</th>
<th class="r header" scope="col">bias</th>
<th class="r header" scope="col">p_lb</th>
<th class="r header" scope="col">p_ub</th>
</tr>
</thead>
<tbody>
<tr>
<th class="r rowheader" scope="row">1</th>
<td class="r data">0.43333</td>
<td class="r data">.000636566</td>
<td class="r data">0.27954</td>
<td class="r data">0.57413</td>
</tr>
</tbody>
</table>
</article>
</section>
</body>
</html>




#### Conclusion

Because the bootstrapping confident interval doesn’t contains the zero and is very similar to the confident interval of the `GENMOD` we conclude that the estimated slope of 0.4333 for the `MATH` explanatory variable is statistically significant and that our assumption in the GLM are correct.

The SAS code can be useful also to bootstrap the standard errors. Another good example consists in check the FEMALE variable that has a p-value close to 0.05.

#### Reference: 

The code is based in the article titled ["How can I bootstrap estimates in SAS?"](http://www.ats.ucla.edu/stat/sas/faq/bootstrap.htm) where I changed the R-squared statistic by the slope of the `MATH` explanatory variable. There are also some other slightly differences.
