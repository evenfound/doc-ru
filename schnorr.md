## Агрегированная подпись Schnorr с валидацией ECDSA.

Сила _ECDSA _заключается в том, чтобы доказать, что полученный фрагмент данных может быть получен только от пользователя, который имеет закрытый ключ к используемому открытому ключу. Иными словами, владелец закрытого ключа может легко создавать действительные подписи, а все остальные могут легко проверить подпись с помощью открытого ключа, созданного из этого закрытого ключа, даже не зная закрытого ключа.

Формальная математика, стоящая за подписанием и проверкой с помощью _ECDSA_, может быть найдена в [3] , но основной алгоритм состоит в следующем, предполагая циклическую группу _T_, простой порядок _p_ и точку генератора_ G_ в _T_:

Подпись:

1. Формируем ключевую пару приватный/публичный ключ $(x,  P)$, где $P = xG$;
1. Выбираем случайное число $r$ из диапазона генератора;
2. Вычисляем $R = rG$;
3. Выбираем сообщение $z$;
4. Рассчитываем $s = (z + Rx) / r$;
5. Подпись $(R, s)$ готова;

Проверка:

1. Получаем подпись $(R, s)$;
2. Рассчитываем $u=z/s$;
3. Рассчитываем $v=R/s$;
4. Если равенство $rG = uG + vP$ истинно, то подпись верна.

Подпись _ECDSA_ прекрасно работает. К сожалению, он использует _разделение полей_ , что является сложным в вычислительном отношении, и его использование для мультиподписей требует подписи и проверки каждого участника транзакции с несколькими подписями.

Подписи Шнорра[5] были впервые созданы _К.П. Шнорром_ в 1989 году. Затем он запатентовал алгоритм подписи. Срок действия патента истек в 2008 году, и, вероятно, именно поэтому _Сатоши Накамото_ использовал в Bitcoin _ECDSA_ вместо _Schnorr_. Алгоритм подписи Шнорра устраняет дорогостоящее деление полей _ECDSA_ и заменяет его _хэшированием_. Полное математическое доказательство можно найти в [5] .

Подпись:

1. Формируем ключевую пару приватный / публичный ключ $(x,  P)$, где $P = xG$;
2. Выбираем случайное число $r$ из диапазона генератора;
3. Вычисляем $R = rG$;
4. Выбираем сообщение $z$;
5. Вычисляем новую переменную $c = H (P, R, z)$;
6. Рассчитываем $s = r + cx$;
7. Подпись $(R, s)$ готова;

Проверка:

1. Получаем подпись $(R, s)$;
2. Если равенство $sG = R + cP$, где $c = H(P, R, z)$ - истинно, то подпись верна;

В [4] _Питер Уилле_ кратко описывает схемы с несколькими подписями:

>_“Схема мульти-подпись представляет собой комбинацию алгоритма подписи и проверки, где несколько подписантов (каждый со своей ключевой парой) совместно подписывают одно сообщение, в результате чего получают одну подпись. Эта единственная подпись может быть проверена любым, кто знает сообщение и открытые ключи подписывающих лиц. Обратите внимание, что в контексте Биткойна термин «мультисигнал» обычно относится к политике $k$ - из - $n$ , где $k$ может отличаться от $n$ . В криптографической литературе мульти-подписи действительно относятся только к политикам $n$-из-$n$ , хотя   мы можем легко построить $k$-из-$n$ поверх  $n$-из-$n$“._


Как упоминалось ранее, мульти-подписи для _ECDSA_ просты, но не эффективны и не имеют никакой конфиденциальности для тех, кто участвует в транзакции с несколькими сигнатурами: публичный ключ каждого участника открыт.

Шнорр меняет правила игры на мульти-подписи. _Schnorr_ позволяет использовать собственную мульти-подпись благодаря использованию алгоритма хеширования вместо деления поля. Та же самая математика, приведенная выше, работает для группы из $n$ участников транзакции, каждый из которых имеет свою собственную ключевую пару. Есть только несколько изменений в уравнении подписи выше:

1. $X=\sum\limits_{i=0}^NP_i$ - сумма публичных ключей всех участников подписи;
2. Каждый участник выбирает свой уникальный $r_i$ и вычисляет свой уникальный R_i=r_iG, а $R_\Sigma$ является суммой всех $R_i$;
3. Далее, каждый подписант должен вычислить свой собственный $s_i=r_i+c_ix_i$, а $S_\Sigma$ - это сумма всех;
4. Сигнатура теперь является суммой всех уникальных $R$ и всех уникальных $s$, дающих сигнатуру $(R_\Sigma,S_\Sigma)$;
5. Проверка идентична одной транзакции, за исключением того, что публичный ключ $P$ заменяется суммированием всех $P, X$.
6. Если истинно равенство $S_\Sigma G=R_\Sigma+c_iX_i$,  где $c_i=H(X_i,R_i,z)$, то подпись верна.

Проверка нескольких подписей _Schnorr_ аналогична проверке единственной подписи _Schnorr_, заменяющей $P$ на $X$, поэтому проверка не требует знания отдельных открытых ключей, только совокупного открытого ключа.

Теперь очевидно, почему у _Schnorr_ есть встроенная функция мульти-подписи. Для проверки алгоритма подписи просто берется ОДНА подпись $(R_\Sigma,S_\Sigma)$. Невозможно узнать, является ли эта подпись одним или несколькими лицами - это выглядит одинаково. Эта функция известна как агрегация подписи.

К сожалению, существует легкая атака на схему множественных подписей _Schnorr_. _Питер Уилле_ [4] объясняет:

>_“У Алисы есть пара ключей (xA, XA), а у Боба (xB, XB) . Однако ничто не мешает Бобу утверждать, что его открытый ключ XB' = XB - XA . Если он это сделает, другие будут предполагать, что XA + XB' - это агрегированный ключ, с которым Алисе и Бобу нужно взаимодействовать, чтобы подписаться. К сожалению, эта сумма равна XB , и Боб может четко подписать это сам.”_

Это означает, что необходимо использовать решение, которое позволит продолжать использовать агрегацию сигнатур, но исключит эту атаку.

Алгоритм подписи _Bellare-Neven (BN)_ был предложен, чтобы остановить эту атаку. Это адаптация сигнатуры Шнорра, где вводится новая переменная, _L_- хэш публичного ключа каждого участника. Но в то время как _BN_ устраняет атаку, осуществляемую несколькими подписями с использованием сигнатур Шнорра, она просто вновь вводит проблему, которая была в _ECDSA_: _проверяющему нужно снова знать публичный ключ каждого участника_. Теперь мы теряем преимущества конфиденциальности, которые мы достигли с помощью подписей _Schnorr_, поэтому _BN _работать не будет.

_Грег Максвелл_, _Эндрю Поэльстра, Янник Сёрин_ и _Питер Уилле_ придумали решение [4], позволяющее сохранить агрегацию сигнатур и исключить атаку, которую ввели наивные сигнатуры Шнорра. Они назвали его _MuSig_, и он основывается как на _Schnorr_, так и на _BN_.

Подпись:

1. $L_i=H(P_i)$- хэш публичного ключа каждого участника;
2. Вводится переменная $X=\sum\limits_{i=0}^NH(L_i,P_i,)P_i$;
3. Каждый подписант вычисляет свой уникальный $r_i$ и вычисляет свой уникальный $R_i=r_iG$, а $R_\Sigma=\sum\limits_{i=0}^NR_i$ - сумма  всех $R_i$;
4. $c$ также меняется от уравнения _BN_. $c_i=H(X,R_\Sigma,z)\alpha$. Введена новая переменная $\alpha$, где каждый участник использует свой уникальный открытый ключ для вычисления $\alpha=H(L_i,P_i)$. Таким образом, $c_i=H(X,R_\Sigma,z)H(L_i,P_i)$;
5. Каждый участник теперь должен вычислить свой собственный $s_i=r_i+c_ix_i$, а $S_\Sigma=\sum\limits_{i=0}^Ns_i$ - это сумма всех $s_i$;
6. Сигнатура теперь является суммой всех уникальных $R$ и всех уникальных $s$, дающих сигнатуру $(R_\Sigma,S_\Sigma)$.

Проверка:

1. Получаем подпись $(R_\Sigma,S_\Sigma)$;
2. Если действительно равенство $S_\Sigma=R_\Sigma+c_iX$, где $c_i=H(X,R_\Sigma,z)$ - подпись верна.

Мы возобновили агрегирование подписи, поскольку алгоритм проверки не требует открытого ключа отдельного участника $P_i$, а вместо этого требует только суммирования всех $P_i - X$. Это означает, что транзакция с одной подписью будет выглядеть точно так же, как и транзакция с несколькими подписями участников. Это дает гораздо большее увеличение конфиденциальности.

Но, что еще важнее, _MuSig_ ввел ключевое агрегирование, которое устраняет уязвимость обобщенных сигнатур Шнорра. Поскольку $X$ теперь является суммой хэша $L_i$ и их открытого ключа $P_i$, умноженного на $P_i$, теперь нам нужен только один открытый ключ для транзакций с несколькими подписями. Узлы не смогут видеть различные открытые ключи $P_i$, которые составляют значение $X$.
_MuSig_ приводит к собственным и частным транзакциям с несколькими подписями с агрегацией сигнатур - только одна подпись  $(R_\Sigma,S_\Sigma)$, и агрегацией ключей:  только с одним $X$ для представления «публичного ключа» группы - для создания и проверки транзакций.

Преимущества _MuSig_ многочисленны:

1. Данные подписи для транзакций с несколькими подписями могут быть большими и громоздкими. _MuSig_ позволит пользователям создавать более сложные транзакции, не обременяя сеть и не раскрывая компрометирующую информацию;
2. Сегодня более эффективен, чем отдельные транзакции _ECDSA_, поэтому это должно облегчить работу новых и существующих узлов;
3. Это также означает, что это должно создать больше места в каждом блоке и, следовательно, снизить комиссию за транзакцию.
4. Стимулирует использование других инструментов конфиденциальности, таких как _coinjoin_.

Все это красиво выглядит но в настоящее время практически ни в одной сети без адаптации не применимо. 

Обратим внимание на процесс верификации подписей. 

Если в большинстве сетей, использующих алгоритм верификации подписи _ECDSA _верификация подписи $(R,s)$ с публичным ключом $P$ выглядит как:

$$rG=uG+vP\tag{1}$$ где $u=z/s$ и $v={R}/{s}$


То в версии _MuSig_- по другому:
$$S_\Sigma G=R_\Sigma+c_iX\tag{2}$$ где $c_i=H(X,R_\Sigma,z)$
что позволяет предположить, что валидация сообщений практически во всех сетях, использующих подпись _ECDSA_, будет сомнительной. 

Для того, чтобы валидация по алгоритму _MuSig_ была применима в сетях с валидацией _ECDSA_ предлагается использовать следующий метод верификации подписи.

Первым делом нужно провести анализ переменных обоих уравнений  и попробовать определить их подобие:

1. Переменная $G$, на самом деле, является константой общей для обоих;
2. Переменные $s$ из (1) - это аналог $S_\Sigma$из (2), $R$ из (1) - это аналог $R\Sigma$ из (2);
3. Учесть $r$ из (1) можно заменив запись $rG$ на $R$, которую в свою очередь можно представить как $R_\Sigma$;
4. $z$ из (1) - это $c_i$ из (2);
5. Из приведенные в (1) переменных, неясно как с использованием мультиподиси _Schnorr_ будет учитываться значение $P#;

Из приведенных наблюдений видно, что практически все переменные, используемый в транзакции для подписи идентичны и вычислимы, за исключением _P_, которой нет, как таковой в (2) и которая вообще в ней нежелательна, так как дискредитирует подписанта. Попытаемся выразить _P_ через равенство следующего вида, при условии что $rG$ из (1) - это $R_\Sigma$ из (2):
$$\frac{c_i}{S_\Sigma}G+\frac{R_\Sigma}{S_\Sigma}P=S_\Sigma G-c_iX\tag{3}$$

Далее, путем несложных преобразований, получаем:

$$P=R_\Sigma ^{-1}(S_\Sigma ^2 G-S\Sigma c_iX - c_iG)\tag{4}$$
После подстановки  $P$, полученного в (4) в (1) и с учетом подобия переменных, равенство верно:
$$ \frac{c_i}{S_\Sigma}G+\frac{R_\Sigma}{S_\Sigma}R_\Sigma ^{-1}(S_\Sigma ^2-S_\Sigma c_iX - c_iG)=S_\Sigma G-c_iX\tag{5}$$
Учитывая высокие требования к производительности вычислений и учитывая тот факт, что часть переменных не являются волатильными каждую транзакцию, часть из них, для ускорения расчетов, можно объединить и вычислять заранее.

Эксперименты в тестовым сетях сетей, использующих _ECDSA_ на основе кривой _secp256k1_ показали работоспособность предложенного метода при сохранении основных преимуществ _MuSig_ с не существенными потерями по скорости обработки.

[3] [Wikipedia, _Elliptic Curve Digital Signature Algorithm](https://en.wikipedia.org/wiki/Elliptic_Curve_Digital_Signature_Algorithm)
[4]  [Pieter Wuille, _Key Aggregation for Schnorr Signatures, January 23, 2018](https://blockstream.com/2018/01/23/musig-key-aggregation-schnorr-signatures/)
[5] [Wikipedia](https://en.wikipedia.org/wiki/Schnorr_signature)

## Пороговая подпись ECDSA с гомоморфным шифрованием

Гомоморфное шифрование позволяет производить операции над шифротекстами таким образом, что результат шифрования операций над ними будет равен результату операций над их шифрами. Это можно так проиллюстрировать:
$$ab_1=E(a)+eE(b)$$
$$ab_2=E(a+b)$$
$$D(ab_1)=D(ab_2)$$
Таким образом, мы можем иметь два значения - $а$ и $b$. Сначала мы можем зашифровать эти значения, а затем, используя эту специальную операцию, добавить эти два шифротекста вместе. Или мы можем сначала, имея эти два значения, сложить их вместе, а затем зашифровать результат. Когда мы расшифруем, мы получим одно и то же значение в обоих случаях. 
Одним из примеров аддитивно-гомоморфной схемы шифрования является $Paili$, для которой существует эффективно вычисляемая операция «добавить», так что операция сложения это плюс с индексом e в первой формуле, который позволяет добавлять два шифротекста вместе.

Следующее - это $(t, n)$ пороговое шифрование. Самый простой способ объяснить это - сказать, что:
* у нас есть n игроков; 
* каждый из них имеет один и тот же открытый ключ, но каждый игрок имеет свою уникальную долю секретного ключа. 

Схема обязательств- на примере:
> Например, скажем, что Алиса и Кэролайн играют в игру с монетами. Алиса выбирает сторону монеты, шифрует свой выбор и отправляет обязательства Кэролайн. Теперь Кэролайн подбрасывает монету в воздух и говорит, каков результат. Затем Алиса отправляет специальное значение, называемое ключом разложения, которое позволяет Кэролайн оценить обязательство и посмотреть, действительно ли значение, которое изначально было выбрано Алисой, действительно соответствует ее словам.
Ключ обязательства позволяет его подтвердить, но также имеет приятную особенность, позволяющую безоговорочно определять схемы сокрытия. Таким образом, независимо от того, что делает Кэролайн, независимо от того, насколько высока ее вычислительная мощность, она не может угадать значение, которое Алиса взяла на себя, не имея ключа декомпозиции. Она не может сделать это только из обязательства [cхема обязательства](https://ru.wikipedia.org/Схема_обязательства).

В криптографии, схема обязательств или битовая схема обязательств (англ. Commitment scheme) — это метод, позволяющий пользователю подтверждать какое-либо значение, которое не разглашается, то есть в случае разглашения этого значения благодаря этой схеме будет известно, что пользователь знал его на момент выдачи обязательства и что оно не изменилось. Работу данной схемы можно представить как две стадии:
* _«Commit»_ — посылку закрытой на ключ коробки (обязательство),
* _«Reveal»_ — более поздняя отправка ключа от коробки (значение).
 
#### _*Протокол генерации ключа для подписи*_.
 
У нас есть $n$ - подписывающих, каждый подписывающий инициализируется с помощью схемы аддитивного гомоморфного порогового шифрования, и это происходит на этапе настройки.
1. На первом этапе каждый игрок выбирает случайное целое число x , которое будет использоваться как доля секретного ключа этого игрока<br><br>$x_i\in Z_q\rightarrow y_i=x_iG$ и $[C_iD_i]=commitTo(y_i)$<br><br>$х$ не может быть больше, чем $q$  - количества эллиптических кривых, то есть количества точек на эллиптической кривой. Каждый игрок вычисляет $y$ как $g$ в степени $x$ . Это операция с эллиптической кривой, и, по сути, мы умножаем точку генератора кривой на $x$ . Это обозначение, которое вы часто найдете для групп, но поскольку эллиптическая кривая является группой, можно использовать ее и здесь. Затем каждый игрок вычисляет обязательство Ci для этого значения.  
2. Во втором раунде публикует обязательство  $C_i$ для всех игроков в группе. 
3. Затем, на третьем шаге, каждый игрок показывает x в зашифрованном виде $\alpha_i=E(x_i), y_i, D_i - ZKP(Zero-knowledge proof*)$ что означает: <br><br>$\exists x_i$ из $[q^{-3},q^{3}]$ так что $y_i=x_iG$ и $D(\alpha_i)=x_i$, где $q$ - мощность эллиптической кривой.<br><br>Шифрование выполняется с помощью аддитивно-гомоморфной схемы шифрования, которую инициализировали на этапе настройки. Более того, каждый игрок раскрывает общий доступ к открытому ключу, ключ к выводу из игры и доказательство отсутствия знаний $D_i$ о том, что все эти значения вместе имеют смысл. Это позволяет подтвердить обязательство $C_i$, но также позволяет всем игрокам увидеть, имеют ли смысл все те акции, которые игрок только что показал вместе.<br>Итак, доказательство с нулевым знанием говорит, что существует такое число $x_i$, что точка генератора кривой, умноженная на $x_i$, дает точку $y_i$, и $y_i$ является публичным в этот момент, потому что оно была только что обнаружена, и что если мы расшифруем значение $\alpha_i$ , то имеем только что опубликованный - это зашифрованный секретный ключ - мы получим это число $x_i$. <br>Конечно, это доказательство с нулевым знанием, поэтому невозможно угадать, что на самом деле представляет собой $x_i$, но здесь мы говорим о том, что он лежит в диапазоне от $[q^{-3},q^{3}]$ , и поскольку $q$ - это мощность эллиптической кривой, этот диапазон действительно огромен.
4. На четвертом шаге все подписывающие участники используют операцию добавления аддитивно-гомоморфной пороговой схемы для получения окончательного ключа t-ECDSA, поэтому можно добавить все зашифрованные доли $x_i$:<br><br>$\alpha=\alpha_1+e\alpha_2+e\alpha_3...+e\alpha_n = E(x_1)+E(x_2)+E(x_3)+...+E(x_n)$
$y= y_1+ y_2+ y_3+...+ y_n$<br><br>В результате мы получаем секретный ключ $\alpha$ в зашифрованном виде, все открытые общие доли y могут быть сложены - и в результате получаем открытый ключ. Операция сложения здесь - это просто сложение точек эллиптической кривой, так что это нечто простое.<br><br>$y - plaintext, t-ECDSA public key$
$\alpha=E(x), ciphertext, t-ECDSA private key$


#### _*Алгоритм генерации ключа для подписи*_

У нас есть $\alpha$, который является закрытым ключом $t-ECDSA$ в зашифрованном виде, совместно используемом всеми подписавшими, и у нас есть $y$, который является открытым ключом $t-ECDSA$  - это просто точка на эллиптической кривой.
 
1. В первом раунде каждый из участников вычисляет случайное целое число $p_i$:<br><br>$p\in Z+q$<br><br>Затем шифрует это значение с помощью аддитивно-гомоморфной схемы порогового шифрования:<br><br>$u_i=E(p_i)$<br><br>И умножает секретный ключ $ECDSA$  $\alpha$ на это случайное значение:<br><br>$v_i=p_i\alpha=E(p_i,x)$<br><br>Это возможно, потому что аддитивно-гомоморфная схема шифрования имеет операции не только сложения, но и умножения. 
2. Во втором раунде участники раскрывают все эти результаты вместе с доказательством нулевого знания, заявляя, что они имеют смысл и  объединяют выявленные акции вместе, используя операцию сложения, определяемую гомоморфной схемой шифрования:<br><br>$u=\sum\limits_{i=0}^Nu_i=E(p)$<br>$V=\sum\limits_{i=0}^Nv_i=E(px)$<br><br>Во втором раунде участники раскрывают все эти результаты вместе с доказательством нулевого знания, заявляя, что они имеют смысл и  объединяют выявленные акции вместе, используя операцию сложения, определяемую гомоморфной схемой шифрования. У участников есть все параметры, которые были вычислены до сих пор, и у всех игроков они одинаковы.
3. В третьем раунде каждый участник вычисляет случайное целое число $k_i$:<br><br>$k_i\in Z_q$<br><br>И случайное число $c_i$<br><br>$c_i\in [-q^6,q^6]$<br><br>$q$ все время обозначает мощность эллиптической кривой, то есть количество точек на эллиптической кривой.<br>Каждая сторона вычисляет $r_i$ как $g$ в степени $k$  -  но в нашем случае можно просто умножить точку генератора кривой на нее.<br><br>$r_i=g^{k_i}$<br><br>Каждый участник вычисляет параметр $w$, который равен $k$ раз $\rho$ плюс $c$ раз $q$:<br><br>$w_i=E(k_ip+c_iq)$<br><br>$q$ все время - мощность эллиптической кривой, и мы можем вычислить ее, потому что мы используем аддитивно гомоморфное пороговое шифрование.В конце, каждая сторона принимает все эти параметры.
4. В 4 раунде сгенерированные параметры раскрываются вместе с доказательством нулевого знания о том, что они имеют смысл вместе:<br><br>Представляет ZKP, доказывая, что значения $r_i$ и $w_i$ корректны.<br>Имея все эти параметры от всех участников группы, мы можем сложить их вместе, как это сделали в конце 2 раунда. Производится суммирование всех $k$ значений, всех $c$ значений, всех $w$ значений. Более того, мы оцениваем параметр $r$ как сумму всех долей $r_i$ и используем специальную хеш-функцию:<br><br>$k=\sum\limits_{i=0}^Nk_i$<br>$c=\sum\limits_{i=0}^Nc_i$<br>$w=\sum\limits_{i=0}^Nw_i=E(kp+cq)$<br>$r=h(\sum\limits_{i=0}^Nr_i)=H(g^k)$<br><br>На самом деле это та функция, которую мы знаем из стандартного протокола $ECDSA$: это координата точки $x$ по модулю эллиптической кривой. В конце раунда все параметры, перечисленные ниже, являются общими для всех подписавшихся в группе:<br><br>$u=\sum\limits_{i=0}^Nu_i=E(p)$, где $u_i \in Z_q$<br>$v=\sum\limits_{i=0}^Nv_i=E(px)$<br>$k=\sum\limits_{i=0}^Nk_i=E(p)$, где $k_i \in Z_q$<br>$w=\sum\limits_{i=0}^Nw_i=E(kp+cq)$<br>$r=H(\sum\limits_{i=0}^Nr_i)=H(g^k)$<br><br> 
5. Теперь необходимо сделать некоторые дискретные математические преобразования, чтобы создать подпись. Используя все эти параметры, которые до сих пор были получены. Поскольку мы работаем с зашифрованными данными, подпись также будет зашифрована.<br>Первое, что нужно сделать, - это запустить механизм расшифровки порога, чтобы все участники расшифровали параметр $w$ и присвоили это значение $\eta$:<br><br>$\eta=TDec(w)=kp mod q$ - все участники одновременно дешифруют $w$<br><br>Далее вычисляется параметр $\Psi$, который является мультипликативным обратным к $\eta$ по модулю $q$ , а $q$ - это мощность всего эллиптической кривой по времени:<br><br>$\Psi=n^{-1}mod q = k^{-1}p^{-1}$<br><br>Затем, имея $m$ , который является хэшем сообщения, которое мы подписываем (или хэш транзакции), производится оценка подписи с помощью следующего уравнения:<br><br>$\sigma=\Psi\times _e[(m\times _eu)+_eE(r\times_ev)]$<br><br>$c$ - значение, которое мы только что оценили, а $u$ , $r$ и $v$ - параметры, совместно оцененные всеми подписавшими сторонами в предыдущих раундах.<br>Итак, поскольку $u$ является зашифрованным $r$ , а $v$ является зашифрованным $ρ$, умноженным на секретный ключ $ECDSA$, мы можем выполнить следующее преобразование:<br><br>$\sigma=\psi\times_e[E(mp)+_eE(rpx)]$<br><br>И если мы заменим $\Psi$ значением, которое оно представляет, мы получим следующее уравнение:<br><br>$\sigma=(k^{-1}p^{-1})\times_e[E(mp)+eE(rpx)]$<br><br>И, наконец, если мы исключим $ρ$, мы получим это:<br><br>$\sigma=E(k^{-1}(m+xr))$<br><br>А это есть ни что иное, как уравнение для стандартной сигнатуры $ECDSA$, где $k$- криптографически безопасное случайное целое число, $m$ - хеш сообщения, $x$ - наш секретный ключ $ECDSA$, а $r$ - точка, сгенерированная кривой умноженное в $k$ раз по модулю $q$. Так что это уравнение для стандартного протокола $ECDSA$.<br>Так как все уравнения были сделаны на шифротекстах, то и результирующая подпись будет зашифрована:<br><br>$\sigma=E(s)$<br><br> 
6. Теперь необходимо использовать механизм расшифровки пароля, чтобы узнать значение $s$. И расшифрованное значение $s$ и полученное в 4 раунде значение $r$, вместе составляют подпись.