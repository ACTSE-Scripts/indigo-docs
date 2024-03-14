   ## Child
   
    class Child(Model):
    
        name = models.CharField(max_length=48, blank=True)
        age = models.FloatField(blank=True,
            validators=[MinValueValidator(0.0), MaxValueValidator(20.0)])
    
        last_modified_by = models.ForeignKey(to='customusers.CustomUser',
            on_delete=models.SET_NULL, blank=True, null=True)
    
        parent = models.ForeignKey(to='Contact', on_delete=models.CASCADE,
            blank=True, null=True)
        ghl_contact_id = models.CharField(max_length=32, blank=True, default='')
        ghl_opportunity_id = models.CharField(max_length=32, blank=True, default='')
        ghl_note_id = models.CharField(max_length=32, blank=True)
        responsible_user = models.ForeignKey(to='customusers.CustomUser',
            on_delete=models.SET_NULL, blank=True, null=True, related_name='my_childs')
    
        contact_registration_comment = models.TextField(blank=True)
        sales_comment = models.TextField(blank=True)
        sales_test_lecture_comment = models.TextField(blank=True)
        teacher_test_lecture_comment = models.TextField(blank=True)
    
        sales_group_comment_1 = models.TextField(blank=True)
        sales_group_comment_2 = models.TextField(blank=True)
        sales_group_comment_3 = models.TextField(blank=True)
        sales_group_comment_4 = models.TextField(blank=True)
        sales_group_comment_5 = models.TextField(blank=True)
        contacted = models.BooleanField(default=False)
    
        status = models.CharField(max_length=4, choices=[
            ['open', 'Open'],
            ['won', 'Won'],
            ['lost', 'Lost'],
        ], default='Open')
        stage = models.CharField('Stage', choices=STAGES_NAME, max_length=32, default='NOWE ZGŁOSZENIE')
    
        test_lecture = models.ForeignKey(to='TestLecture', on_delete=models.SET_NULL, blank=True, null=True)
        group = models.ForeignKey(to='Group', on_delete=models.SET_NULL, blank=True, null=True)
    
        location = models.ForeignKey(to='Location', on_delete=models.SET_NULL, blank=True, null=True)
        lecture_type = models.ManyToManyField(to='GroupLectureType', blank=True)
    
        lost_reason = models.ForeignKey(to='LostReason', on_delete=models.SET_NULL, blank=True, null=True)
    
        user_days = models.TextField(blank=True)
        user_types = models.TextField(blank=True)
    
        payment_lecture = models.CharField('Lecture paid', max_length=32, blank=True)
        payment_tools = models.CharField('The set is paid', max_length=32, blank=True)
        tools_sent = models.CharField('The set has been sent', max_length=32, blank=True)
    

        matrix = models.ForeignKey(to='AvailabilityMatrix', on_delete=models.SET_NULL, blank=True, null=True)

        date_last_update = models.DateTimeField(auto_now=True)
        dates = models.ForeignKey(to='StageDate', on_delete=models.SET_NULL, blank=True, null=True)

        agreement_flag = models.BooleanField(default=False)

        extend_time_flag = models.BooleanField(default=False)

        agreed_participate_flag  = models.BooleanField(default=False)

        reserved_flag = models.BooleanField(default=False)

        test_flag = models.BooleanField(default=False)

        archive_flag = models.BooleanField(default=False)
        archive_data = models.TextField(blank=True)


 **Системные для связи с GHL**
 Поля для корректной работы с апи GHL
 
 - ghl_contact_id
 - ghl_opportunity_id
 - ghl_note_id

 **Функциональные**
 - parent
 - responsible_user
 - stage
 - test_lecture
 - group
 - location
 - lecture_type
 - payment_lecture
 - payment_tools
 - tools_sent
 - agreement_flag
 - matrix

*все остальные поля можно считать функциональными (хотя это не всегда будет так)), хотя мне просто влом их всех перечислять, сами прочитаете))

## Методы
**run_create**
Метод создания ребенка, параметры аналогичны базовым для создания модели из DRF

*все остальные методы отвечают за отправку емейлов. В них нет ничего сложного ))
Хотя отмечу что скрипт отправки емейлов на GHL не будет включен в документацию.

GHL contact имеет поле со значением 1. Оно ничего не делает, однако если изменить значение, то запуститься скрипт который отправит емейл в зависимости от значения, а потом снова установит значение 1. Вот и все)

## Сигналы
 indigoghlretool.models.handlers.child

**pre_save**
Основная логика
Создание зависимых объектов, если объект создается

Процесс смены ответственного (***C3***)
При любом изменение ответственного, фильтруем TransferContract
Если их нет, создаем НЕ использованную запись ожидающую подтверждения, останавливаем прошлого ответственного
Если есть запись, ничего не делаем 

Если объект создается и идет смена родителя с None
Присваиваем его инфу с ghl
создаем заметку

Если есть родитель
Проверяем любые изменения в стадиях и логируем дату, если это смена на проигрыш
устанавливаем статус и отправляем письмо в зависимости от указанной причины проигрыша
Если это победа, меняем статус

Проверяем изменения в лекциях и группах и переносим ребенка на нужную стадию (***C4***)

Проверяем изменения в ключевых для лекций и груп стадиях
отправляем письмо

При любых изменениях в стадиях
Обновляем сделку родителя, если ребенок является stage_unit своего родителя
**post_save**
Смена stage_unit

Любые изменения в поле parent и если ребенок был создан
проверит есть ли у родителя stage_unit, если нет то сам им станет вызвав реакцию **C2**

Если есть родитель
И стадия равна won или lost
Я он является stage_unit для родителя
Уберет себя с роли stage_unit вызвав реакцию **C1**

Если есть родитель
обновляем заметку
**pre_delete**
Можно удалять)
**post_delete**
Удалит зависимые объекты

Если есть ghl_note_id, удалит заметку
Если есть родитель и у него есть пусто stage_unit
Отфильтрует детей которіе подходят на роль stage_unit и при наличии таковых, установит первого попавшегося вызвав реакцию **C2**

Немного объясню этот момент. Так как эот ребенок был stage_unit, то после его удаления у его родителя будет установлен stage_unit как None, однако если у родителя есть другие дети, то они должны занять место stage_unit

