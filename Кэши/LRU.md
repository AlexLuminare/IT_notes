**LRU (Least Recently Used)** — это алгоритм кэширования, при котором вытесняются данные, которые были использованы дольше всего. Говоря простым языком, это означает, что когда кэш заполняется, из него удаляются элементы, к которым обращались реже всего.

**Реализация на Go:**

Для реализации LRU кэша на Go мы будем использовать комбинацию двух структур данных:

- **Двусвязный список:** Для отслеживания порядка доступа к элементам.
- **Карта (map):** Для быстрого поиска элементов по ключу.

```go
package lru

type Cache struct {
        maxItems int
        items    map[string]*list.Element
        ll       *list.List
}

type entry struct {
        key   string
        value interface{}
}

func New(maxItems int) *Cache {
        return &Cache{
                maxItems: maxItems,
                items:    make(map[string]*list.Element),
                ll:       list.New(),
        }
}

func (c *Cache) Add(key string, value interface{}) {
        if ee, ok := c.items[key]; ok {
                c.ll.MoveToFront(ee)
                ee.Value.(*entry).value = value
                return
        }
        ee := c.ll.PushFront(&entry{key, value})
        c.items[key] = ee
        if c.ll.Len() > c.maxItems {
                el := c.ll.Back()
                if el != nil {
                    c.ll.Remove(el)
                    delete(c.items, el.Value.(*entry).key)
                }
        }
}

func (c *Cache) Get(key string) (value interface{}, ok bool) {
        if ee, ok := c.items[key]; ok {
                c.ll.MoveToFront(ee)
                return ee.Value.(*entry).value, true
        }
        return
}	
```
**Объяснение кода:**

- **Структура `Cache`:** Содержит максимальное количество элементов, карту для быстрого поиска и двусвязный список для отслеживания порядка.
- **Структура `entry`:** Представляет собой элемент кэша и хранит ключ и значение.
- **Метод `New`:** Создает новый экземпляр кэша с заданным максимальным размером.
- **Метод `Add`:** Добавляет элемент в кэш. Если элемент уже существует, обновляется его значение и перемещается в начало списка. Если кэш заполнен, удаляется последний элемент.
- **Метод `Get`:** Получает значение по ключу. Если элемент найден, перемещается в начало списка.