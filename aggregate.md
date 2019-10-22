## Описание алгоритмов смарт-контрактов для агрегированной подписи 

Кратко о последовательности действий и исполнителях транзакций. 

Напомним, что все транзакции в  SHARED подтверждаются транзакциями валидации в LEDGER. 
1. Владелец ноды имеет счет во внешней крипто-сети и его баланс отображается в общем списке его счетов мультивалютного кошелька;
2. У владельца возникает желание обменять свой актив из внешней сети на актив в той-же, либо EVEN, либо другой внешней сети;
3. Он открывает прокси-счет (брокерский), управляемый сетью EVEN в сети актива, который он хочет поменять;
4. Для этого он генерирует транзакцию в SHARED с предложением обмена N-го актива* (формат транзакции - в транзакциях). В транзакции указываются:
   - адрес счета владельца внешней сети;
   - размер актива, если есть - сумма;
   - сигнатуру;
   - мультихеш файла смарт контракта, который генерирует **транзакции для агрегируемой подписи***;

5. Ноды-валидаторы, имеющие доступ к сети предлагаемого ресурса в силу того, что экономически заинтересованы в организации обмена, постоянно мониторят этот тип транзакций, находят и выполняют следующие действия:
   - загружают транзакцию со смарт контрактом и выполняют его;
   - на вход смарт контракта нода передает публичный ключ, секретное случайное число из диапазона G ECDSA;
   - смарт-контракт (*сценарий виртуальной машины ноды*) **генерирует часть агрегированной подписи и бинарную часть целевого смарт-контракта (!)**;
   - выдает результаты в виде транзакции, которая содержит:
     1. часть мультиподписи;
     2. результат Палле хеш-разницы от ключа агрегированной подписи; 
     3. часть бинарника целевого контракта с указанием его индекса;
     4. сигнатуру ноды валидатора, которая шифрует секрет формулы шифрования подписи (прочитает только валидатор);
     5. адрес “заявки” на открытие счета;
     6. адрес счета инициатора;
   - валидатор помещает транзакцию в папку SHARED (напоминаем: все валидируется PoSn);
6. Количество валидаторов и их подписей регулируется смарт контрактом в процессе работы - мониторит по заданному алгоритму количество *партиалов* (вроде нормальное слово) в SHARED по хешу заявки;
7. **Внимание!**: *кошелек ноды инициатора после обновления статуса и подсчета баланса сообщает о готовности сети к открытию брокерского счета и дает ему возможность либо подтвердить операцию, либо сделает это автоматически*;
8. Для этого выполняется другой смарт-контракт в **адресном пространстве ноды заявителя**, который собирает валидированные партиалы  и последовательно (Палле-гомоморфно) суммирует сегменты подписей и бинарников: последовательность не имеет значению, так как генерация происходит m x n по какому-то критерию (настраивается);
9. По достижению пороговой суммы, **формируется агрегированная подпись, которая является валидной для сформированного из сегментов смарт-контракта** и он автоматически выполняет целевую функцию - в данном случае открытие брокерского счета, генерирует транзакцию в SHARED об открытии счета, с указанием всех вышеописанных атрибутов вкупе с хеш-адресом открытого счета;
10. В кошельке инициатора появляется брокерский счет;
11. Если он не передумал, то переводит деньги используя соответствующие транзакции сети, об активе которой идет речь, на брокерский счет;

Таким образом, более детальное описание алгоритмов смарт-контрактов CIP протокола выглядит следующим образом.

**Смарт-контракт генерации сегментов (партиалов) агрегированной подписи:**

> Сразу оговоримся, что стратегией функции является не конкретный хеш, а упорядочение аргументов фукнций гомоморфного шифрования - секретов. Поэтому сколь угодное количество разных партиалов, содержащие  правила формирования секрета дает правильное решение ключа.

```
Input: kr Key Of Request (hash), pk Publik Key (hash), r (random from G) 

Output: bool

Прочитать из SHARED уже созданные транзакции по данному смарт-контракту c учетом валидаций. 
Если таковых нет  
{ 
сгенерировать новую ключевую пару;
хешировать ключевую пару с использованием гомоморфной функции;
создать целевой смарт контракт в бинарном виде с её использованием;
} 
 если есть
 
читать их все и извлечь разницу (п.2);
прочитать индексы частей смартконтракта (не страшно, что они будут повторятся, это даже хорошо); 
}
 
Если разница позволяет сделать гомоморфное разделение суммы разниц (остатка) на m-ную долю от n
{
делаем расчет собственной доли с использованием случайного чиcла r;
получаем свою долю ключа;
} 
если нет
{
прекращаем выполнение - return false;
}

Далее  вычисляем гомоморфный хеш ключа и восстанавливаем целевой смарт-контракт, генерируем бинарник и выделяем из него любую свободную часть;

 Формируем транзакцию (п.2);
 ```

**Смарт-контракт агрегированного решения:**
```
Input:  pk Publik Key (hash) инициатора

Output: bool

Прочитать из SHARED уже созданные партиал транзакции по данному смарт-контракту  c учетом валидаций:
поиск сначала по pk - ищем запрос на открытие, и потом по нему - транзакции . 
Если таковых нет  
{ 
прекращаем выполнение - return false;
} 

Последовательно по списку полученных транзакций, в цикле:
{ 
проводим гомоморфное суммирование частей ключа из транзакций;
конкатенацию целевого смарт-контракта;
тестируем - Run;
если возвращает true - выходи с true;
}

Выходим с false;
```
**Смарт-контракт целевой функции** (пример: открытие счета, может быть разным, в зависимости от назначения. По сути первые два - это шаблоны): 

```
Input:  pk Publik Key (hash) инициатора, apk Publik Key (hash) агрегированной подписи

Output: bool

Прочитать из SHARED транзакцию-запрос на открытие брокерского счета c учетом валидаций;
Если таковых нет  
{ 
прекращаем выполнение - return false;
} 

		Отправляем транзакцию на открытие брокерского счета по apk;

Выходим с true;
```