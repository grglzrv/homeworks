1. Login on grizzly and get the home folders of the usernames that have numbers in them.
------------------------------------------------------------------------------------------------------------------


За целта се логваме на гризли(който в момента е offline) и изпълняваме командата:

	ls /home/ -1 | grep '[0-9]'

Резлутата ще бъде всички домашни папки на потребители, които съдържат цифра в тях.


2. With a single regular expression get all services that have the string 'file' in their names from /etc/services and are runing only on UDP.
------------------------------------------------------------------------------------------------------------------


За да извлечем всички услуги, които съдържат file и работят на UDP изпълняваме:

	grep 'file.*udp' /etc/services

Резултата е от моята машина:

	afs3-fileserver 7000/udp	bbs



3. On grizzly, parse /var/spool/mail/root and give me the timestamps of every e-mail that has 'SECURITY information' in its subject.
------------------------------------------------------------------------------------------------------------------

За да извлечем от файла /var/spool/mail/root, всички дати на съобщения озаглавени 'SECURITY information' изпълняваме в терминал:

	grep -A3 'Security information' /var/spool/mail/root | grep date 



