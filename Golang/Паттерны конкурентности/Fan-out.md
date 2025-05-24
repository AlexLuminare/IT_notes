Один канал на входе - несколько каналов на выходе
![[Pasted image 20250524105323.png]]

```go 



func FanOut(in chan int, num_chans int) []chan int {  
    chans := make([]chan int, num_chans)  
    for i := range chans {  
       chans[i] = pipeline(in)  
    }  
  
    return chans  
  
}  
  
func pipeline(in chan int) chan int {  
    out := make(chan int)  
    go func() {  
       for v := range in {  
          out <- sqr(v)  
       }  
       close(out)  
    }()  
    return out  
}  
  
func sqr(v int) int {  
    return v * v  
}
```