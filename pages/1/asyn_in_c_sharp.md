- [1. Task](#1-task)
  - [1.1 Task không có kiểu trả về](#11-task-không-có-kiểu-trả-về)
  - [1.2 Task có kiểu trả về](#12-task-có-kiểu-trả-về)
- [2. Từ khóa Async, Await](#2-từ-khóa-async-await)
  - [2.1 Đối với không có kiểu trả về](#21-đối-với-không-có-kiểu-trả-về)
  - [2.2 Đối với có kiểu trả về](#22-đối-với-có-kiểu-trả-về)

# 1. Task

## 1.1 Task không có kiểu trả về

Nhận vào một Action không có param

```c#
Task t2 = new Task(Action);
```

Nhận vào một action có param, đối số thứ 2 chính là giá trị của param mà ta cần truyền vào action

```c#
Task t3 new Task(Action<object>, object);
```

Ta muốn chờ tất cả các task phải thực thi xong mới được gọi đến `Console.WriteLine("Press any key")` thì ta dùng method `Wait()
Khi ta gọi đến phương thức này thì các task lúc thực thi vẫn thực thi bất đồng bộ, nhưng nó sẽ chờ cho đến khi cả 2 task t2 và t3 thực thi xong
thì nó mới gọi đến statement tiếp theo.

```c#
t2.Wait();
t3.Wait();
Console.WriteLine("Press any key");
```

Thay vì ta muốn gọi ra từng task phải `Wait()` như vậy thì ta dùng phương thức sau cho ngắn gọi

```c#
Task.WaitAll(t2, t3);
```

Ta xem xét đoạn chương trình sau:

```c#
static Task Task2()
{
    Task t2 = new Task(() =>
    {
        // Do something
    });

    t2.Start();
    t3.Wait();
    Console.WriteLine("T2 Finish");
    return t2;
}

static Task Task3()
{
    Task t3 = new Task(() =>
    {
        // Do something
    });

    t3.Start();
    t3.Wait();
    Console.WriteLine("T3 Finish");
    return t3;
}
```

Khi thực thi đoạn lệnh bên dưới , mặc dù 2 Task nằm trên 2 luồn khác nhau nhưng mỗi task lại gọi đến `Wait()` cụ thể là `t2.Wait()` và `t3.Wait();` thì kết quả thực thi chương trình như là chạy đồng bộ
Vì các task t2 nó phải đợi thực thi xong mới được gọi đến dòng lệnh `Console.WriteLine("T2 Finish");` điều này tương tự với t3.

```c#
Task t2 = Task2();
Task t3 = Task3();
```

## 1.2 Task có kiểu trả về

Trả về là một giá trị string và không có tham số đầu vào

```c#
Task<string> t3 = new Task<string>(Func<string>);
```

Trả về là một giá trị kiểu string và có tham số đầu vào, tham số đầu vào sẽ được truyền vào đố số thứ 2 của task (đang được khai báo là object)

```c#
Task<string> t3 = new Task<string>(Func<object, string>, object);
```

Ta xem xét đoạn chương trình bên dưới

```c#
Task<string> t4 = new Task<string>(() =>
{
    // Do something
    return "Result from T4";
});

Task<string> t5 = new Task<string>((object obj) =>
{
    // Do something
    string t = (string) obj;
    return $"Result from {t}";
}, "T5");

t4.Start();
t5.Start();
// Do something
Task.WaitAll(t4, t5);
// Lấy kết quả từ t4 và t5
var resultT4 = t4.Result;
var resultT5 = t5.Result;
Console.WriteLine(" Press any key");
```

Biến đổi lại thành function con như sau:

```c#
static Task<string> Task4()
{
    Task<string> t4 = new Task<string>(() =>
    {
        // Do something
        return "Result from T4";
    });
    t4.Start();
    return t4;
}

static Task<string> Task5()
{
    Task<string> t5 = new Task<string>((object obj) =>
    {
        // Do something
        string t = (string)obj;
        return $"Result from {t}";
    }, "T5");
    t5.Start();
    return t5;
}
```

Lúc gọi thực thi

```c#
Task<string> t4 = Task4();
Task<string> t5 = Task5();
            
Task.WaitAll(t4, t5);
// Lấy kết quả từ t4 và t5
var resultT4 = t4.Result;
var resultT5 = t5.Result;
Console.WriteLine(" Press any key");
```

# 2. Từ khóa Async, Await

Khi ta dùng từ khóa async, await như bên dưới thì lúc này:
Khi gọi đến `await t2;` hoặc `await t3;` thì chương trình vẫn đợi cho t2 thực thi xong mới chạy statement tiếp theo
Nhưng nó sẽ không khóa thread chính, 2 task vẫn chạy trên 2 luồng khác nhau và không depend lên nhau.
Ta có thể thấy kết quả của 2 task sẽ luân phiên nhau chứ không phải `t2` xong hết rồi mới đến `t3`.
Bản chất của việc này khi nó gặp từ khóa `await` thì nó sẽ chờ t2 thực thi xong mới gọi đến statement tiếp theo, nhưng nó sẽ không lock luồn chính, mà nó vẫn trả về một task để luồng chính tiếp tục thực thi việc của nó.

## 2.1 Đối với không có kiểu trả về

```c#
static async Task Task2()
{
    Task t2 = new Task(() =>
    {
        // Do something
    });

    await t2;
    Console.WriteLine("T2 Finish");
}

static async Task Task3()
{
    Task t3 = new Task(() =>
    {
        // Do something
    });

    t3.Start();
    await t3;
    Console.WriteLine("T3 Finish");
}
```

## 2.2 Đối với có kiểu trả về

```c#
static async Task<string> Task4()
{
    Task<string> t4 = new Task<string>(() =>
    {
        // Do something
        return "Result from T4";
    });
    t4.Start();
    var result = await t4;
    Console.WriteLine("T4 finish");
    return result;
}

static async Task<string> Task5()
{
    Task<string> t5 = new Task<string>((object obj) =>
    {
        // Do something
        string t = (string)obj;
        return $"Result from {t}";
    }, "T5");
    var result = await t5;
    Console.WriteLine("T5 finish");
    return result;
}
```

Khi gọi thực thi

```c#
Task<string> t4 = Task4();
Task<string> t5 = Task5();

// Lấy kết quả từ t4 và t5
var resultT4 = await t4;
var resultT5 = await t4;

Console.WriteLine(" Press any key");
```
