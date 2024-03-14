## Управление сигналами и собітиями в них

util_systems.advanced_model_reciver

## AMR (advanced model receiver)

Класс декоратор для классов моделей
Позволяет быстро и просто прикрепить нужные сигналы к модели или создать новые (также быстро их вызывать)

Регистрация сигналов
@AMD.register({  })
{
	"название сигнала":функция его обработки
}

Создание своего сигнала
AMD.create_new_signal("название сигнала")

Вызов сигнала
AMR.signals.название сигнала(sender, instance, *args, **kwargs)

## Watcher (наблюдение за событиями)

Упрощает работу с событиями, так как не нужно писать много условий

Watcher.start_session(sender, instance, required=None, origin=None)
Создает сессию наблюдателя в instance, принимает sender, instance объект модели, required список обязательных полей (полей которые не равны None, 0, ''), объект origin

Использование
Watcher.start_session(sender, instance)
instance.watcher.
-- changeTo(key, value, _from=NoneValue)
	возвращает True, когда поле key принимает значение value, если указан _from, то прошлое значение должно равняться _from
-- anyChange(key, _from=NoneValue)
	возвращает True, когда поле key меняет значение, если указан _from, то прошлое значение должно равняться _from
-- when(key, value)
	возвращает True, когда поле key имеет значение value
	
	