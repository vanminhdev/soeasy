---
title: Sử dụng Isolation Level
sidebar_position: 2
---

# Sử dụng Isolation Level trong EF-Core

**Transaction** (giao dịch) trong Entity Framework Core đảm bảo rằng một loạt các thao tác cơ sở dữ liệu được thực thi nguyên tử, tức là tất cả các thao tác đều thành công hoặc tất cả đều thất bại. Điều này giúp đảm bảo tính toàn vẹn của dữ liệu. Bên cạnh đó, **Isolation Level** (mức độ cô lập - **I** trong thuộc tính **ACID** - là viết tắt của bốn thuộc tính cơ bản mà một Transaction trong hệ thống quản lý cơ sở dữ liệu (DBMS) cần phải có để đảm bảo tính toàn vẹn và độ tin cậy của dữ liệu
) xác định cách các Transaction có thể ảnh hưởng đến nhau, đặc biệt là cách các thao tác đọc và ghi dữ liệu có thể nhìn thấy nhau.

## Isolation Levels trong SQL Server

Dưới đây là một số mức độ cô lập phổ biến:

- **Read Uncommitted**: Transaction có thể đọc dữ liệu chưa được commit(cam kết) từ các Transaction khác, có thể gây ra hiện tượng "dirty read".
- **Read Committed**: Mặc định của SQL Server. Transaction chỉ có thể đọc dữ liệu đã được commit.
- **Repeatable Read**: Đảm bảo rằng dữ liệu đã đọc sẽ không bị thay đổi bởi các Transaction khác trong suốt thời gian Transaction này đang hoạt động.
- **Serializable**: Mức độ cô lập cao nhất, ngăn không cho các Transaction khác đọc hoặc ghi dữ liệu mà Transaction này đang làm việc.
- **Snapshot**: Transaction đọc một ảnh chụp nhanh của dữ liệu tại thời điểm Transaction bắt đầu, ngăn chặn việc "dirty read" và "phantom read".
## So sánh các mức độ cô lập trong SQL Server

SQL Server cung cấp bốn mức độ cô lập chính: **Read Uncommitted**, **Read Committed**, **Repeatable Read**, và **Serializable**. Mỗi mức cô lập khác nhau về khả năng ngăn chặn các hiện tượng không mong muốn như **Dirty Read**, **Non-Repeatable Read**, và **Phantom Read**. Dưới đây là bảng so sánh chi tiết:

| **Mức độ cô lập**       | **Dirty Read** | **Non-Repeatable Read** | **Phantom Read** | **Phạm vi khóa**           | **Hiệu suất** | **Ứng dụng** |
|-------------------------|----------------|-------------------------|------------------|----------------------------|---------------|--------------|
| **Read Uncommitted**     | Có             | Có                      | Có               | Không khóa bản ghi nào      | Tốt nhất      | Dùng cho các truy vấn không yêu cầu dữ liệu chính xác tuyệt đối, hiệu suất là ưu tiên. |
| **Read Committed**       | Không          | Có                      | Có               | Khóa các bản ghi hiện tại trong suốt thời gian đọc | Tốt          | Sử dụng phổ biến cho các ứng dụng, cân bằng giữa tính toàn vẹn dữ liệu và hiệu suất. |
| **Repeatable Read**      | Không          | Không                   | Có               | Khóa các bản ghi đã đọc trong toàn bộ Transaction | Trung bình   | Dùng khi cần dữ liệu không thay đổi trong suốt Transaction nhưng không cần ngăn chặn việc thêm bản ghi mới. |
| **Serializable**         | Không          | Không                   | Không            | Khóa toàn bộ phạm vi dữ liệu được truy vấn     | Kém nhất     | Dùng cho các tình huống đòi hỏi tính toàn vẹn dữ liệu tuyệt đối, ngăn cả thay đổi và thêm mới. |

### Giải thích các hiện tượng:

1. **Dirty Read**: Một Transaction có thể đọc dữ liệu mà Transaction khác đang thực hiện nhưng chưa commit. Nếu Transaction kia rollback, dữ liệu đã đọc trở nên không hợp lệ (bẩn).
   
2. **Non-Repeatable Read**: Một Transaction đọc một bản ghi nhiều lần nhưng có thể thấy dữ liệu khác nhau do Transaction khác đã sửa dữ liệu trong lần đọc thứ hai.

3. **Phantom Read**: Một Transaction chạy hai lần cùng một truy vấn nhưng trong lần thứ hai, một hoặc nhiều bản ghi mới đã được thêm hoặc xóa bởi Transaction khác.

### Chi tiết về từng mức cô lập:

#### 1. **Read Uncommitted**:
- **Hiện tượng**: Cho phép tất cả các hiện tượng không mong muốn, bao gồm **Dirty Read**, **Non-Repeatable Read**, và **Phantom Read**.
- **Khóa**: Không khóa bất kỳ bản ghi nào.
- **Hiệu suất**: Tốt nhất vì không có khóa nào được sử dụng, nhưng dữ liệu đọc có thể không chính xác.

#### 2. **Read Committed** (Mặc định trong SQL Server):
- **Hiện tượng**: Ngăn **Dirty Read**, nhưng vẫn cho phép **Non-Repeatable Read** và **Phantom Read**.
- **Khóa**: Khóa bản ghi trong suốt thời gian đọc để tránh việc đọc dữ liệu chưa được commit.
- **Hiệu suất**: Tốt, là mức cô lập phổ biến nhất.

#### 3. **Repeatable Read**:
- **Hiện tượng**: Ngăn **Dirty Read** và **Non-Repeatable Read**, nhưng vẫn cho phép **Phantom Read**.
- **Khóa**: Khóa các bản ghi đã đọc trong toàn bộ Transaction, ngăn cản các thay đổi nhưng cho phép thêm bản ghi mới.
- **Hiệu suất**: Trung bình, phù hợp khi cần đảm bảo dữ liệu đã đọc không thay đổi.

#### 4. **Serializable**:
- **Hiện tượng**: Ngăn tất cả các hiện tượng không mong muốn: **Dirty Read**, **Non-Repeatable Read**, và **Phantom Read**.
- **Khóa**: Khóa phạm vi dữ liệu đã truy vấn, không cho phép thay đổi hoặc thêm bản ghi trong phạm vi.
- **Hiệu suất**: Kém nhất
## Sử dụng Transaction và Isolation Level trong EF Core

- Entity Framework Core cho phép bạn kiểm soát transaction và isolation level bằng cách sử dụng phương thức `BeginTransaction` với tham số `IsolationLevel`. Dưới đây là một số ví dụ chi tiết:
> **Mức cô lập Read Uncommitted, Mô phỏng hiện tượng "Dirty Read"**

```csharp title="C#"
using System;
using System.Data;
using System.Threading;
using Microsoft.EntityFrameworkCore;

class Program
{
    static void Main()
    {
        using (var context = new SchoolDbContext())
        {
            var transaction1 = context.Database.BeginTransaction(); // Mặc định sẽ là level ReadCommitted
            // Thêm một bản ghi mới vào bảng Students nhưng chưa commit
            context.Students.Add(new Student
            {
                Name = "Lê Văn C",
                Age = 24,
                Code = "112233"
            });
            context.SaveChanges();
        }

        using (var context = new SchoolDbContext())
        {
            var transaction2 = context.Database.BeginTransaction(IsolationLevel.ReadUncommitted);
            try
            {
                // Đọc dữ liệu trong khi transaction1 chưa commit
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
}
```

> **Mức cô lập Read Committed**

```csharp title="C#"
using System;
using System.Data;
using System.Threading;
using Microsoft.EntityFrameworkCore;

class Program
{
    static void Main()
    {
        using (var context = new SchoolDbContext())
        {
            var transaction1 = context.Database.BeginTransaction(); // Mặc định sẽ là level ReadCommitted
            // Thêm một bản ghi mới vào bảng Students nhưng chưa commit
            var student = context.Students.FirstOrDefault(x => x.Age >= 10); //Name = "Lê Văn C", Age = 24, Code = "112233"
            student.Code = "153276"
            context.SaveChanges();
        }

        using (var context = new SchoolDbContext())
        {
            var transaction2 = context.Database.BeginTransaction();
            try
            {
                // Đọc dữ liệu trong khi transaction1 chưa commit
                var student = context.Students.FirstOrDefault(x => x.Age >= 10); 
                Console.WriteLine($"student: {student}"); //Name = "Lê Văn C", Age = 24, Code = "112233"
                student.Name = "Trần Công D"
                context.SaveChanges();
                transaction2.Commit();
            }
            catch (Exception ex)
            {
                Console.WriteLine("Transaction 2 failed: " + ex.Message);
            }
        }
    }
}
```

> **Mức cô lập Repeatable Read**

```csharp title="C#"
using System;
using System.Data;
using System.Threading;
using Microsoft.EntityFrameworkCore;

class Program
{
    static void Main()
    {
        using (var context = new SchoolDbContext())
        {
            var transaction1 = context.Database.BeginTransaction(); // Mặc định sẽ là level ReadCommitted
            // Thêm một bản ghi mới vào bảng Students nhưng chưa commit
            var student = context.Students.FirstOrDefault(x => x.Name == "Lê Văn C"); 
            student.Code = "153276" //Khóa đọc được đặt trên bản ghi với Name = "Lê Văn C"
            context.SaveChanges();
        }

        using (var context = new SchoolDbContext())
        {
            var transaction2 = context.Database.BeginTransaction(IsolationLevel.RepeatableRead);
            try
            {
                var studentsWithoutC = context.Students.Where(x => x.Name != "Lê Văn C"); // query vẫn sẽ chạy bản ghi ngoại trừ sinh viên C
                // Đọc dữ liệu trong khi transaction1 chưa commit
                var student = context.Students.FirstOrDefault(x => x.Name == "Lê Văn C"); // query sẽ bị lock tại đây
                // Tiếp tục chạy khi transaction1 commit
                Console.WriteLine($"student: {student}");
                student.Name = "Trần Công D"
                context.SaveChanges();
                transaction2.Commit();
            }
            catch (Exception ex)
            {
                Console.WriteLine("Transaction 2 failed: " + ex.Message);
            }
        }

        using (var context = new SchoolDbContext())
        {
            var student = context.Students.FirstOrDefault(x => x.Name == "Lê Văn C"); // query vẫn sẽ chạy bản ghi được commit gần nhất
        }
    }
}
```

> **Mức cô lập Serializable**

```csharp title="C#"
using System;
using System.Data;
using System.Threading;
using Microsoft.EntityFrameworkCore;

class Program
{
    static void Main()
    {
        using (var context = new SchoolDbContext())
        {
            var transaction1 = context.Database.BeginTransaction(IsolationLevel.Serializable);
            try
            {
                var students = context.Students.Where(x => x.Age >= 10); // query sẽ lock range bản ghi sinh viên có tuổi >= 10
            }
            catch (Exception ex)
            {
                Console.WriteLine("Transaction 2 failed: " + ex.Message);
            }
        }

        using (var context = new SchoolDbContext())
        {
            var transaction2 = context.Database.BeginTransaction(); // Mặc định sẽ là level ReadCommitted
            var student = context.Students.FirstOrDefault(x => x.Age == 9);
            Console.WriteLine($"student: {student}");
            // Thêm một bản ghi mới vào bảng Students nhưng chưa commit
            var studentWith10Age = context.Students.FirstOrDefault(x => x.Age >= 10); 
            student.Code = "153276"// query lock tại đây
            context.SaveChanges();
        }

      
        using (var context = new SchoolDbContext())
        {
            var transaction3 = context.Database.BeginTransaction(); // Mặc định sẽ là level ReadCommitted
            // Thêm một bản ghi mới vào bảng Students
            context.Students.Add(new Student
            {
                Name = "Trần Quốc E",
                Age = 9,
                Code = "115633"
            });
            context.SaveChanges();
            transaction3.Commit(); // Save thành cong do transaction 1 chỉ lock range >= 10
        }

        using (var context = new SchoolDbContext())
        {
            var transaction4 = context.Database.BeginTransaction(); // Mặc định sẽ là level ReadCommitted
            // Thêm một bản ghi mới vào bảng Students
            context.Students.Add(new Student
            {
                Name = "Vũ Quang F",
                Age = 20,
                Code = "118933"
            });
            context.SaveChanges();
            transaction4.Commit(); // Save khôngthành cong do transaction 1 lock range >= 10 không cho phép thêm mới
        }
    }
}
```
