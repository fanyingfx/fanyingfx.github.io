---
title: Using Mecab with Unidic
feed: show
date: 31-212-2023
---
According [https://www.dampfkraft.com/nlp/japanese-tokenizer-dictionaries.htmlUniDic](https://www.dampfkraft.com/nlp/japanese-tokenizer-dictionaries.htmlUniDic) , using Uidic may have a good parsed result than the dictionary which the MeCab default dictionary IPADic.

The Uindic dictionary can download from [https://clrd.ninjal.ac.jp/unidic/](https://clrd.ninjal.ac.jp/unidic/)

For modern Japanese you may need first two dict, one is for written Japanese, another is for Spoken Japanese, you can choose one of it according the reading material.
![[Pasted image 20231231175703.png]]

![[Pasted image 20231231175718.png]]
Then Download the latest version Unidic, I think the light version is for Lute is enough and you can also download the full version which needs more space.

![[Pasted image 20231231175748.png]]
unidic-cwj-xxx.zip is for written Japanese

unidic-csj-xxx.zalled directory, ususally is `C:\Program Files\MeCab`

then extract the unidic dictionary to the dic directory

![[Pasted image 20231231175819.png]]
then in the MeCab etc folder, change the `mecabrc` file

change the `dicdir` to the uindic dictionay which in the dic folder

![[Pasted image 20231231175829.png]]