``` F#
open System

[<Struct>]
type A (value : int) =
  override this.ToString() : string = value.ToString()

type B (a : A) =
  override this.ToString() : string = (a :> Object).ToString()

[<EntryPoint>]
let main args =
  for i in 1 .. 3 do
    Console.WriteLine(B(A(42)))
  0
```

Output:
```
>dotnet run -c Release
1
1
42
```

A variation which reads memory garbage:
``` F#
open System

[<Struct>]
type A (value : int) =
  override this.ToString() : string = value.ToString()

type B (a : A) =
  override this.ToString() : string = (a :> Object).ToString()
  
[<EntryPoint>]
let main args =
  for i in 1 .. 3 do
    let a = A(42)
    let b = B(a)
    printfn "i: %d, a: %s, b: %O" i ((a :> Object).ToString()) b

  0
```

Possible output:
```
dotnet run -c Release
i: 1, a: 42, b: 506977008 // this value is different across runs
i: 2, a: 42, b: 506977008
i: 3, a: 42, b: 42
```

To repro the bug, you need
* release build
  * optimization enabled
  * tail calls enabled
* .NET Core runtime 2.1.0 or above.

The versions I tried:

| net461 | 2.0.0 | 2.0.5 | 2.0.7 | 2.1.0 | 2.1.3 | 2.1.5 | 2.1.6 | 2.2.0 |
|--------|-------|-------|-------|-------|-------|-------|-------|-------|
|   -    |   -   |   -   |   -   |  bug  |  bug  |  bug  |  bug  |  bug  |


