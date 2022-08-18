---
theme: gaia
paginate: true
backgroundColor: #fff
marp: true
---



# 8 Finite-State Machines

### **8.1 Basic Finite-State Machine**

### **8.2 Faster Output with a Mealy FSM**

### **8.3 Moore versus Mealy**

### **8.4 Exercise**

---

# **8.1 Moore Machine**

![Figure 8.1: A finite-state machine (Moore type).](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-26_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_3.00.27.png)

---

## The state diagram of an alarm FSM

![Figure 8.2: The state diagram of an alarm FSM.](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-26_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_3.00.42.png)

---

## State table for the alarm FSM

![Table 8.1: State table for the alarm FSM.](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-26_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_3.04.03.png)

---

## 

```scala
import chisel3._ 
import chisel3.util._

class SimpleFsm extends Module { 
  val io = IO(new Bundle{
    val badEvent = Input(Bool()) 
    val clear = Input(Bool())
    val ringBell = Output(Bool())
  })

  // The three states
  val green :: orange :: red :: Nil = Enum(3)

  // The state register
  val stateReg = RegInit(green)

  // Next state logic
  switch (stateReg) { 
    is (green) {
      when(io.badEvent) { 
        stateReg := orange
      } 
    }
    is (orange) { 
      when(io.badEvent) {
        stateReg := red
      } .elsewhen(io.clear) {
        stateReg := green 
      }
    }
    is (red) {
      when (io.clear) { 
        stateReg := green
      } 
    }
  }

  // Output logic
  io.ringBell := stateReg === red 
}
```

---

## Chisel Bundle:

```scala
val io = IO(new Bundle{
  val badEvent = Input(Bool()) 
  val clear = Input(Bool())
  val ringBell = Output(Bool())
})
```

---

## FSM States

```scala
val green :: orange :: red :: Nil = Enum(3)
```

```scala
val stateReg = RegInit(green)
```

---

## Next state logic

```scala
switch (stateReg) { 
  is (green) {
    when(io.badEvent) { 
      stateReg := orange
    } 
  }
  is (orange) { 
    when(io.badEvent) {
      stateReg := red
    } .elsewhen(io.clear) {
      stateReg := green 
    }
  }
  is (red) {
    when (io.clear) { 
      stateReg := green
    } 
  }
}
```

---

## Output Logic

```scala
io.ringBell := stateReg === red
```

---

# **8.2 Faster Output with a Mealy FSM**

```scala
val risingEdge = din & !RegNext(din)
```

![Figure 8.3: A rising edge detector (Mealy type FSM).](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-26_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_3.33.37.png)

---



![Figure 8.4: A Mealy type finite-state machine.](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-26_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_3.33.53.png)

---

# **8.3 Moore versus Mealy**



![](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-26_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_3.34.29.png)

![](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-26_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_3.36.35.png)



---

```scala
import chisel3._ 
import chisel3.util._

class RisingFsm extends Module { 
  val io = IO(new Bundle{
    val din = Input(Bool())
    val risingEdge = Output(Bool()) 
  })
  
  // The two states
  val zero :: one :: Nil = Enum(2) 

  // The state register
  val stateReg = RegInit(zero) 

  // default value for output
  io.risingEdge := false.B

  // Next state and output logic
  switch (stateReg) { 
    is(zero) {
      when(io.din) {
        stateReg := one 
        io.risingEdge := true.B
      } 
    }
    is(one) { 
      when(!io.din) {
        stateReg := zero 
      }
    } 
  }
}
```

---

![Figure 8.7: Mealy and a Moore FSM waveform for rising edge detection.](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-26_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_3.36.52.png)

---

```scala
import chisel3._ 
import chisel3.util._

class RisingMooreFsm extends Module { 
  val io = IO(new Bundle{
    val din = Input(Bool())
    val risingEdge = Output(Bool()) 
  })

  // The three states
  val zero :: puls :: one :: Nil = Enum(3) 

  // The state register
  val stateReg = RegInit(zero)

  // Next state logic
  switch (stateReg) { 
    is(zero) {
      when(io.din) { 
        stateReg := puls
      } 
    }
    is(puls) { 
      when(io.din) {
        stateReg := one 
      } .otherwise {
        stateReg := zero 
      }
    }
    is(one) {
      when(!io.din) { 
        stateReg := zero
      } 
    }
  }

  // Output logic
  io.risingEdge := stateReg === puls 
}
```

---

# **8.4 Exercise**

이 장에서는 매우 작은 FSM의 많은 예를 보았습니다. 이제 실제 FSM 코드를 작성할 시간입니다. 조금 더 복잡한 예를 선택하고 FSM을 구현하고 이에 대한 테스트 벤치를 작성하십시오.

FSM의 전형적인 예는 신호등 컨트롤러입니다([3, 섹션 14.3] 참조). 신호등 컨트롤러는 빨간색에서 녹색으로 전환할 때 교차로의 두 도로 사이에 금지 신호등(빨간색 및 주황색)이 있는 단계가 있는지 확인해야 합니다. 이 예를 조금 더 흥미롭게 만들려면 우선 순위를 고려하십시오. 작은 도로에는 두 대의 차량 감지기가 있습니다(교차로로 들어가는 두 입구 모두). 차가 감지될 때만 작은 도로에 대해 녹색으로 전환한 다음 우선 도로에 대해 다시 녹색으로 전환합니다.