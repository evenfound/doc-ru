## Rated DAG и DAG

Блокчейн встраивает кошелек в машины, в результате чего машины могут вести собственный баланс прибылей и убытков, а также дает возможность машинам совершать транзакции с другими машинами в автоматическом режиме. Новые модели бизнеса, ориентированные на взаимодействие между машинами и формы обмена ценностями между ними, появляются чуть ли не каждый день. Однако в том, что касается масштабирования, требований к вычислительным ресурсам и стоимости транзакций, современные технологии блокчейна имеют серьезные ограничения.

В этой связи новым адекватным способом решения известных проблем является технологии распределенного хранения реестра (DLT), которая с каждым днем все глубже проникают в реальный сектор экономики. Причем, они находят себе применение даже в таких традиционно консервативных видах бизнеса, как добыча нефти и газа. Практика показывает, что DLT может успешно применяться везде, где есть большие массивы информации, объединенные в базы данных и нужна их неизменность.

Одной из базовых моделей, успешно реализованных в проектах DLT (IOITA, Byteball) на сегодняшний день является DAG - это тип технологии распределенных реестров, отличающийся от Блокчейнов структурой записей и асинхронностью. 

В классической технологии блокчейн транзакции в начале группируются в блок, после чего, по прошествии какого-то времени такой блок добавляется в цепочку из таких блоков. В такой системе пропускная способность и время транзакции ограничены размером блока и временем, которое необходимо для генерации нового блока.

![Screenshot](_images/1.png)
 
В модели DAG - направленный ацикличный граф (от английского Directed Acyclic Graph) каждая транзакция сразу же добавляется в такой граф, состоящий из множества транзакций, записывающихся не последовательно, а одновременно. Здесь нет блоков, поэтому нет и проблем с его размером.

![Screenshot](_images/2.png)

В такой структуре пользователи сами обслуживают сеть. Прежде, чем отправить какую-либо транзакцию, необходимо подтвердить не менее одной, а чаще две предыдущих. Именно поэтому здесь нет майнеров и мастернод.

Преимущества такого решения:
- Быстрота выполнения транзакций
- Нет комиссий (или они мизерны)
- Более масштабируем по сравнению с блокчейн.

Однако видимые преимущества данной модели в чистом виде компенсируются следующими недостатками:

- Проблемы с масштабируемостью (необходимо синхронизировать блокчейн каждый раз при добавлении новой транзакции).
- При малом количестве участников сети, транзакции могут подтверждаться длительное время, а при их отсутствии вообще никогда не быть подтвержденными.

Имеющиеся недостатки, как показала практика эксплуатации подобного рода сетей, наносят тяжелый урон завоеваниям блокчейн технологий, и речь здесь идет о децентрализации. Например в IOTА, для управления всеми ветвлениями Tangle используется нода Coordiantor, которая по сути является сервером валидации. 

Есть ли пути решить эту проблему и каким-то изящным способом устранить имеющиеся недостатки и превратить их в достоинство?

Анализ существующих подходов в реализации DLT и алгоритма DAG - в частности, позволил предположить, что такое решение есть. При изучении изображений графов транзакций, например сети IOTA,  нетрудно заметить, что временная ось эволюции графа направлена справа налево. Да, так и должно быть:  граф направленный и ациклический, новая транзакция является контентно ориентированным сообщением и в силу этого факта, не ведая своего места в топологии сети, вынуждена уповать на “милость” координатора. И если её не дождаться (а так бывает), транзакция может и не подтвердиться. 

Наше решение, которое позволило изменить направление времени эволюции в DAG и добиться максимальной децентрализации,  это переопределение сущности транзакции из контентно в топологически ориентированную и использование алгоритма подсчета рейтинга. При этом все преимущества модели сохраняются в полном объеме.

**В чем сущность модернизации?**

*1. Так же, как и в классическом DAG, новое сообщение, которое похожим образом формируется из цепочки связанных транзакций со ссылками на хеш заголовки своих транзакций в качестве trunk ветки, представляющие баланс , и в качестве brunch - ссылки от одной до нескольких связанных в сообщения транзакций, которые нода получила для валидации в приватном разделе входящих распределенного хранилища;*

*2. Используя алгоритм подсчета рейтинга, который будет описан ниже, нода формирует список рассылки двух вариантов передач: в первом случае своего сообщения с необходимыми привязками  к графу trunk и brunch,  и во втором - передачу полученных сообщений с признаком валидации после проведенной работы по их проверке с использованием алгоритма МАМ;*

*3. Нода может быть пассивной и ничего не делать - не проводить работу по валидации и транслировать сообщения, но тогда это отразится на её рейтинге, и чтобы осуществить собственные транзакции по переводу средств, ей придется заплатить некоторую комиссию за предоставление сетью необходимого пакета сообщений для валидации.*

Здесь возникает вопрос: каким образом решается вопрос создания топологии в криптозащищенной децентрализованной сети? Как контентно-ориентированные сообщения переходят в разряд адресных посылок без потери конфиденциальности?

Решения  этой проблемы представляется возможным с использованием в базовом протоколе технологии IPFS. Термин «децентрализация» приобрел новое значение с приходом эры криптовалют и технологии блокчейн. Появилось множество новых проектов, аналогов которым просто не существует. Interplanetary File System (IPFS) — это один из характерных примеров. В чем особенность новой технологии и какие возможности она открывает? 

IPFS объединяет в себе шардинг и децентрализованное хранение файлов. Да, большое количество компанией занимается разработкой решений для хранения файлов по частям, в частности, Sia и поддерживаемый корпорацией Google Storj. Однако IPFS работает по другому принципу, наиболее точно воплощая в жизнь симбиоз этих технологий.

Проект начинался как протокол в сети Эфириум в 2016 году. Был выбран именно этот блокчейн из-за большей дружелюбности по отношению к новшествам и инновациям. Биткоин подобным похвастаться не мог. Насколько мудрым было это решение, неясно до сих пор.

Из определения в Wiki мы получаем ответ на главный вопрос: IPFS представляет собой одноранговую распределенную файловую систему, которая соединяет все вычислительные устройства единой системой файлов. В некотором смысле IPFS схожа со всемирной паутиной. IPFS можно представить как единый BitTorrent-рой, обменивающийся файлами единого Git-репозитория. Иными словами, IPFS обеспечивает контентно-адресуемую модель блочного хранилища с контентно-адресуемыми гиперссылками и высокую пропускную способность. **_Это формирует обобщенный древовидный направленный граф._** IPFS сочетает в себе распределенную хэш-таблицу, децентрализованный обмен блоками, а также самосертифицирующееся пространство имён. При этом IPFS не имеет точек отказа, и узлы не обязаны доверять друг другу. Доступ к файловой системе может быть получен различными способами:

- через FUSE
- поверх HTTP.

Локальный файл может быть добавлен в файловую систему IPFS, что делает его доступным всему миру. Файлы идентифицируются по их мультихэшам, что упрощает кэширование. Они распространяются через протокол, основанный на протоколе BitTorrent. Пользователи, просматривающие контент, помогают в доставке контента для других пользователей сети. IPFS имеет сервис имён под названием IPNS, глобальное пространство имен на основе открытых ключей, совместимое с другими пространствами имён и имеющее возможность интегрировать DNS, .onion, .bit и т. д. в IPNS.

Из определения технологии, особенно в части формирования DAG графов, а она действительно использует алгоритм DAG, нетрудно заметить родственность  технологии и возможность их симбиоза в части реализации необходимых для базового протокола свойств.