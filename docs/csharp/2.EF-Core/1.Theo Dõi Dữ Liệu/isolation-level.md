---
title: Sử dụng Isolation Level 
sidebar_position: 2
---
# Sử dụng Isolation Level trong EF-Core

**Transaction** (giao dịch) trong Entity Framework Core đảm bảo rằng một loạt các thao tác cơ sở dữ liệu được thực thi nguyên tử, tức là tất cả các thao tác đều thành công hoặc tất cả đều thất bại. Điều này giúp đảm bảo tính toàn vẹn của dữ liệu. Bên cạnh đó, **Isolation Level** (mức độ cô lập - **I** trong thuộc tính **ACID**) xác định cách các giao dịch có thể ảnh hưởng đến nhau, đặc biệt là cách các thao tác đọc và ghi dữ liệu có thể nhìn thấy nhau.

## Isolation Levels trong SQL Server

Dưới đây là một số mức độ cô lập phổ biến:

- **Read Uncommitted**: Giao dịch có thể đọc dữ liệu chưa được cam kết từ các giao dịch khác, có thể gây ra hiện tượng "dirty read".
- **Read Committed**: Mặc định của SQL Server. Giao dịch chỉ có thể đọc dữ liệu đã được cam kết.
- **Repeatable Read**: Đảm bảo rằng dữ liệu đã đọc sẽ không bị thay đổi bởi các giao dịch khác trong suốt thời gian giao dịch này đang hoạt động.
- **Serializable**: Mức độ cô lập cao nhất, ngăn không cho các giao dịch khác đọc hoặc ghi dữ liệu mà giao dịch này đang làm việc.
- **Snapshot**: Giao dịch đọc một ảnh chụp nhanh của dữ liệu tại thời điểm giao dịch bắt đầu, ngăn chặn việc "dirty read" và "phantom read".

## Sử dụng Transaction và Isolation Level trong EF Core

- Entity Framework Core cho phép bạn kiểm soát transaction và isolation level bằng cách sử dụng phương thức `BeginTransaction` với tham số `IsolationLevel`. Dưới đây là một số ví dụ chi tiết:
> **Mức cô lập Repeatable Read, Mô phỏng hiện tượng "Dirty Read"**
```csharp title="C#"
using System;
using System.Data;
using System.Threading;
using Microsoft.EntityFrameworkCore;

class Program
{
    static void Main()
    {
        // Thread 1: Giao dịch 1 thực hiện thay đổi nhưng chưa cam kết
        var transaction1 = new Thread(Transaction1);
        transaction1.Start();

        // Chờ một 500ms để Transaction1 thực hiện
        Thread.Sleep(500);

        // Thread 2: Giao dịch 2 đọc dữ liệu với Read Uncommitted
        var transaction2 = new Thread(Transaction2);
        transaction2.Start();
    }

    static void Transaction1()
    {
        using var context = new SchoolDbContext();
        using var transaction = context.Database.BeginTransaction();

        try
        {
            // Thêm một bản ghi mới vào bảng Students nhưng chưa cam kết
            context.Students.Add(new Student
            {
                Name = "Lê Văn C",
                Age = 24,
                Code = "112233"
            });

            context.SaveChanges();

            // Đợi 3 giây để mô phỏng thời gian giao dịch lâu
            Console.WriteLine("Transaction 1: Thay đổi dữ liệu nhưng chưa cam kết...");
            Thread.Sleep(3000);

            // Hủy bỏ thay đổi
            transaction.Rollback();
            Console.WriteLine("Transaction 1: Giao dịch bị hủy.");
        }
        catch (Exception ex)
        {
            Console.WriteLine("Transaction 1 failed: " + ex.Message);
        }
    }

    static void Transaction2()
    {
        using var context = new SchoolDbContext();

        // Sử dụng Read Uncommitted
        using var transaction = context.Database.BeginTransaction(IsolationLevel.ReadUncommitted);

        try
        {
            // Đọc dữ liệu trong khi Transaction 1 chưa cam kết
            var students = context.Students.ToList();
            Console.WriteLine("Transaction 2: Đọc dữ liệu với Read Uncommitted.");
            foreach (var student in students)
            {
                Console.WriteLine($"ID: {student.StudentId}, Name: {student.Name}, Age: {student.Age}, Code: {student.Code}");
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine("Transaction 2 failed: " + ex.Message);
        }
    }
}
```
### Giải thích kết quả:
- Transaction 1: Thêm một sinh viên mới nhưng chưa cam kết và sau đó hủy giao dịch (transaction.Rollback()).
- Transaction 2: Đọc dữ liệu với mức độ cô lập mặc định Read Committed.
- Vì Transaction 1 chưa cam kết, Transaction 2 sẽ không thấy bản ghi mới mà Transaction 1 thêm vào.
- Kết quả là Transaction 2 sẽ chỉ thấy các bản ghi đã được cam kết trước đó.
- Tóm lại:
Nếu bạn không sử dụng Read Uncommitted, giao dịch thứ hai sẽ không thể đọc dữ liệu chưa cam kết từ giao dịch đầu tiên.
Đây là lý do tại sao Read Uncommitted có thể dẫn đến hiện tượng "dirty read" trong khi Read Committed thì không.

> **Mức cô lập Repeatable Read, Mô phỏng hiện tượng "Dirty Read"**
```csharp title="C#"
using System;
using System.Data;
using System.Threading;
using Microsoft.EntityFrameworkCore;

class Program
{
    static void Main()
    {
        // Thread 1: Giao dịch 1 thực hiện thay đổi nhưng chưa cam kết
        var transaction1 = new Thread(Transaction1);
        transaction1.Start();

        // Chờ một 500ms để Transaction1 thực hiện
        Thread.Sleep(500);

        // Thread 2: Giao dịch 2 đọc dữ liệu với Read Uncommitted
        var transaction2 = new Thread(Transaction2);
        transaction2.Start();
    }

    static void Transaction1()
    {
        using var context = new SchoolDbContext();
        using var transaction = context.Database.BeginTransaction();

        try
        {
            // Thêm một bản ghi mới vào bảng Students nhưng chưa cam kết
            context.Students.Add(new Student
            {
                Name = "Lê Văn C",
                Age = 24,
                Code = "112233"
            });

            context.SaveChanges();

            // Đợi 3 giây để mô phỏng thời gian giao dịch lâu
            Console.WriteLine("Transaction 1: Thay đổi dữ liệu nhưng chưa cam kết...");
            Thread.Sleep(3000);

            // Hủy bỏ thay đổi
            transaction.Rollback();
            Console.WriteLine("Transaction 1: Giao dịch bị hủy.");
        }
        catch (Exception ex)
        {
            Console.WriteLine("Transaction 1 failed: " + ex.Message);
        }
    }

    static void Transaction2()
    {
        using var context = new SchoolDbContext();

        // Sử dụng Read Uncommitted
        using var transaction = context.Database.BeginTransaction(IsolationLevel.ReadUncommitted);

        try
        {
            // Đọc dữ liệu trong khi Transaction 1 chưa cam kết
            var students = context.Students.ToList();
            Console.WriteLine("Transaction 2: Đọc dữ liệu với Read Uncommitted.");
            foreach (var student in students)
            {
                Console.WriteLine($"ID: {student.StudentId}, Name: {student.Name}, Age: {student.Age}, Code: {student.Code}");
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine("Transaction 2 failed: " + ex.Message);
        }
    }
}
```
### Giải thích kết quả:
- Transaction 1: Thêm một sinh viên mới nhưng chưa cam kết và sau đó hủy giao dịch (transaction.Rollback()).
- Transaction 2: Đọc dữ liệu với mức độ cô lập mặc định Read Committed.
- Vì Transaction 1 chưa cam kết, Transaction 2 sẽ không thấy bản ghi mới mà Transaction 1 thêm vào.
- Kết quả là Transaction 2 sẽ chỉ thấy các bản ghi đã được cam kết trước đó.
- Tóm lại:
Nếu bạn không sử dụng Read Uncommitted, giao dịch thứ hai sẽ không thể đọc dữ liệu chưa cam kết từ giao dịch đầu tiên.
Đây là lý do tại sao Read Uncommitted có thể dẫn đến hiện tượng "dirty read" trong khi Read Committed thì không.