---
title: "CTFLearn - Milk's Best Friend"
date: 2025-04-09 00:01:00 +0000
author: scr4tcher
categories: [CTFLearn]
tags: [binwalk, strings, jpg, steganography]

---

## Description

In this challenge, we're provided with a `.jpg` image file that potentially contains a hidden flag. The goal is to analyze the file using basic steganography techniques to uncover the concealed content.

## Solution

# 1. Downloading the File
I start by downloading the image file `oreo.jpg` from the link provided by the challenge author.
    
# 2. Initial String Analysis 
 <br>I run the strings command to extract printable character sequences from the image file:
    


```console
└─$ strings oreo.jpg                        
JFIF
!1!')-../"
383-7(-.+
%--------------------------------------------------
!1AQ
h4Ph5
w5Hq
r[oK
l%*B
+SD!m
'0yUh
F#;u
OgmR
I +y
uQMv}
JeKmi(L
#Ham|
cj5!o
)SDl@!S
JR\y`
k<4Y
Y#t#%<
&&RTw>[
mWob
T\wz
Q@Ph
~[PS
l@'N
nG[;2
 H{Q
KJ%i*
P>,"
n.^@R
mM8Up
fI'8
)AH-
z$x@
wR?f
H>pD
%J ,
;yl     O
Rcu*
jH/,m
Fq'g
OQ$c
=g8?A
6#b} Q!
'       A:O
:?;`
BaA
(!gI 
'QRU
&<>C
Hxr[q.!
f4j)^
325r
w!I     P
q"2N
^xTt
$8      ;
ov1X(\
dHTc
OJpen
%=`sV
f=}j
#rDm
JJK:H
eJ$r
)H=Ry
(QZ
HT}F@
3#0`
S3M\J
+8Q(3
kI=$B
kMv$
%>(<
y+>D
5~bq>@`})
R0#|
'       W1
bS't
mH<>
\[un;3:ry
,I,x
Np+k
'WBT
- F7
vVw<
yO*P
G*n$
;{$`
>"v/
U_k~)
-l\1
(yr@
]ykqD
Q@Ph
PZT2`
e+IB
Q^{U
9%F?
J       Sc
uLfv
U2/6>.p
Md0z
`Qr,;
kt[u
v{=v
ij)J
Gt\iN:
`9UM1
/i>RD
jZmQ*IL(
        >}[
vp7'F
DO3 
\#ADm
Rar!
1\b.jpg
S*p)
TVTTZZV\XXZ``\VX_
p`ZXj
npvz}
;/)b
*]n18_
Qz}l
K'F6
dGOke
r%7+
$ywm
eRbB
roQg
[g:K
vgG:
j6:]
'kmG>
HH]u
~a>zdV
wiD4
7(qg
|               4
=Y.Hq   ~'
54""*|
T0;Q
?0q+Q
;B(W&|j
9Z_QjBZy?
%D^d&
uzEy
0)ay
#,&6
/98,
.M]B
hfu?
t\:o
s<JS
/#TS
.+[5Na
W[\2
I,,+
jm+:
"8JL
=b/R
}q"RN
N<Qre
dg?i
KsP8
3Osq
L0c>
% I@d
Zn }P
uG@m
=aLy
e6wx
]BHO
v:7I)U
k.Tr
o,ug~
Tpp6{j
cKL3H
vb(cR?
r6G&AZ0B
!7w-
GYJy
6cj=
{S!^
z$"}
RsZl
 soR
%EwU
A*"d.Ed
Tj$Uj
S-O.
'".r
I,Dq
v_,4y
Kq#i'
~p\>
DZb&
N|9D
\Z<~f
This is not the flag you are looking for.Q"t

```

This suggests that the actual flag might be hidden deeper, which is a common trick in CTF challenges.


```console
This is not the flag you are looking for.
```
# 3. Analyzing File Structure with binwalk

<br>To detect and extract any embedded files (such as hidden images or archives), I use binwalk with the extract option:
```console
binwalk -e oreo.jpg  
```

# 4. String Analysis of the extracked file

```console
$ strings b.jpg                           
JFIF
"1$%)+...
383-7(-.+
%----------------------+----------------------+---7
!1AQqa
\5n`]
xsLy
.y fk
vSk:M
DzuMb
_NZ@
]ETyn
Xg3H
nBC_
]95r
C^^[p
Q`';
q`7'
\\o*
.       &
04KZ
)Qc&
Q{k~
st&[
NW89
Lk$[
1Y79
a0\A
$;6g
%mG+$
DysM
2em7
6M>f
Ztn`$F
qUhTmjN
+67*
e6hi 
0d$j
-ko)'
CH;^u
&Du=
$t$Lv
1/i 
/1-6n
Gx#GA
M8n!
iT0?
kVI8
`.}v
gPl,c
bsDKw
O]=6V1
Rx|!
\l&>
!G=*
HSayi-9
#X3i
c>R2
 $+cmk1
u|h]a
tEp#
&Z      2`
ZMmG
a;}V
{2sRpo7%V
0=Q-C:
[e[!A
|5xk
+NgU
;HO+dD
D272}
`h      :
K`8m:-
Finally, flag{eat_more_oreos}
```


 I discover the hidden flag:

 **FLAG : flag{eat_more_oreos}**