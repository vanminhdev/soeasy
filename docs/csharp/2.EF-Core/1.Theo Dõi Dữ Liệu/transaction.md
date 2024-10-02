---
title: Sử dụng Transactions
sidebar_position: 1
---
# Sử dụng Transactions trong EF-Core

**Transactions** (giao dịch) là một khái niệm quan trọng trong lập trình, đặc biệt khi làm việc với cơ sở dữ liệu và các hệ thống phân tán. Giao dịch đảm bảo rằng một tập hợp các thao tác được thực hiện một cách nguyên tử, nghĩa là tất cả các thao tác đều thành công hoặc tất cả đều bị hủy bỏ. Trong .NET, chúng ta có thể sử dụng các lớp như `TransactionScope` và `DbTransaction` để quản lý giao dịch một cách hiệu quả.

## Tại sao cần sử dụng Transactions?

- **Đảm bảo tính toàn vẹn của dữ liệu**: Transactions đảm bảo rằng dữ liệu trong cơ sở dữ liệu luôn nhất quán, ngay cả khi xảy ra lỗi hoặc sự cố.
- **Ngăn ngừa lỗi không mong muốn**: Giúp ngăn ngừa trường hợp chỉ một phần của thao tác được thực hiện thành công, dẫn đến dữ liệu không nhất quán.
- **Hỗ trợ Rollback**: Cho phép hủy bỏ toàn bộ các thao tác nếu một thao tác nào đó thất bại.

## Sử dụng Transaction

`BeginTransaction()` là một phương thức quản lý các giao dịch mạnh mẽ trong EF-Core, cho phép quản lý giao dịch một cách đơn giản. `DbContext.Database.BeginTransaction()` sẽ tự động quản lý việc bắt đầu, `transaction.Commit()` xác nhận các thay đổi giao dịch tùy thuộc vào kết quả của các thao tác trong khối lệnh của nó.

### Cách sử dụng phương thức `BeginTransaction()`
```csharp title="C#"
using System;
using System.Data;
using Microsoft.EntityFrameworkCore;

using var context = new SchoolDbContext();
using var transaction = context.Database.BeginTransaction();

try
{
    var class = context.Classes.Add(new Class 
    { 
        Name = "12A", 
        Grade = 12, 
        Code = "12A01" 
    }).Entity;
    // Lưu thay đổi vào cơ sở dữ liệu
    context.SaveChanges();
    ontext.Students.Add(new Student 
    { 
        Name = "Nguyễn Văn A", 
        Age = 20, 
        Code = "12523A",
        ClassId = class.Id
    });
    // Cam kết transaction nếu tất cả các thao tác đều thành công
    transaction.Commit();
}
catch (Exception)
{
     // Transaction sẽ tự động bị hủy bỏ khi Exception 
     Console.WriteLine("Transaction failed: " + ex.Message);
}
```
_Mã sau khi biên dịch sang query SQL:_
```sql title="SQL"
BEGIN TRANSACTION;

INSERT INTO Classes (Name, Grade, Code) 
VALUES (N'12A', 12, '12A01');

-- Lấy giá trị của ClassId vừa được chèn
DECLARE @ClassId INT = SCOPE_IDENTITY();

INSERT INTO Students (Name, Age, Code, ClassId) 
VALUES (N'Nguyễn Văn A', 20, '12523A', @ClassId);

COMMIT;
```
## Tài liệu tham khảo
1. [Microsoft Docs - Using Transactions](https://learn.microsoft.com/en-us/ef/core/saving/transactions)
2. [Entity Framework Tutorial](https://www.entityframeworktutorial.net/entityframework6/transaction-in-entity-framework.aspx)
