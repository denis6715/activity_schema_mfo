# activity_schema_mfo

**chatgpt_query.txt** - запрос в chatgpt для формирования скрипта генерации необходимых данных

**mfo_data_generation.ipynb** - скрипт формирования данных

**data.zip** - архив со сгенерированными данными, который использовался для построения дашбордов

**dashboard.pdf** - итоговый дашборд в pdf формате

**ссылка на дашборд** - https://lookerstudio.google.com/reporting/0983d448-bbac-4a9f-b63c-23008adc6f95

**sample.csv** - пример из 50 кейсов схемы активности

### Схема активности

Ниже пример сгенерированных данных из csv:
![image](https://github.com/denis6715/activity_schema_mfo/assets/94977703/f089716c-1212-4ed7-96bc-e78a8ff022d6)

Ниже пример сгенрированного json с признаками (features):

![image](https://github.com/denis6715/activity_schema_mfo/assets/94977703/a42d57b5-93aa-4e80-9303-883d0818eba7)

### Задача
Генерация данных и построение схемы активности с целью анализа воронки, запросов и построения дашбордов.
В качестве ситуации рассматривается реклама займов МФО в интернете.

### Цель
Проанализировать, есть ли зависимость вероятности взятия займа клиентом от его поведения в процессе воронки, т.е. можем ли мы понять, насколько сильно ему нужен займ, на основании чего можно затем делать выводы, например, какую ставку ему предлагать и тратить ли ресурсы отделения на данного клиента.
Далее будут выдвигаться гипотезы, которые анализируются в дашбордах (в целях наглядной визуализации данные генерировались не случайным образом, а так, чтобы в основном гипотезы работали)

### Воронка
Клики - чтение подброной инфо - оставление заявки - посещение отделения - займ

![image](https://github.com/denis6715/activity_schema_mfo/assets/94977703/8463daa6-42ca-4cab-a0e1-6379ce73f51c)


### Описание процесса
Клиент переходит по рекламной ссылке и попадает на страницу с подробной инфо о займе. Далее клиент должен согласиться с условиями предоставления займа, он кликает и переходит на страницу ввода телефона, после ввода он попадает на следующую страницу ввода подтверждающего кода из смс. Затем идет форма регистрации, где клиент должен заполнить персональные данные (паспорт, email, ФИО), после чего клиент нажимает кнопку подать заявку (сабмит) с указанием желаемой суммы займа. Далее происходит скоринг его заявки и клиенту приходит смс: “Вам одобрен займ в размере до N1 рублей, со ставкой N2 процентов, приходите в наш офис в течение следующих пяти дней для оформления и выдачи займа”. Клиент приходит в офис, ему рассказывают о подробных условиях и оформляют займ при согласии клиента (при посещении отделения клиент может отказаться взять займ, например, из-за неадекватного отношения со стороны персонала, либо он узнает о дополнительных ограничениях - например надо предоставить залог или нужно подтвердить доходы).

### Решение задачи и анализ
Данные генерировались python скриптом, который в свою очередь генерировался chatgpt с минимальными ручными доработками. Генерировалась следующая схема активности:

![image](https://github.com/denis6715/activity_schema_mfo/assets/94977703/7e8df2c5-b174-46c3-812b-d319507eaaaf)

Рассматривались следующие активности:
1. "клик по рекламе"
2. "клик на переход с информации на ввод телефона"
3. "ввод телефона"
4. "ввод смс"
5. "ввод паспорта"
6. "ввод почты"
7. "ввод фио"
8. "сабмит"
9. "приход в отделение"
10. "взял займ"

Для таких активностей, где клиент вводит данные (3 - 7 п.), генерировались "поведенческие" признаки на основе заполнения анкеты для проверки гипотез (см. ниже). Рассматривались следующие признаки:
1. value - результат ввода
2. num_of_keypress - число нажатий клавиш при заполнении поля
3. num_of_backspace - число нажатий backspace при заполнении поля
   
Для активности "сабмит" генерировалась желаемая сумма займа (value).

Также далее формировались временные признаки с использованием поля timestamp.

Далее описаны выдвинутые гипотезы на основе поведения клиента при заполнении анкеты, а также приведены SQL-запросы для анализа и построения дашбордов.

**1. Гипотеза:** если клиент во время заполнения полей часто переходит туда-сюда, то он менеее вероятно возьмет займ

![image](https://github.com/denis6715/activity_schema_mfo/assets/94977703/87a5ce01-aea6-49dd-bad8-012689f55b7f)

![image](https://github.com/denis6715/activity_schema_mfo/assets/94977703/9e7eba77-2872-4b3c-ae44-1b8d9d35c94a)

**2. Гипотеза:** если клиент больше времени читал подробную инфо о займе (первая активность), то он в нем наиболее заинтересован 

![image](https://github.com/denis6715/activity_schema_mfo/assets/94977703/a3ed6dde-46ac-4e36-bc82-6757247f7ebe)

![image](https://github.com/denis6715/activity_schema_mfo/assets/94977703/43b9ab4b-db8a-4dbf-8ea2-0d8e10f76e79)

**3. Гипотеза:** если клиент дошел до сабмита и потратил больше времени на заполнение заявки, то он заполнял вдумчиво и более заинтересован в займе. Проведем анализ, сколько времени клиенты тратили на каждую из активностей в разрезе конверсии.

![image](https://github.com/denis6715/activity_schema_mfo/assets/94977703/d70e7d11-853c-4a5a-a693-fb8b77b23e49)

![image](https://github.com/denis6715/activity_schema_mfo/assets/94977703/dfed005e-b720-41b1-8278-c2db2594cef3)

**4. Гипотеза:** если клиент во время заполнения полей часто использует копипаст, то он менее вероятно возьмет займ, например, потому что он заполняет анкеты в другие МФО и копипастит данные в разные анкеты.

![image](https://github.com/denis6715/activity_schema_mfo/assets/94977703/b9acd792-4e67-4530-9f56-6fbb3c4b3815)

![image](https://github.com/denis6715/activity_schema_mfo/assets/94977703/85a3fa89-c2ec-41d2-b83f-00e94c0ae087)

**5. Гипотеза:** если клиент подал заявку и быстро пришел в отделение (в этот или следующий день), то он наиболее заинтересован в займе.

![image](https://github.com/denis6715/activity_schema_mfo/assets/94977703/0d6e2396-4486-414c-b273-945388b5e6ca)

![image](https://github.com/denis6715/activity_schema_mfo/assets/94977703/22f3f9f3-129e-4d2a-b0df-ec0fad380ba3)

**6. Гипотеза:** так как при достижении суммы займа в 50000 и 100000 существуют дополнительные условия в виде требования справки о доходах и залога соответственно, то клиенты, желаемая сумма которых лишь немного превышает пороги будут отказываться на последнем этапе (например, пожелают найти МФО без таких условий для своей суммы), поэтому у конверсии на порогах будут скачки. Возможным решением данной проблемы является предложение в отделении суммы, равной порогу.

![image](https://github.com/denis6715/activity_schema_mfo/assets/94977703/e7f5d59b-22eb-4386-a6af-ba35372e0b7e)

![image](https://github.com/denis6715/activity_schema_mfo/assets/94977703/083318a6-1151-4903-a32b-131a73091b33)

**7. Гипотеза:** если клиент во время заполнения полей использует backspace, значит он вводит данные руками, перепроверяет правильность введенных данных и более заинтересован в быстром займе (логично считать статистику по клиентам, которые дошли до полей с вводом), проанализируем среднюю конверсию в квантилях количества backspace.

![image](https://github.com/denis6715/activity_schema_mfo/assets/94977703/496fbb7f-f7fc-4b9a-a936-fdb0f6cc8c1b)

![image](https://github.com/denis6715/activity_schema_mfo/assets/94977703/a0048227-7142-4c21-9d4a-97b55610e0b0)




