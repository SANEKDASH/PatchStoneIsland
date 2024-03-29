# PatchStoneIsland
# Взлом
## 0.Введение
Этот Readme описывает то, как я выполнял задачу взлома программы, 
которая проверяет правильность введенного пароля. Заведомо было известно, что
в программе находится уязвимость, связанная с переполнением буфера ввода.

Программа была написана на языке Assembler.

Всего стояло две задачи:
1) "Получить доступ" (получить на экране сообщение о получение доступа)
    путем введения __неправильного__ пароля (без изменения кода программы) с 
    использованием уязвимости связанной с переполнением буфера ввода.
2) Изменить уод программы так, чтобы при вводе __неправильного__ пароля 
    на экране выводилось сообщение о том, что был введен __правильный__.
    
Далее описано то, как я выполнял эти задачи.
## 1. Анализ .com файла
Мною был получен файл CRACK.com, который мне, собственно, и нужно было взломать.

Чтобы получить базовое представление об программе, я загрузил его в дизассемблер Ida.

После процесса дизассемблирования я четко увидел 4 функции:
>Функции были написаны именно в такой очередности
1) Функция ввода в память __правильного__ хэш-значения
1) Функция ввода пароля и помещения его в буфер
2) Функция хэширования __введенного__ пароля
3) Функция помещающая __правильное__ хэш значение в регистр __DX__

И еще один немало важный элемент 
этой программы -- __сравнение__ хэш значений __правильного__ и __введенного__ пароля.
Мы обратим на него внимание позже.

> Я заметил, что в функции ввода нет проверки на 
> переполнение буфера ввода, а также то, что по счастливым 
> обстоятельствам кусок памяти, отвечающий за правильное хэш значение,
> лежит за буфером ввода.
> Как итог, я этим и воспользовался.

## 2.Взлом переполнением
Исходя из вещей, замеченых ранее (см. пункт "Анализ .com файла"), я выработал 
методику взлома этой программы с помощью переполнения.

### Методика взлома:
Программу всего придется запускать два раза.

Первый -- чтобы понять, каким значением хэша пароля затирать 
правильное хэш значение.
>(Он проводится в TurboDebugger, чтобы видеть, что происходит происходит с памятью)

Второй -- собственно взлом.
>(Можно проводить этот запуск где угодно)
#### Первый запуск:
1) Полностью заполняем буфер ввода пароля. 
Не забываем ввести терминальный символ. (В данной реализации это оказался '$')
> Я сделал это с помощью последовательности символов: '11111'.
> (Буфер состоял всего из 5 байт). 

2) После работы функции хэширования введенного пароля смотрим 
    хэш значение и запоминаем его 
> При вводе '11111' получил соответственно значение 072h
3) Далее ожидаем конца программы.
#### Второй запуск:
1) Т.к. мы уже знаем, что паролю '11111' соответствует хэш значение 072h, 
нам нужно затереть правильное значение хэша (которое лежит в памяти за паролем)
на наше (072h). 
Соответственно вводим следующую последовательность символов '1111172'.
Не забываем ввести терминальный символ. (В данной реализации это оказался '$')

2) Фактически, взлом уже выполнен, и мы можем видеть на экране
заветные слова "Access to info выдан (гыгыгы)".

Можем выпить бокал шампанского за победу над программой!
## 3.Взлом с помощью изменения кода
В данной части мы вспомним про еще один 
значимый элемент программы -- проверка хэш значений (см.: Анализ .com файла).

Напишем программу на языке Си, которая изменит код программы, а именно 
выставит вместо битов, являющихся битами команд cmp и jnz, биты команды
nop (090h).
>Так называемый 'кряк' 
Исчезновение этой проверки, можно сказать, превращает нашу данный для взлома файл 
в программу, выводящую строку о правильно введенном пароле.

После работы 'кряка' запустим CRACK.com и посмотрим, что случится.
Вводим пароль, например: '12345'.
Видим сообщение 'Access to info approved.'

Взлом выполнен!.



