---
{"dg-publish":true,"permalink":"/digital-garden/code/code-refactoring/"}
---



```
using SOLID_AND_OOP;
using System;
using System.Collections.Generic;
using System.Globalization;

namespace SOLID_AND_OOP
{

    // Класс нашего объекта с полями
    public class Item
    {
        public string Name { get; set; }
        public string Price { get; set; }
        public string Date { get; set; }
    }

    // Интерфейс для реализации нашей сортировки
    public interface ISortingStrategy
    {
        List<Item> Sort(List<Item> items);
    }

    // Интерфейс для реализации нашей конвертации
    public interface IPriceConverter
    {
        int ConvertPrice(string value);
    }

    // Сортировка по имени
    public class NameSortingStrategy : ISortingStrategy
    {
        public List<Item> Sort(List<Item> items)
        {
            items.Sort((x, y) => string.Compare(x.Name.ToLower(), y.Name.ToLower()));
            return items;
        }
    }

    // Сортировка по цене
    public class PriceSortingStrategy : ISortingStrategy
    {
        private readonly IPriceConverter _priceConverter;

        public PriceSortingStrategy(IPriceConverter priceConverter)
        {
            _priceConverter = priceConverter;
        }

        public List<Item> Sort(List<Item> items)
        {
            items.Sort((x, y) => _priceConverter.ConvertPrice(x.Price).CompareTo(_priceConverter.ConvertPrice(y.Price)));
            return items;
        }
    }

    // Сортировка по дате
    public class DateSortingStrategy : ISortingStrategy
    {
        public List<Item> Sort(List<Item> items)
        {
            items.Sort((x, y) =>
                DateTime.ParseExact(x.Date, "dd.MM.yyyy", CultureInfo.InvariantCulture)
                .CompareTo(DateTime.ParseExact(y.Date, "dd.MM.yyyy", CultureInfo.InvariantCulture))
            );
            return items;
        }
    }

    // Конвертация цены учитывая валюту
    public class CurrencyPriceConverter : IPriceConverter
    {
        private readonly Dictionary<string, int> _rates;

        public CurrencyPriceConverter(Dictionary<string, int> rates)
        {
            _rates = rates;
        }

        public int ConvertPrice(string value)
        {
            string currency;
            int amount;

            if (value.StartsWith("$"))
            {
                currency = "USD";
                amount = int.Parse(value.Substring(1));
            }
            
            else if (value.StartsWith("Rub"))
            {
                currency = "Rub";
                amount = int.Parse(value.Substring(3));
            }
            else
            {
                throw new ArgumentException("Неподдерживаемая валюта");
            }

            // Применяем соответствующий курс валют
            if (!_rates.ContainsKey(currency))
            {
                throw new ArgumentException("Неподдерживаемая валюта");
            }

            return amount * _rates[currency];
        }

    }

    public class SortingStrategyFactory
    {
        private readonly IPriceConverter _priceConverter;

        public SortingStrategyFactory(IPriceConverter priceConverter)
        {
            _priceConverter = priceConverter;
        }

        public ISortingStrategy CreateStrategy(string sortingParameter)
        {
            return sortingParameter switch
            {
                "name" => new NameSortingStrategy(),
                "price" => new PriceSortingStrategy(_priceConverter),
                "date" => new DateSortingStrategy(),
                _ => throw new ArgumentException("Неизвестный параметр сортировки")
            };
        }
    }

    public class ItemManager
    {
        private readonly ISortingStrategy _sortingStrategy;

        public ItemManager(ISortingStrategy sortingStrategy)
        {
            _sortingStrategy = sortingStrategy;
        }

        public List<Item> SortItems(List<Item> items)
        {
            return _sortingStrategy.Sort(items);
        }
    }

}

class Program
{
    static void Main()
    {
        // Список товаров
        var items = new List<Item>
        {
            new Item { Name = "keyboard", Price = "$30", Date = "10.08.2024" },
            new Item { Name = "клавиатура", Price = "Rub4000", Date = "11.08.2024" },
            new Item { Name = "Монитор", Price = "Rub40000", Date = "11.07.2024" },
            new Item { Name = "Монитор 2", Price = "$400", Date = "11.07.2024" }
        };

        // Курс валют к рублю
        var rates = new Dictionary<string, int>
        {
            { "USD", 90 },
            { "Rub", 1 }
        };

        // Создаем сортировки
        var priceConverter = new CurrencyPriceConverter(rates);
        var sortingFactory = new SortingStrategyFactory(priceConverter);

        // Выбор критерия сортировки
        Console.WriteLine("Выберите критерий сортировки: 'name', 'price' или 'date'");
        var sortingParameter = Console.ReadLine().ToLower();
        var sortingStrategy = sortingFactory.CreateStrategy(sortingParameter);


        // Менеджер для сортировки
        var manager = new ItemManager(sortingStrategy);
        var sortedItems = manager.SortItems(items);

        // Выводим отсортированные результаты
        Console.WriteLine($"Сортировка по {sortingParameter}:");
        foreach (var item in sortedItems)
        {
            Console.WriteLine($"{item.Name} - {item.Price} - {item.Date}");
        }
    }
}
```