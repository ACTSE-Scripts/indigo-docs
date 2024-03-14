## Group

    class Group(Model):
    
        date_create = models.DateTimeField(auto_now_add=True)
        name = models.CharField(max_length=64)
    
        teacher = models.ForeignKey(to='Teacher', on_delete=models.SET_NULL, blank=True, null=True)
        opcode = models.CharField(max_length=3, blank=True)
        sequence_length = models.PositiveIntegerField(default=4)
    
        age_range = models.CharField('Age', max_length=12, default='')
        location = models.ForeignKey(to='Location', on_delete=models.CASCADE)
        lecture_type = models.ForeignKey(to='GroupLectureType', on_delete=models.PROTECT)
    
        start_date = models.DateField(blank=True, default=None, null=True)
    
        group_started_flag = models.BooleanField(default=False)
        extend_group_flag = models.BooleanField(default=False)
        finished_flag = models.BooleanField(default=False)

**Методы**
**start_group**
перемещает всех детей на этап WON, и устанавливает соответсвующий флаг

## TestLecture

    class TestLecture(Model):
	    date_create = models.DateTimeField(auto_now_add=True)
    name = models.CharField(max_length=64)

	    teacher = models.ForeignKey(to='Teacher', on_delete=models.SET_NULL, blank=True, null=True)
	    opcode = models.CharField(max_length=3, blank=True)
	    sequence_length = models.PositiveIntegerField(default=4)

	    age_range = models.CharField('Age', max_length=12, default='')
	    location = models.ForeignKey(to='Location', on_delete=models.CASCADE)
	    lecture_type = models.ForeignKey(to='GroupLectureType', on_delete=models.PROTECT)

	    start_date = models.DateField(blank=True, default=None, null=True)

Очень близкие модели так как имеют почти идентичную структуру

## Сигналы
**pre_delete**
удаляет экземпляр из ребенка вызывая реакцию **C4**
## Методы
**start_date_full**
создает datetime на основе date и поля с опкодом
использует get_time_for_lecture(opcode, length)
**get_calendar**
использование LectureCalendar и GroupCalendar
**create_from_calendar**
процесс получения данных включая преобразования времени в опкод и создания экземпляра с использованием этих данных



---
Календари - это сводная таблица доступности всех учеников или учителей
Задача, посчитать сколько учителей могут потратить N своих последовательных опкодов начиная с опкода S
Задача применима и для детей, но без учета вычитаний опкодов

Алгоритм
1. Получить ВСЕХ учителей
2. Получить их примитивы
3. Получить лекции и группы для учителей
4. Получить примитивы для исключения на основе лекций и груп
5. Исключить их из примитивов учителя
6. Для каждого примитива вычислить наличие последовательности длиной в N
7. Если она есть, заполнить ячейку примитива инфой о преподавателе
8. Вывести статистику
Для детей убираем шаги 3-5
для вычисления наличия последовательности используем функцию findSequnces

# LectureCalendar
indigoghlretool.models.adapters.calendars

Особенности

 - поиcк идет только по учителям, с учетом груп и лекций
 - длина последовательности всегда 4 (1 час)

# GroupCalendar
indigoghlretool.models.adapters.calendars

Особенности

 - поиcк идет как по учителям так и по детям, с учетом груп
 - длина последовательности зависит от возраста
