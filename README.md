# Kanz-ul-Iman  english


A simple attempt to scrape english verison of "This English version of the Holy Quran is the translation of Kanz-ul-Iman the Urdu version of Imam Ahl-e-Sunnat Maulana Shah Ahmed Raza Khan" 
that has been scrapped from ahadees.com website.

There is no current version of a developer friendly english version of "kanzul iman" which can be used by developers of quran apps for example.

The idea is that we can get the entire quran in a json form so that developers can  integrate it as they wish.


I take no responsibility of any additions or editorializing any titles. All data is received as is.

i will store the original scrapped data in txt files with surah:verse in 1.1 filename so that if anyone wants to use the data in any other form, they can do so. 

Please feel free to submit PRs and any corrections.


# the how

I've been trying to find Kanzul Iman in english version which is sadly not present in many of the quran apps so when i found ahadees.com, it looked nice but because of their arcane UX, the whole experience was something that left a lot to desire. 
Anyway, i figured out that the website actually shows each verse in a different page.

https://ahadees.com/quran/ayat.php?surah=1&ayat=1 is Surah 1 Aayat 1 or 1:1

and 

https://ahadees.com/quran/ayat.php?surah=10&ayat=15 is Surah 10 Aayat 15 or 10:15

Next, i figured out that their website has the english text inside a definite xpath

```
/html/body/table[2]/tbody/tr/td/table[2]/tbody/tr/td[2]/table[2]/tbody/tr[4]
```

That got me thinking. If i can extract all the verses, i could somehow wrangle them into something that a developer could use. Onto the trusty terminal.

First i tried ```curl``` but i could not manage to use variables ALONGWITH xpath constraint because i did not want the whole page and having to deal with stripping a single line out of 6228 html pages was simply unreasonable.

Then my DDG search took me to a tool called ```xidel``` and it was suprisingly easy. A single command could download and parse the whole thing.

```
xidel "https://www.ahadees.com/quran/ayat.php?surah=1&ayat=2"  -e  "/html/body/table[2]/tbody/tr/td/table[2]/tbody/tr/td[2]/table[2]/tbody/tr[4]" 
```

Yayyy..

Now i had to figure out how to encode the surah:verse data alongwith the actual verse data. I ended with naming the file with this surah.verse.txt

```
xidel "https://www.ahadees.com/quran/ayat.php?surah=1&ayat=2"  -e  "/html/body/table[2]/tbody/tr/td/table[2]/tbody/tr/td[2]/table[2]/tbody/tr[4]" > 1.2.txt
```

Now, The big problem of acutally putting together a script that understands that surah 1 contains only 7 verses and surah 2 contains 286 so i could not just DOS the website with 1.1....1.300 and then moving to 2.1....2.300 on and on. This would create thousands of empty files, unreasonable.

On to the next tool. Libreoffice Calc.

I am a free software guy so i extensively use (and report bugs) Libreoffice.

I ended up with taking the above command, splitting it into its constituents,

```  xidel "https://www.ahadees.com/quran/ayat.php?surah= ```  ```$1``` ```&ayat=``` ```$2``` ```-e  "/html/body/table[2]/tbody/tr/td/table[2]/tbody/tr/td[2]/table[2]/tbody/tr[4]" >``` ```$1``` ```.``` ```$2``` ```.txt```

Then it was easy. Just use "fill series" feature of Calc and first put $2, ie, verse of surah 1 which counted to 7, then i wrote the corresponding $1, then rince and repeat 114 times. 

Now once that was done, it was a simple job of using ``` concat``` formulae to recompile the commands. (Clac says the whole thing took around 6 hours)

I took these lines and put them in a bash file.


```
!/bin/sh
xidel "https://www.ahadees.com/quran/ayat.php?surah=1&ayat=1"  -e  "/html/body/table[2]/tbody/tr/td/table[2]/tbody/tr/td[2]/table[2]/tbody/tr[4]" > 1.1.txt
xidel "https://www.ahadees.com/quran/ayat.php?surah=1&ayat=2"  -e  "/html/body/table[2]/tbody/tr/td/table[2]/tbody/tr/td[2]/table[2]/tbody/tr[4]" > 1.2.txt
```

Then a simple 
``` bash download.sh```

got me all the files ( 1 second per line so around 6228 seconds to get all the verses)

Then the next step was to compile all the verses in a JSON so that it can be used elsewhere.

Enter https://github.com/kskarthik 

I am a noob when it comes to bash scripting and he helped me write a script

```
echo { >> ~/output.json;

cd files;

for file in $(ls -1 *);
do
echo \"$file\":\"$(cat $file)\", >> ~/output.json;
done
echo } >> ~/output.json;
```
So i store all the files inside the "files" directory and run this script which generates the output.json in my home directory. 

Mission Accomplished.

Well. Sorta.

There is a new kid on the block, https://github.com/AlfaazPlus/QuranApp and i found that their way of holding the verses in a very particular manner like https://github.com/AlfaazPlus/QuranApp/blob/master/inventory/translations/en/en_irving.json so the next step is to redo the generation script so that the output can be in this format and submit a PR.


Apparently, I had download 12 files less. 

Then spent half an hour trying to find the missing ones.

Simple.

ls > list.txt

then import the same in the calc_formula file in a new sheet, using csv import using a ```.``` as a delimiter. Then comparing that with the index sheet that was already present.
12/6236= .19% error. Not bad for a day's worth of work.

I typed these links manually and imported them.
