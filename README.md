# Functional Visual Basic
Let's embark to functional way !!
***

## ตอนนี้เรามีอะไรกันบ้าง
- Tail call loop
- Recursion
- Immutable
- Link list
***
# Object ต่างๆ ที่เกี่ยวข้องกับการเขียนแบบ functional ที่ต้องใช้ประกอบ
## Value tuple
`Value tuple` นั้นเริ่มมีใช้ใน `.net framework 4.7` เป็นต้นไป(และ `Visual Basic 15.3` สำหรับการเรียกใช้ชื่อสมาชิกซ้ำซ้อน)
Object นี้เป็นส่วนสำคัญมากเนื่องจากเป็นการสร้าง structure แบบอัตโนมัติซึ่งช่วยให้เราใช้ value type ได้อย่างสะดวกและหลากหลาย

- วิธีสร้างแบบไม่ตั้งชื่อสมาชิก
```vb
Dim Data = (999, 666)
'Data.Item1 == 999
'Data.Item2 == 666
```
```vb
Dim Data As (Integer, Integer) = (999, 666)
'Data.Item1 == 999
'Data.Item2 == 666
```

- การสร้างแบบตั้งชื่อสมาชิก
```vb
Dim Data = (a:=999, b:=666)
'Data.a == 999
'Data.b == 666
```
```vb
Dim Data As (a As Integer, b As Integer) = (999, 666)
'Data.a == 999
'Data.b == 666
```

## Lamda method
`Lamda method` เป็นการสร้างฟังค์ชั่นขึ้นมาภายในฟังค์ชั่นนั้นๆ ซึ่งจะถูกเก็บเอาไว้ในรูปแบบ `fuction pointer` หรือ `Delegate` ทำให้เราไม่ต้องวิ่งไปวิ่งมาเที่ยวสร้างฟังค์ชั่นไว้ที่นู้นนี่นั้นให้ต้องมานั่งหาเวลากลับมาอ่านโค๊ดแถมยังจำกัดขอบเขตการใช้งานให้ฟังค์ชั่นนั้นๆ สามารถใช้งานเฉพาะในฟังค์ชั่นที่ถูกสร้างได้อีกด้วย

- การสร้าง `Lamda method` แบบปรกติ
```vb
Dim Plus1 = Function(Input As Integer) As Integer
                Return Input + 1
            End Function

Dim Write = Sub(Input As Integer)
                Console.Write(Input)
            End Sub
```

- การสร้าง `Lamda method` แบบลดรูป ซึ่งการสร้างแบบนี้นั้นจะจำกัดโค๊ดในฟังค์ชั่นนั้นๆ ให้ยาวเพียง 1 การทำการ(ไม่จำเป็นต้อง 1 บรรทัด)
```vb
Dim Plus1 = Function(Input As Integer) Input + 1

Dim Write = Sub(Input As Integer) Console.Write(Input)
```

- การสร้าง `Lamda method` แบบใช้ `Infer` หรือการคาดการณ์ `type` ของวัตถุ การใช้สร้างแบบนี้นั้นจะต้องมีองค์ประกอบที่ทำให้ Intellisense นั้นคาดการณ์ได้ว่าฟังค์ชั่นนั้นๆ จะรับค่าแบบไหนเข้าไปบ้างและคืนค่าอะไรกลับออกมา
```vb
Dim Plus1 As Func(Of Integer, Integer) = Function(Input) Input + 1

Dim Write As Action(Of Integer) = Sub(Input) Console.Write(Input)
```

### ข้อควรระวังสำคัญในการใช้ lamda method นั้น ต้องระวังการนำ value และ varient จากภายนอกเข้ามาใช้ในฟังค์ชั่นเนื่องจากคอมไพล์เลอร์จะสร้างคลาสลับขึ้นมาเพื่อผนวกค่านั้นๆ เข้ากับ lamda method ซึ่งจะเป็นการเพิ่มงาน GC โดยใช่เหตุ

## Pipeline
`Pipeline` เป็นแนวการเขียนแบบทำหลายๆ อย่างใน 1 การทำการ
```vb
Call (Sub(Input As Integer) Console.Write(Input))(Function(Input As Integer) Input + 1)(8))
' Console write :: 9
```
***
# ฟังค์ชั่นต่างๆ ใน FVB รุ่นล่าสุด

## Recursion
การเรียกใช้ฟังค์ชั่นนั้นๆ ซ้ำๆ เองนั้นโดยปรกติแล้วจะทำให้เกิดการดอง `stack` เกิดขึ้นซึ่งจะนำมาซึ่งปัญหา `Stack overflow`ได้ ดังนี้เราจึงจำเป็นต้องมีฟังค์ชั่นที่จะช่วยทำการ `tail call/jmp` เพื่อไม่ให้เกิดปัญหาดังกล่าว

```vb
<Extension>
recursive(Of T)(Data As T, Do_until_condition As Func(Of T, Boolean), Recursive_method As Func(Of T, T)) As T
```
- `Data` ข้อมูลที่จำเป็นที่เราต้องใช้เพื่อให้ได้ผลลัพย์ที่ต้องการ
- `Do_until_condition` ฟังค์ชั่นตรวจสอบเงื่อนไขที่จะบอกฟังค์ชั่น `recursive` ว่าเราไม่ต้องการเรียกใช้ `Recursive_method` ซ้ำอีกต่อไปแล้ว
- `Recursive_method` ฟังค์ชั่นที่เราต้องการจะเรียกใช้ซ้ำๆ

ตัวอย่างการใช้งาน
```vb
Console.WriteLine(
    (input:=5, sum:=1).recursive(
        Function(Data As (input As Integer, sum As Integer)) (Data.input - 1, Data.sum * Data.input),
        Function(Data As (input As Integer, sum As Integer)) Data.input < 2
    ).sum
)
' Console write :: 120
```
```vb
Console.WriteLine(
    (input:=5, sum:=1).recursive(
        Function(Data) Data.input < 2,
        Function(Data) (Data.input - 1, Data.sum * Data.input)
    ).sum
)
' Console write :: 120
```


```vb
<Extension>
recur(Of T)(Data As (Is_end As Boolean, o As T), Recursive_method As Func(Of (Is_end As Boolean, o As T), (Is_end As Boolean, o As T))) As T
```

- `Data` 
  - `Is_end` หากมีค่าเป็น True ให้ทำการหยุดฟังค์ชั่น 
  - `o` ข้อมูลที่จำเป็นที่เราต้องใช้เพื่อให้ได้ผลลัพย์ที่ต้องการ
- `Recursive_method` ฟังค์ชั่นที่เราต้องการจะเรียกใช้ซ้ำๆ

```vb
Console.WriteLine(
    (False, (input:=5, sum:=1)).recur(
        Function(G) (G.o.input < 2, (G.o.input - 1, G.o.sum * G.o.input))
    ).sum
)
' Console write :: 120
```

## Tail call loop
การวนแบบทิ้งหางเดิม หรือ `tail call/jmp` นั้นแตกต่างจากการวนด้วย `for` `do` `while` แบบปรกติที่จะใช้คำสั่ง `br/goto` ในการทวนคำสั่งภายในฟังค์ชั่นนั้นๆ ซึ่งไม่ต่างจากการเขียนฟังค์ชั่นยาวๆ เลยสำหรับ CPU
### ฟั้งค์ชั่นในกลุ่ม Tail call loop นี้จะไม่ทำการอัพเดตนำเข้าค่าเหมือนกับฟังค์ชั่นในกลุ่ม Recursion

- Array loop
```vb
<Extension>
each(Of T)(Array As T(), For_each As Action(Of T)) As T()
```
```vb
Call {1, 2, 3, 4, 5}.each(Sub(Item) Console.Write(Item))
' Console write :: 54321
```

```vb
<Extension>
each(Of T, V)(Array As T(), Param As V, For_each As Action(Of T, V)) As (T(), V)
```
```vb
Call {1, 2, 3, 4, 5}.each(10, Sub(Item, P) Console.Write(P - Item))
' Console write :: 56789
```

```vb
<Extension>
loop(Of T)(Array As T(), Do_loop As Action(Of T(), Integer)) As T()
```
```vb
Call {1, 2, 3, 4, 5}.loop(Sub(Array, Index) Console.Write(Array(Index)))
' Console write :: 54321
```

```vb
<Extension>
loop(Of T, V)(Array As T(), Param As V, Do_loop As Action(Of T(), Integer, V)) As (T(), V)
```
```vb
Call {1, 2, 3, 4, 5}.loop(10, Sub(Item, Index, P) Console.Write(P - Array(Index)))
' Console write :: 56789
```
- Do loop
```vb
<Extension>
loop(Of T)(Data As T, Max As Integer, Do_loop As Action(Of T, Integer)) As T
```
```vb
Call {1, 2, 3, 4, 5}.loop(2, Sub(Array, Index) Console.Write(Array(Index)))
' Console write :: 321
```

```vb
<Extension>
loop(Of T)(Data As T, Max As Integer, Min As Integer, Do_loop As Action(Of T, Integer)) As T
```
```vb
Call {1, 2, 3, 4, 5}.loop(3, 1, Sub(Array, Index) Console.Write(Array(Index)))
' Console write :: 234
```

## Immutable
เนื่องจากการเขียนแบบ functional นี้จะใช้ value type เสียมากผมเลยเห็นว่าการที่สามารถกำหนดให้ค่านั้นๆ เปลี่ยนแปลงไม่ได้จะช่วยป้องกันเหตุที่คาดไม่ถึงได้
```vb
<Extension>
refer(Of T As Structure)(Input As T) As reference(Of T)
```
```vb
<Extension>
read_only(Of T As Structure)(Input As T) As constant(Of T)
```
```vb
<Extension>
seal(Of T As Structure)(Input As T) As immutable(Of T)
```
- reference เป็นคลาสช่วยในการ boxing ค่า
- constant เป็นการ boxing ค่าเช่นกันแต่ค่านี้จะเปลี่ยนแปลงไม่ได้
- immutable เป็นการทำให้ค่าเปลี่ยนแปลงไม่ได้เฉยๆ ไม่ได้ทำการ boxing แต่อย่างใด

## Link list
เป็นการสร้างลิงค์ลิซด้วย Value tuple 

```vb
<Extension>
put(Of T, V)(O As (V, T), Val As V) As (V, o As (V, T))
```
```vb
<Extension>
link(Of T As Structure, V)(O As (V, T), Length As int) As (l As int, o As (V, T))
```
```vb
<Extension>
list(Of T, V As Structure)(Input As V, _Index As Integer) As T
```

- `put` LIFO หุ้มค่าใหม่ใส่ก่อนหน้าค่าเก่าเช่น `(0,1).put(2).put(3) == (3, (2, (0, 1)));`
- `link` เพิ่มค่าตัวเลขเอาไว้หน้าสุดเพื่อระบุว่าลิงค์นี้มีสมาชิกเท่าไหร
- `list` เข้าถึงค่าแบบเดียวกับอาเรย์ด้วยการระบุตำแหน่งที่ แทนการเรียกชื่อสมาชิก

```vb
Dim Link_list = (" my", "Hello").put(" brand").put(" new").put(" world.").link(4)
Link_list.loop(Link_list.l, Sub(G, I) G.o.list(Of String)(I).write)
' Console write :: Hello my brand new world.
```
***
# Guildline แนวทางในการเขียนสไตล์ functional

- Stay fresh พยายามรักษาความสดใหม่ของค่านั้นๆ เอาไว้ หรืออีกนัยหนึ่งก็คือแม้ตัวแปรนั้นๆ จะไม่ได้เป็น `immutable` แต่เราก็จะถือว่ามันเป็นตัวแปรแบบแก้ไขค่าไม่ได้

- Less variant ลดละเลิกการใช้ตัวแปรภายในฟังค์ชั่น หรืออย่างน้อยก็ใช้ Anonymous local variant ซึ่งใน VB นั้นจะมี `With statement`อยู่ อย่างไรก็ดีพยายามให้ไม่มีจะดีกว่า

- No side effect ฟังค์ชั่นไหนฟังค์ชั่นนั้น มีหน้าที่อะไรก็ให้ทำไปตามนั้น ไม่ควรแทรกงานอื่นๆ เข้าไปในฟังค์ชั่น ถ้าจะทำอย่างอื่นก็ให้สร้างฟังค์ชั่นใหม่มารับหน้าที่นั้นๆ แทน

> A pure function is a function where the return value is only determined by its input values, without observable side effects. This is how functions in math work: Math.cos(x) will, for the same value of x , always return the same result. Computing it does not change x.

- Short and clear พยายามทำให้ฟังค์ชั่นนั้นๆ สั้นและชัดเจนที่สุดเท่าที่จะทำได้ ไม่อย่างนั้นอาจจะเกิดอาการยืนงงอยู่กลางดง pipeline ได้ง่ายๆ ครับ
