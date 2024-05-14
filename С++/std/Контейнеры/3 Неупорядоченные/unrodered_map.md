#unordere_map #not_complited 

set - набор константных ключей
multiset - set, который может храниться несколько одинаковых ключей одновременно
multimap - map, но у которого может хранться несколько значений  с одинаковым ключем

Метод открытой адресации
Метод цепочек - в каждой вершине храним связанный список или вектор

25:00
https://www.youtube.com/watch?v=ZDMoiCeLOEY

double load_factor() - показывает отношение количества лежащих ключей к размерам массива
double max_load_factor() - число, при привышении которого unordered_map решает, что нужно сделать rehash и переложить все элементы
get_max_load_factor() - 
set_max_load_factor() -
