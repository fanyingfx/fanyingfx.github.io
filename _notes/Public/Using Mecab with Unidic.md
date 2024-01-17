---
title: Using Mecab with Unidic
feed: show
date: 17-01-2024
---
According [An Overview of Japanese Tokenizer Dictionaries](https://www.dampfkraft.com/nlp/japanese-tokenizer-dictionaries.html) , using Uidic may have a good parsed result than the dictionary which the MeCab default dictionary IPADic.

The Uindic dictionary can download from [https://clrd.ninjal.ac.jp/unidic/](https://clrd.ninjal.ac.jp/unidic/)

For modern Japanese you may need first two dict, one is for written Japanese, another is for Spoken Japanese, you can choose one of it according the reading material.

![](/assets/img/Pasted%20image%2020231231175703.png)

![](/assets/img/Pasted%20image%2020231231175718.png)

Then Download the latest version Unidic, I think the light version is for Lute is enough.

![](/assets/img/Pasted%20image%2020231231175748.png)
unidic-cwj-xxx.zip is for written Japanese
unidic-csj-xxx.zip is for spoken Japanese

then extract the unidic-xxx-xxx.zip 

in the unidic folder  you can find the dicrc and change the content
add these lines for Lute to get reading

```
; yomi
node-format-yomi = %f[9]
unk-format-yomi = %m
eos-format-yomi  = \n
```
Then the full dicrc should be like this
```
dictionary-charset = utf8
config-charset = utf8
cost-factor = 700
max-grouping-size = 10
bos-feature = BOS/EOS,*,*,*,*,*,*,*,*,*,*,*,*,*
eval-size = 12
unk-eval-size = 4
;output-format-type=default

; yomi
node-format-yomi = %f[9]
unk-format-yomi = %m
eos-format-yomi  = \n

node-format-unidic = %m\t%f[9]\t%f[6]\t%f[7]\t%F-[0,1,2,3]\t%f[4]\t%f[5]\t%f[13]\n
unk-format-unidic  = %m\t%m\t%m\t%m\tUNK\t%f[4]\t%f[5]\t\n
;unk-format-unidic  = %m\t%m\t%m\t%m\t%F-[0,1,2,3]\t%f[4]\t%f[5]\t\n
eos-format-unidic  = EOS\n

node-format-chamame = \t%m\t%f[9]\t%f[6]\t%f[7]\t%F-[0,1,2,3]\t%f[4]\t%f[5]\t%f[15]\t%f[11]\n
unk-format-chamame  = \t%m\t\t\t%m\t未知語\t\t\t\t\n
;unk-format-chamame  = \t%m\t\t\t%m\t%F-[0,1,2,3]\t\t\t\t\n
eos-format-chamame  = 
bos-format-chamame  = B

```

For Windows:

find the unidic installed directory, ususally is `C:\Program Files\MeCab`

then extract the copy/move unidic dictionary to the dic directory


![](/assets/img/Pasted%20image%2020231231175819.png)
then in the MeCab etc folder, change the `mecabrc` file

change the `dicdir` to the uindic dictionay which in the dic folder(it also can be the fullpath of the unidic dictionary )

![](/assets/img/Pasted%20image%2020231231175829.png)

On Linux the mecabrc could be in /etc/mecabrc
just replace the `dicdir` to the unidic fullpath
such as: 
```
; using semicolon to comment the old dicdir line then add the new dicdir
;dicdir = /var/lib/mecab/dic/debian
dicdir = /home/fan/.local/share/Lute3/unidic-csj
```