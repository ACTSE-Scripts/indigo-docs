## Contact

    class Contact(Model):

	    first_name = models.CharField(max_length=48)
	    last_name = models.CharField(max_length=48)
	    full_name = models.CharField(max_length=97)

	    emails = models.ManyToManyField(to='Email', blank=True)
	    phones = models.ManyToManyField(to='Phone', blank=True)

	    ghl_contact_id = models.CharField(max_length=32, blank=True)
	    ghl_opportunity_id = models.CharField(max_length=32, blank=True)

	    stage_unit = models.ForeignKey(to='Child', on_delete=models.SET_NULL,
	        blank=True, null=True, related_name='stage_unit')

	    comunication_type = models.ForeignKey(to='ComunicationType',
	        on_delete=models.SET_NULL, blank=True, null=True)

	    test_flag = models.BooleanField(default=False)
	    archive_flag = models.BooleanField(default=False)

	    date_create = models.DateTimeField(auto_now_add=True)
	    date_last_update = models.DateTimeField(auto_now=True)
	    date_last_call = models.DateTimeField(blank=True, default=None, null=True)
	    responsible_user = models.ForeignKey(to='customusers.CustomUser',
	        on_delete=models.SET_NULL, blank=True, null=True, default=None, related_name='owner')

	    nie_odbiera_counter = models.PositiveIntegerField(default=0)
	    zadzwonic_pozniej_counter = models.PositiveIntegerField(default=0)
	    nie_odbiera_counter_date = models.DateTimeField(blank=True, default=None, null=True)
	    zadzwonic_pozniej_counter_date = models.DateTimeField(blank=True, default=None, null=True)

	    last_modified_by = models.ForeignKey(to='customusers.CustomUser',
	        on_delete=models.SET_NULL, blank=True, null=True)

	    track_data = models.ForeignKey(to='TrackData', on_delete=models.SET_NULL, blank=True, null=True)

Модель контакта имеет несколько групп полей
**Информационные**
 - first_name
 - last_name
 - full_name
 - comunication_type
 - responsible_user
 - date_create
 - date_last_update
 - nie_odbiera_counter_date
 - zadzwonic_pozniej_counter_date
 - last_modified_by
 - track_data
 - date_last_call
 
 **Поисковые**
 Поля этой группы нужны в первую очередь для идентификации и поиска контакта
 
 - emails
 - phones
 - nie_odbiera_counter
 - zadzwonic_pozniej_counter
 
 **Системные для связи с GHL**
 Поля для корректной работы с апи GHL
 
 - ghl_contact_id
 - ghl_opportunity_id
 
 **Функциональные**
Поля этой группы выполняют определенную функцию. Они берут активное участие в событиях и изменениях во время событий
 - stage_unit


## События
Для контакта есть два уникальных сигнала

 1. contact_can_be_added_to_ghl
 2. change_contact_method
 
 **change_contact_method**
Это процесс, который запускается при изменении emails или phones (также при создании контакта, когда ему присваиваются его номер и емейл)

**contact_can_be_added_to_ghl**
Процесс, который запускается при готовности к внесению изменений в GHL


## Методы

 1. rest_create
 2. change_relation
 3. fix
 4. tilda
 
 **rest_create**
 Метод реализующий создание контакта.
 Является входной точкой процесса создания **rest**
 **change_relation**
 Метод реализующей изменение номера или емейла
  Является входной точкой процесса создания **change_relation**
  **fix**
  Метод запускающий сигнал **contact_can_be_added_to_ghl** с использованием только номера
  **tilda**
   Метод реализующий создание контакта с формы.
   Является входной точкой процесса создания **rest**
   А также имеет свою дополнительную логику привязки поля track_data

## Процессы
**change_relation**
indigoghlretool.models.adapters.contact

Так как при работе с моделями Email и Phone в отдельности были сложности, было принято решение добавлять/удалять/изменять их через модель контакта к которому они привязаны. Это поможет упростить отслеживание этих событий, а также создать единое событие (я также могу ошибаться, однако для меня это был самый явный и прямой способ создать лишь один способ что-то сделать).

Данный процесс описывает что должно выполняться при изменении, создании, удалении Email или Phone.
Для каждой модели есть своя версия реализации данного процесса, однако они абстрагированы и управляются через входные параметры.

**Входные параметры**

 - request - POST запрос с полями 
	 - id:int|str - объект для  удаления или изменения
	 - phone:str - явное указание поля номер
	 - email:str - явное указание поля емейл
	 - phone_is_main: bool
	 - email_is_main: bool
	 - value:str - значение которое применяется как входное для модели указанной	через поле model
	 - is_main: bool
 - action:str - событие которое должно быть выполненным (add|remove|change)
 - model:str - тип модели (phone|email)
 - obj:Contact - контакт
 - init:bool - нужно ли вызывать событие **change_contact_method**

**contact_can_be_added_to_ghl**
indigoghlretool.models.adapters.contact

Процесс создания контакта из входных данных в виде запроса.
- request - POST запрос с полями 
	 - phone:str - явное указание поля номер
	 - email:str - явное указание поля емейл
	 - phone_is_main: bool = True
	 - email_is_main: bool = True
	 - first_name: str
	 - last_name: str
	 - comunication_type: int|str - id объекта ComunicationType
	 - last_modified_by: int|str - id объекта CustomUser

В процессе создания также используется процесс **change_relation**, а такде валидация входных данных.
В случае если валидация входных данных успешна процесс вернет следующий результат:

    answer = {
        'status':'success',
        'id': int,
        'phone_add': {
	        'status': str,
	        'obj':int
        },
        'email_add': {
	        'status': str, 
	        'obj':int
	    }
    }
   
   Иначе
   
    answer = {
        'status':'invalid',
        'id': None,
        'phone_add': "undefined",
        'email_add': "undefined",
        'validation': {
	        'validate':bool,
	        'messages':[str],
	        'state':[str],
	        'contact_id': int}
    }

## Сигналы
 indigoghlretool.models.handlers.contact

**change_contact_method**
описан процесс обновления в GHL только номера или емейла
**contact_can_be_added_to_ghl**
описан процесс создания контакта и сделки, а также присвоение полей для связи с GHL
**pre_delete**
удаление контакта из GHL, а также удаление track_data
**pre_save**
Есть несколько событий завязанных на изменениях

Любое изменение в имени или фамилии вызовет обновление полного имени и контакта + сделки в GHL

Если stage_unit станет пустым (***C1***)
при  наличии открытого ребенка - установит новый stage_unit и укажет на явное изменение поля stage_unit

Любое изменение stage_unit или явное изменение stage_unit (***C2***)
Обновление сделки в GHL

любое изменение в поле nie_odbiera_counter
обновление поля nie_odbiera_counter_date

любое изменение в поле zadzwonic_pozniej_counter
 обновление поля zadzwonic_pozniej_counter_date

изменение поля track_data с пустоты
создание заметки в GHL

## Немного о Phone
В обработку номеров входит нормализация входного параметра
Это значит что при передаче номера +1 234 567 890 это будет то же самое что и другие варианты
+1 (234) 567 890
+1 (234) 567-890
+1 234-567-890
P.S. но я вообще этим не пользуюсь)) ибо я мог где-то что-то упустить)