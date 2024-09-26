---
title: Giới thiệu về LINQ
sidebar_position: 0
---

# Giới thiệu về LINQ

LINQ (Language-Integrated Query) là một tính năng mạnh mẽ của C# cho phép bạn thực hiện các truy vấn dữ liệu một cách nhất quán và dễ hiểu. LINQ cung cấp một cú pháp truy vấn thống nhất để làm việc với các nguồn dữ liệu khác nhau như các bộ sưu tập (collections), cơ sở dữ liệu, XML, và nhiều nguồn khác. LINQ tích hợp truy vấn dữ liệu vào trong chính ngôn ngữ lập trình, giúp việc thao tác dữ liệu trở nên trực quan và dễ bảo trì hơn.

## Ví dụ cơ bản với LINQ

Dưới đây là ví dụ cơ bản về cách sử dụng LINQ để truy vấn một danh sách các số nguyên:

```csharp title="C#"
using System.Linq;
int[] numbers = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

// Sử dụng LINQ để lọc các số chẵn
IEnumerable<int> evenNumbers = from number in numbers
                               where number % 2 == 0
                               select number;

foreach (var num in evenNumbers)
{
    Console.WriteLine(num); // Kết quả: 2, 4, 6, 8, 10
}
```

## Tài liệu tham khảo
1. [Microsoft Docs - LINQ Overview](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/linq/)
2. [Wikipedia - LINQ](https://en.wikipedia.org/wiki/Language_Integrated_Query)
3. [Scott Allen - LINQ in Action](https://www.manning.com/books/linq-in-action)
