В заголовке, пожалуйста, используйте портрет Ф.Фурье. Вам все равно, а мне приятно. 
https://yandex.ru/images/search?text=%D1%84%D1%83%D1%80%D1%8C%D0%B5%20%D0%BF%D0%BE%D1%80%D1%82%D1%80%D0%B5%D1%82

Аллитерирующиеся продажи и колтрекинг
Изучаем
Мы очень любим передовые технологии. Скоро год, как мы познакомились с колтрекингом (call tracking). Это такая технология, которая показывает каждому посетителю на сайте уникальный номер рекламного телефона и привязывает этого посетителя к рекламной кампании. Таким образом к тривиальному для нас и для вас (поскольку этот опыт уже описывался в статьях http://qlikdaily.ru/podklyuchaem-google-analytics-v-qlikview/ и http://qlikdaily.ru/podklyuchaem-yandeks-metriku-v-qlikview/) анализу добавляется факт телефонного контакта с клиентом (не только входящего, но и исходящего, ибо в продвинутых системах колтрекинга имеется возможность заказа обратного звонка).
На самом деле эта технология появились еще в прошлом веке. Кооператор давал объявление в газету «Из рук в руки», указывая телефон тещи, а в газету «Все для вас» свой, а потом смотрел, кто продал больше. Уже тогда он мониторил эффективность вложений в рекламу.
К сожалению, у нас размер средней покупки невелик и хорошо, если первая продажа,  покрывает наши затраты на привлечение клиента. К счастью повторные покупки регулярны как строфы стиха (20% клиентов покупают раз в неделю, 50% клиентов покупают в течении месяца, 90% в течении квартала). Возникает идея растянуть воронку продаж на вторые, третьи и т.п. сделки, подтянув всю историю продаж. Таким образом можно решать задачу нахождения глобального максимума прибыли в вопросе выбора рекламной кампании, а не поиска локального экстремума в конверсии до 1-й покупки. Кстати, факт нажатия кнопки «купить» не гарантирует продажу, а тем более оплату. 
С точки зрения аналитики, обогащение рекламных данных звонками позволяет построить более детальную воронку продаж, зафиксировав телефонные контакты, случившиеся по дороге.  В некоторых отраслях считается, что с клиентом надо обязательно поговорить, подкрепив его правильный выбор. В таких случаях, аналитика позволит учесть предварительные ласки разговоры и отразить их в затратах на клиента, а может даже развенчать некоторые мифы.
Тренируемся
Есть поставщик, которая предоставляет услуги колтрекинга. Дабы не считать этот пост рекламой, называть его не буду, а интересующиеся могут подсмотреть его домен в строках подключения. Есть история звонков, переплетенная с рекламными кампаниями. Более того, на сайте поставщика прекрасно описан REST API (https://ru.wikipedia.org/wiki/REST), позволяющий забирать интересующие нас данные. 
Пробуем подключиться к ним из QlikView по http. Ан нет, не получается. Сервер возвращает ошибку «111406 Not acceptable». Расследование показало, что QlikView (11.20SR11, а также QlikSense 2.01) не заполняет параметр Accept в GET запросе. Отдающий сервер не может самостоятельно принять ответственное решение о выборе формата, и вместо того, чтобы выполнить свою работу в формате по умолчанию,  бодро рапортует об ошибке. Кого-то мне это напоминает… Печалька. Покупать прекрасный продукт General QVSource коннектор (http://market.qlik.com/qvsource-the-qlikview-api-connector.html), в котором этой проблемы нет, который поддерживает json, умеет работать с Эдвардз и Google Analitics, подключается к социальным сетям, варит кофе, не позволяет бюджет. Для диагностики используем утилиту curl. С ней API REST прекрасно взаимодействует и отдает нужные нам данные.
curl -A QlikView -H "Accept: application/xml" -v "http://api.comagic.ru/api/login/?login=L&password=P"

<?xml version="1.0" encoding="UTF-8" ?><root><data type="dict"><session_key type="str">777...</session_key></data><success type="bool">true</success></root>

curl -A QlikView -H "Accept: application/json" -v "http://api.comagic.ru/api/login/?login=L&password=L"
{"data": {"session_key": "777..."}, "success": true}

curl -A QlikView -H "Accept: application/html" -v "http://api.comagic.ru/api/login/?login=L&password=P”
{  "message": "111406:Not Acceptable",  "success": false}

curl -A QlikView -H "Accept: ;" -v "http://api.comagic.ru/api/login/?login=L&password=P"
{  "message": "111406:Not Acceptable",  "success": false}

Ошибка возникает только в последних командах при запросе данных в HTML и в «пустом» формате.
Вот пример получения данных из QlikView с помощью wget
[code]
rem Подключаемся к целебному источнику данных;
Execute wget.exe "http://api.comagic.ru/api/login/?login=LOGIN&password=PaSsWoRd" -O wget.json;

rem Вытаскиваем ключ сессии;
session_tab: LOAD  
TextBetween([@1:n],'"session_key": "','"',1) as session_key 
FROM [wget.json] (fix, codepage is 1251)
;
let sk=Peek('session_key');
DROP Table session_tab;

rem Список рекламных кампаний;
let req='http://api.comagic.ru/api/v1/ac/?session_key=' & sk;
Execute wget.exe "$(req)" --header="Accept: application/xml" -O СписокРекламныхКомпаний.XML;

rem Список сайтов;
let req='http://api.comagic.ru/api/v1/site/?session_key=' & sk;
Execute wget.exe "$(req)" --header="Accept: application/xml" -O СписокСайтов.XML;

rem Список тегов;
let req='http://api.comagic.ru/api/v1/tag/?session_key=' & sk;
Execute wget.exe "$(req)" --header="Accept: application/xml" -O СписокТегов.XML;

for d=today(2)-1 to today(2)-7 step -1;// Перегружаем данные за последнии несколько дней

let ds=date(d,'YYYY-MM-DD'); //Дата начала периода 
let de=date(d+1,'YYYY-MM-DD'); //Дата окончания периода (исключительно)

rem Информация о звонках;
let req='http://api.comagic.ru/api/v1/call/?session_key=' & sk & '&date_from=' & ds & '%2000:00:00&date_till=' & de & '%2000:00:00'; //limit=100 
Execute wget.exe "$(req)" --header="Accept: application/xml" -O $(ds).XML;

NEXT d; 

rem Завершение сессии;
let req='http://api.comagic.ru/api/logout/?session_key=' & sk;
Execute wget.exe "$(req)" --header="Accept: application/xml" -O wget.json;
[/code]
QlikView здесь вовсе не при чем и выступает в качестве манипулятора текстовыми строками. Но наши злостные админы запрещают запускать негуевые утилиты на боевом сервере. Поэтому обратились к поставщику данных и, о чудо, после нескольких раундов напряженных переговоров, поставщик согласился внести изменения в свой REST API.
Реализуем
В REST API появилась недокументированный (?) параметр запроса accept, в который мы и запихнем непередаваемые QlikView данные для запроса. В нашем случае это accept=application/xml
Что значит: хотим получить данные в формате XML. Json экономнее, но QlikView пока с ним не дружит.
Новый код, без использования внешних утилит
[code] 
rem Получение ключа сессии;
tmpSID: LOAD 
success, // надо бы поверять на 'true'
[data/session_key] as session_key
FROM [http://api.comagic.ru/api/login/?accept=application/xml&login=LOGIN&password=PASSWORD] (XmlSimple, Table is [root]);

let sk=Peek('session_key');
DROP Table tmpSID;

rem Список рекламных кампаний;
let req='http://api.comagic.ru/api/v1/ac/?accept=application/xml&session_key=' & sk;
ac: LOAD 
id,
name
FROM [$(req)] (XmlSimple, Table is [root/data/item]);
	STORE ac into [..\ComagicData\СписокРекламныхКомпаний.qvd] (qvd); 
DROP Table ac;

rem Список сайтов;
let req='http://api.comagic.ru/api/v1/site/?accept=application/xml&session_key=' & sk;
site: LOAD 
id,
domain
FROM [$(req)] (XmlSimple, Table is [root/data/item]);
	STORE site into [..\ComagicData\СписокСайтов.qvd] (qvd);
DROP Table site;

rem Список тегов;
let req='http://api.comagic.ru/api/v1/tag/?accept=application/xml&session_key=' & sk;
tag: LOAD 
id,
name
FROM [$(req)] (XmlSimple, Table is [root/data/item]);
	STORE tag into [..\ComagicData\СписокТегов.qvd] (qvd);
DROP Table tag;

for d=today(2)-1 to today(2)-7 step -1;// За последние несколько дней
set ErrorMode=0;	
let ds=date(d,'YYYY-MM-DD'); //Дата начала периода 
let de=date(d+1,'YYYY-MM-DD'); //Дата окончания периода (исключительно)

rem Информация о звонках;
let req='http://api.comagic.ru/api/v1/call/?accept=application/xml&session_key=' & sk & '&date_from=' & ds & '%2000:00:00&date_till=' & de & '%2000:00:00'; //limit=100 
call: LOAD 
ua_client_id,
gclid,
utm_content,
numa,
visitor_id,
wait_time,
duration,
is_transfer,
id,
search_engine,
utm_term,
call_date,
session_start,
visits_count,
utm_medium,
scenario_name,
city,
status,
os_ad_id,
utm_campaign,
communication_type,
operator_name,
utm_source,
referrer_domain,
os_service_name,
visitor_type,
numb,
region,
os_campaign_id,
visitor_first_ac,
communication_number,
referrer,
site_id,
session_id,
ac_id,
other_adv_contacts,
search_query,
country,
os_source_id,
yclid,
page_url,
[tags/item],
[file_link/item]
FROM [$(req)] (XmlSimple, Table is [root/data/item]);
STORE call into [..\ComagicData\ИнформацияОЗвонкахQVD\$(ds).qvd] (qvd);
DROP Table call;
NEXT d; 

rem Завершение сессии;
let req='http://api.comagic.ru/api/logout/?accept=application/xml&session_key=' & sk;
tmpSID: LOAD success FROM [$(req)] (XmlSimple, Table is [root]);

set ErrorMode=1;
[/code] 
Новый код ничем не отличается от старого, за исключением установки переменной ErrorMode и сброса, поскольку загрузка серверов поставщика и канала данных иногда приводит к ошибке по тайм ауту. Кроме того, этот код сохраняет исходные данные в qvd файлы, которые позже легко собрать.
Рефлексируем
После эйфории от интеграции, пришло понимание, что колтрекинг работает на больших числах (QlikView, впрочем, тоже). Но преимущество QlikView в том, что можно добраться до самой пустячной сделки и это будет факт — в случае колтрекинга, наоборот, тренды видны только в массе, опускаться до конкретного контакта нужно с осторожностью: а вдруг этот посетитель стирает по ночам куки или того хуже — параноик, а вдруг он записал уникальный рекламный номер месяц назад, а позвонил только сегодня, а вдруг он просто забил номер телефона в записную книжку и звонит с него, не заходя на сайт.
Требуйте отстоя пены!
Не всегда с первого раза удается привычными инструментами добиться результата. В описанном случае и данные: вот они, да не уцепишь. Вроде и подсекли, а рыбка-то сорвалась, кто же даст права на исполнение безокошечного wget на сервере?!
Пробуйте!
В процессе коммуникации с поставщиком данных самым сложным было объяснить, что это не его ошибка, ибо он сразу выставлял несколько линий техподдержки для обороны. И можно было бы сослаться на RFC, но ради результата пришлось закусить удила. 
Закусывайте!
Для полноценной интеграции пришлось пообщаться на стороне поставщика и с поддержкой, и с продавцами, и с директором по маркетингу, и даже с генеральным директором. Нами было предложено несколько направлений решения этой задачи, в ходе обсуждения поставщик сам выбрал наиболее приемлемое со своей стороны: изменение API REST.
Общайтесь!

Как обычно, исходники на GitHub
https://github.com/Q2B3/Qlik-CoMagic

А чему вас научило последнее разработанное вами приложение QlikView?
