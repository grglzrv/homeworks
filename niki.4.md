5. Write a bash script that will resize all .jpg images in one directory to width of 1024px and proportional height. Hint: user convert(from the ImageMagick package).
------------------------------------------------------------------------------------------------------------------


Скрипта е доста елементарен, почти няма нищо за обяснение. От текущата директория се изважда списък с файлове и се вземат всички
които са с разширение *.jpg. След което вече избраните файлове се завъртат в цикъл и за всеки от тях се изпълнява конверт,
което цели да оразмери всички картинки с височина 1024 и пропорционална широчина.

Скрипта е наименуван convert.sh:

	#!/bin/bash
	if [[ -n `ls | grep .jpg` ]]; then
		echo "This will resize all jpg files with fixed width - 1024"
		for i in *.jpg
		do 
 			convert $i -resize 1024x $i
			echo -e "Resizing $i ... \e[32mOk\e[0m"  
		done 
	else 
		echo -e "\e[31mThere is no images in this directory\e[0m"
	fi	

Записваме и го правим изпълним с chmod +x convert.sh.

Може да се копира в директорията /usr/local/bin/, но след като променим името му,
защото вече има такава команда и когато сме например в ~/Pictures/Snimki/ и желаем да оразмерим всички снимки
то извикваме скрипта и той ще подмени всички оригинални снимки с оразмерените.
