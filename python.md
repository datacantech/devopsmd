##переменные
x = 10              # целое число (int)
name = "Alice"      # строка (str)
pi = 3.14           # число с плавающей точкой (float)
is_active = True    # булево значение (bool)
my_list = [1, 2, 3] # список (list), упорядочены (индексированы), доступ по индексу (например, my_list[0]). 
my_dict = {"key": "value"}  # словарь (dict), неупорядочены (доступ по ключу, а не по позиции), доступ по ключу (например, my_dict["key"])
np.array([[1, 2], [3, 4]]) # массив можно заюзать через библиотеку numpy: import numpy as np; my_array = np.array([1, 2, 3]); my_2d_array = np.array([[1, 2], [3, 4]])
##функции
def greet(name):
    return f"Hello, {name}!"  # определение

message = greet("Alice")
print(message)                # вывод: Hello, Alice!

def power(base, exponent=2):
    return base ** exponent   # если нужно по умолчанию
def sum_all(*args):
    return sum(args)
print(sum_all(1, 2, 3))       # если нужно много переменных
##классы
### создание
class Dog:
    # Конструктор (инициализатор)
    def __init__(self, name, age):
        self.name = name  # Атрибут объекта
        self.age = age

    # Метод класса
    def bark(self):
        return "Woof!"

объект(экземпляра класса)
my_dog = Dog("Rex", 5) # определение
print(my_dog.name)     # использование
