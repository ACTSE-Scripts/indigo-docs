## Tokenizer

util_systems.tokenizer

    class Tokenizer:
	    def get_ranges(self, string)
		    // парсит нахождение ключей, значений, операций и скобок
	    def make_logic_sequence(self, string, ranges)
		    // создает Q объекты из строки, используя зарание известные диапазоны в которых присутсвуют значения
	    def validate_logic(self, logic)
		    // проверяет открытие/закрытие скобок
	    def search_pattern(self, sequence)
		    // ищет подходящие к выполнению патерны последовательностей Q объектов и бинарных операций
	    def executed_pattern(self, sequence, to_process, to_remove)
		    // формирует применяет бинарные операции к Q объектам
	    def execute_logic_sequence(self, sequence)
		    // ищет паттерны для бинарного обединения Q объектов, и объединяет их
	    def parse_string(self, string)
		    // входная точка парсинга, создает логику, валидирует ее, и выполняет преобразование в Q объект для поиска
