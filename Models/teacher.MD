## Teacher

    class Teacher(Model):
        name = models.CharField('Name', max_length=128)
        email = models.CharField('Email', max_length=128)
        zoom_link = models.CharField('Zoom Link', max_length=256)
        zoom_id = models.CharField('Zoom ID', max_length=128)
        zoom_pass = models.CharField('Zoom PASS', max_length=128)
    
        location = models.ForeignKey(to='Location', on_delete=models.CASCADE, blank=True, null=True)
        lecture_type = models.ForeignKey(to='LectureType', on_delete=models.CASCADE, blank=True, null=True)
    
        matrix = models.ForeignKey(to='AvailabilityMatrix', on_delete=models.CASCADE, blank=True, null=True)
    
        next_time_change = models.DateField(default=None, blank=True, null=True)
        is_active = models.BooleanField(default=True)

## Сигналы
**pre_save**
если не создано
создает кастомного пользователя, генерирует пароль, отправляет письмо с данными
иначе следит за активностью и устанавливает аналогичную для кастомного пользователя
**post_save**
если создано, создает матрицу
**pre_delete**
удаляет кастомного пользователя
**post_delete**
удаляет матрицу
## Методы
**get_for_lecture**
**get_for_group**
**get_teachers**
роутер для других методов

## Процессы
Задача, имея ид лекции или группы подобрать всех свободных преподавателей которые могут вести это занятие

Общий алгоритм
1. отсортировать преподавателей по ключевым признакам: локация и тип занятий
2. определить другие занятия которые идут в день проведения этого занятия
3. определить сет занимаемых занятием ячеек времени
4. определить сет занятых другими занятиями ячеек времени
5. исключить его из сета ячеек преподавателя
6. проверить полное вхождение сета ячеек лекции в сет оставшихся ячеек преподавателя

**get_for_lecture**
поправки
занятия для расчета недоступного времени преподавателя - лекции и группы

**get_for_group**
поправки
занятия для расчета недоступного времени преподавателя - группы

---
Стоит помнить что мы делаем шаги 2-6 для каждого преподавателя полученного в результате шага 1

Действия используемые для алгоритма
TeacherFilter - формирует объект фильтрации учителей с оптимизацией запроса

getObjectPrimitives( [Group|TestLecture] )
splitToPrimitives(Teacher)
&
