# Курсовая работа по модулю "Мобильная разработка в 1С"

### Цель

Реализовать мобильное приложение для курьера, работающее совместно с конфигурацией Торговое предприятие

### Чеклист готовности к курсовой работе

- [ ] Подготовить информационную базу, полученную по итогу выполнения [домашнего задания к занятию 15-3](homework-15-3.md).
- [ ] Подготовить информационную базу, полученную по итогу выполнения [домашнего задания к занятию 12-3](../BSP/homework-12-3.md) или более позднего.

### Описание задачи

Реализовать мобильное приложение для курьеров со следующей функциональностью:
1. Получение списка своих заказов из центральной базы
2. Возможность установки статуса заказам из перечня: Запланирован, Взят в работу, Отклонен курьером, Отклонен клиентом. При остановке статуса Курьер должен указать комментарий.
3. Прикрепление к заказу фото с информацией о выполненной доставке
4. Транспорт статуса заказа, комментария курьера и фото в десктопную базу при очередном сеансе обмена данными
5. Возможность выполнения звонков клиентам, которым запланирована доставка с помощью встроенных средств телефонии

### Инструкция к заданию 

1. Решите описанные задачи в конфигураторе.
2. Протестируйте решение в пользовательском режиме.
3. В личном кабинете Нетологии отправьте на проверку ссылку на файл dt с выгрузкой десктопной информационной базы и файл cf с конфигурацией мобильного приложения.

### Процесс выполнения

1. В мобильном приложении из [домашнего задания к занятию 15-1](homework-15-1.md) создать документ Доставка с реквизитами Контрагент, АдресДоставки, Комментарий и табличная часть Товары с колонками Номенклатура и Количество.
2. Реализуйте хранения Номенклатуры и Контрагентов в отдельных справочниках.
3. Добавьте в документ Доставка реквизит Фото, реализуйте возможность создания фото с помощью камеры мобильного устройства и вывод сделанного фото на форму.
5. В десктопном приложении на базе БСП полученную из [домашнего задания к занятию 12-3](../BSP/homework-12-3.md) или более позднего реализуйте план обмена для регистрации изменений документа Доставка с выключенной авторегистрацией. У плана обмена должен быть реквизит Пользователь. Реализуйте регистрацию документа Доставка на всех узлах плана обмена, у которых значение реквизита Пользователь заполнено и соответствует значению реквизита Ответственный документа.

<details>
  <summary>Пример кода регистрации на узле плана обмена</summary>

```bsl
	Запрос = Новый Запрос;
	Запрос.Текст = "ВЫБРАТЬ
	               |	ОбменСМобильнымУстройством.Ссылка КАК Ссылка
	               |ИЗ
	               |	ПланОбмена.ОбменСМобильнымУстройством КАК ОбменСМобильнымУстройством
	               |ГДЕ
	               |	ОбменСМобильнымУстройством.Пользователь = &Пользователь
	               |	И НЕ ОбменСМобильнымУстройством.ПометкаУдаления";
	
	Запрос.УстановитьПараметр("Пользователь", Ответственный);
	
	Выборка = Запрос.Выполнить().Выбрать();
	
	Пока Выборка.Следующий() Цикл
		ОбменДанными.Получатели.Добавить(Выборка.Ссылка);
	КонецЦикла; 
```
    
</details>

6. Реализуйте HTTP-сервис для обмена с мобильным приложением. Создайте роль для работы с сервисом и создайте пользователя с логином **Курьер**. Сервис должен:
    - При первом вызове инициализировать необходимые узлы планов обмена и выполнять первичную регистрацию объектов - Документов доставка, в которых реквизит Ответственный соответствует текущему пользователю;
    - В качестве ответа передавать сериализованные данные документов, зарегистрированных на узле плана обмена, соответствующих устройству.

<details>
  <summary>Пример кода инициализации узла плана обмена</summary>

```bsl
	УстановитьПривилегированныйРежим(Истина);
	
	ИдентификаторМобильного = Запрос.Заголовки.Получить("X-Mobile-ID");
	
	Узел = ПланыОбмена.ОбменСМобильнымУстройством.НайтиПоКоду(ИдентификаторМобильного);
	Если Не ЗначениеЗаполнено(Узел) Тогда
		УзелОбъект = ПланыОбмена.ОбменСМобильнымУстройством.СоздатьУзел();
		УзелОбъект.Код = ИдентификаторМобильного;
		УзелОбъект.Наименование = ИдентификаторМобильного;
		УзелОбъект.Пользователь = Пользователи.ТекущийПользователь();
		УзелОбъект.Записать();		
		Узел = УзелОбъект.Ссылка;
		ВыполнитьПервичнуюРегистрацию(Узел);
	КонецЕсли;
	
	ТекущийУзел = ПланыОбмена.ОбменСМобильнымУстройством.ЭтотУзел();
	Если Не ЗначениеЗаполнено(ТекущийУзел.Код) Тогда
		УзелОбъект = ТекущийУзел.ПолучитьОбъект();
		УзелОбъект.Код = "main";
		УзелОбъект.Наименование = "Центральный узел";
		УзелОбъект.Записать();
	КонецЕсли;

    // Получение зарегистрированных данных

    // Сериализация зарегистрированных данных

	Ответ = Новый HTTPСервисОтвет(200);
	Ответ.УстановитьТелоИзСтроки(ТелоОтвета);	
	Возврат Ответ;
```

</details>

<details>
  <summary>Пример кода начальное регистрации данных</summary>

```bsl
	Запрос = Новый Запрос;
	// Установка текста запроса
	Запрос.УстановитьПараметр("Ответственный", Пользователи.ТекущийПользователь());
	
	Выборка = Запрос.Выполнить().Выбрать();  
	
	ДанныеКРегистрации = Новый Массив;
	
	Пока Выборка.Следующий() Цикл
		ДанныеКРегистрации.Добавить(Выборка.Ссылка);
	КонецЦикла;
	
	ПланыОбмена.ЗарегистрироватьИзменения(Узел, ДанныеКРегистрации);
```
    
</details>


7. Реализуйте на стороне мобильного приложения механизм синхронизации данных:
    - Создайте константы для хранения адреса подключения к базе, логина и пароля;
    - Создайте обработку с командой для выполнения синхронизации данных;
    - В рамках синхронизации данных выполните вызов сервиса, созданного на предыдущем шаге и создайте документы, которые получены из центральной базы.

<details>
  <summary>Пример кода выполнения синхронизации</summary>

```bsl
	СистемнаяИнформация = Новый СистемнаяИнформация;
	ИдентификаторМобильного = СистемнаяИнформация.ИдентификаторКлиента;

	Узел = ПланыОбмена.ОбменСМобильнымУстройством.НайтиПоКоду("main");
	Если Не ЗначениеЗаполнено(Узел) Тогда
		УзелОбъект = ПланыОбмена.ОбменСМобильнымУстройством.СоздатьУзел();
		УзелОбъект.Код = "main";
		УзелОбъект.Наименование = "Центральный узел";
		УзелОбъект.Записать();		
		Узел = УзелОбъект.Ссылка;
	КонецЕсли;
	
	ТекущийУзел = ПланыОбмена.ОбменСМобильнымУстройством.ЭтотУзел();
	Если Не ЗначениеЗаполнено(ТекущийУзел.Код) Тогда
		УзелОбъект = ТекущийУзел.ПолучитьОбъект();
		УзелОбъект.Код = ИдентификаторМобильного;
		УзелОбъект.Наименование = ИдентификаторМобильного;
		УзелОбъект.Записать();
	КонецЕсли;
	
	НастройкиСоединения = НастройкиСоединения();

	// Функцию СоединениеСЦентральнойБазой() необходимо реализовать самостоятельно
	Соединение = СоединениеСЦентральнойБазой(НастройкиСоединения);
	
	// Выполнение HTTP-запроса к центральной базе
	
	Если Ответ.КодСостояния <> 200 Тогда
		ВызватьИсключение "Ошибка проверки соединения";
	КонецЕсли;
	
	ТекстОтвета = Ответ.ПолучитьТелоКакСтроку();
	
	// Чтение ответа из центральной базе
	
	КоличествоСозданных = 0;
	КоличествоОбновленных = 0;
	
	// Создание и обновление объектов с накоплением информации о количестве в счетчиках
	
	ТекстСообщения = СтрШаблон("Обмен выполнен. Создано %1, обновлено %2",
		КоличествоСозданных,
		КоличествоОбновленных);
		
	Сообщить(ТекстСообщения);
```
    
</details>

8. Протестируйте обмен. В результате запуска обмена документа Доставка должны попадать в мобильное приложение.
9. В десктопном приложении реализовать возможность хранения Фото в документе Доставка. Вывести изображение на форму документа.
10. В мобильном приложении создать план обмена и реализовать регистрацию документов доставка при записи.
11. На стороне десктопного приложения доработать сервис для обмена с мобильным приложением методом по приему данных фотографии.
12. На стороне мобильного приложения доработать команду Синхронизации данных - после получения данных из десктопной информационной базы выгружать фотографии из зарегистрированных на узле плана обмена документов.

### Самопроверка

1. Создайте в десктопном приложении 2 пользователей с ролью Курьер;
2. Создайте несколько заказов, распределите их между 2 курьерами;
3. Настройте в мобильном приложении подключение к центральной базе, убедитесь, что при синхронизации в приложение поступили только заказы выбранного курьера;
4. Убедитесь, что есть возможность набрать номер клиента с помощью средств телефонии;
5. Убедитесь, что есть возможность изменить статус Доставки и указать комментарий;
6. Убедитесь, что есть возможность прикрепить фото к Доставке;
7. Проверьте, что при очередной синхронизации статусы, комментарии и фото поступают в центральную базу и есть возможность их посмотреть.

### Критерии оценки

1. Зачёт — выполнены все задания курсовой работы, в выполненных заданиях нет противоречий и нарушения логики.
2. На доработку — задание выполнено частично или не выполнено, в логике выполнения заданий есть противоречия, существенные недостатки.

Все задачи обязательны к выполнению. Пожалуйста, присылайте на проверку все задачи сразу.

Любые вопросы по решению задач задавайте в чате учебной группы.

*Примерное время выполнения: до 300 минут*
