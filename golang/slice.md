切片底层

```go
type slice struct {
    array unsafe.Pointer
    len   int
    cap   int
}
```

1. **指针：** 指向 slice 可以访问到的第一个元素。
2. **长度：** slice 中元素个数。
3. **容量：** slice 起始元素到底层数组最后一个元素间的元素个数。



扩容动作

1. **分配新数组**：根据扩容策略（可能是原容量的2倍、1.25倍或其他策略），分配一个新的、更大的数组。
2. **复制元素**：将原数组中的元素复制到这个新的数组中。
3. **更新指针**：将切片的`array`字段更新为指向这个新的数组。
4. **调整长度和容量**：更新切片的`len`和`cap`字段，以反映新的数组的长度和容量



Go 1.17版本切片扩容

Go 1.17切片扩容时会进行内存对齐，这个和内存分配策略相关。进行内存对齐之后，新 slice 的容量是要 大于等于老 slice 容量的 2倍或者1.25倍。

当新切片需要的容量cap大于两倍扩容的容量，则直接按照新切片需要的容量扩容；
当原 slice 容量 < 1024 的时候，新 slice 容量变成原来的 2 倍；
当原 slice 容量 > 1024，进入一个循环，每次容量变成原来的1.25倍,直到大于期望容量。
```go
// src/runtime/slice.go

func growslice(et *_type, old slice, cap int) slice {
    // ...

    newcap := old.cap
    doublecap := newcap + newcap
    if cap > doublecap {
        newcap = cap
    } else {
        if old.cap < 1024 {
            newcap = doublecap
        } else {
            // Check 0 < newcap to detect overflow
            // and prevent an infinite loop.
            for 0 < newcap && newcap < cap {
                newcap += newcap / 4
            }
            // Set newcap to the requested cap when
            // the newcap calculation overflowed.
            if newcap <= 0 {
                newcap = cap
            }
        }
    }

    // ...

    return slice{p, old.len, newcap}
}
```

Go 1.18版本切片扩容

Go1.18不再以1024为临界点，而是设定了一个值为256的threshold，以256为临界点；超过256，不再是每次扩容1/4，而是每次增加（旧容量+3*256）/4；

当新切片需要的容量cap大于两倍扩容的容量，则直接按照新切片需要的容量扩容；
当原 slice 容量 < threshold 的时候，新 slice 容量变成原来的 2 倍；
当原 slice 容量 > threshold，进入一个循环，每次容量增加（旧容量+3*threshold）/4。

```go
// src/runtime/slice.go

func growslice(et *_type, old slice, cap int) slice {
    // ...

    newcap := old.cap
    doublecap := newcap + newcap
    if cap > doublecap {
        newcap = cap
    } else {
        const threshold = 256
        if old.cap < threshold {
            newcap = doublecap
        } else {
            // Check 0 < newcap to detect overflow
            // and prevent an infinite loop.
            for 0 < newcap && newcap < cap {
                // Transition from growing 2x for small slices
                // to growing 1.25x for large slices. This formula
                // gives a smooth-ish transition between the two.
                newcap += (newcap + 3*threshold) / 4
            }
            // Set newcap to the requested cap when
            // the newcap calculation overflowed.
            if newcap <= 0 {
                newcap = cap
            }
        }
    }

    // ...

    return slice{p, old.len, newcap}
}
```

