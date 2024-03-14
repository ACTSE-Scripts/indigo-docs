
## Config Extended (CFEX)

util_systems.cfex

    class CFEX:
		def __init__(self, cfex_path, context={})
		def _read(self)
			// получение данных файла в необработаном виде
			
		def load(self)
			// точка входа, загрузка и обработка данных, возвращает словарь с конфигурациями
			
		def process(self)
			// основная функция обработки строк, в ходе работы заполняет конфигурацию
			
		def get_lines(self)
			// превращает сырые данные в итерируемый набор строк
			
		def getter(self, lines)
			// функция определения типа строки, принимает решения как должны быть интерпретированы те или иные строки
			
		def get_var(self, line)
			// функция обработки переменной, и ее запись в соответствующий блок
			
		def get_section(self)
			// получение секции для чтения
			
		def set_section(self, line, _type)
			// создание секции для записи
			
		def get_value_by_link(self, key, link)
			// установка значений вида ссылки на словарь или секцию
			
		def set_value(self, v)
			// обработка переменных внутри указываемых значений
			
		def check_items(self, key, ref, value)
			//	проверка типа данных
			
		def include(self, link)
			// функция зпгрузки контекста с другого файла
			
		def clean_ref(self)
			// очистка ссылки
			
		def clean_private(self)
			// удаление приватных переменных

