---
theme: gaia
paginate: true
backgroundColor: #fff
marp: true
---



# **9 Communicating State Machines**

### 9.1 A Light Flasher Example

### 9.2 State Machine with Datapath

### 9.3 Ready-Valid Interface

---

# **9.1 A Light Flasher Example**



---

 

![Figure 9.1: The light flasher split into a Master FSM and a Timer FSM.](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-26_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_3.46.12.png)

---

```scala
val timerReg = RegInit(0.U) 
timerDone := timerReg === 0.U

// Timer FSM (down counter)
when(!timerDone) {
  timerReg := timerReg - 1.U
}
when (timerLoad) {
  when (timerSelect) { 
    timerReg := 5.U
} .otherwise { 
  timerReg := 3.U
  } 
}

val off :: flash1 :: space1 :: flash2 :: space2 :: flash3 :: Nil = Enum (6)
val stateReg = RegInit(off)

val light = WireDefault(false.B) // FSM output

// Timer connection
val timerLoad = WireDefault(false.B) // start timer with a load
val timerSelect = WireDefault(true.B) // select 6 or 4 cycles 
val timerDone = Wire(Bool())

timerLoad := timerDone

// Master FSM
switch(stateReg) { 
  is(off) {
    timerLoad := true.B
    timerSelect := true.B
    when (start) { stateReg := flash1 }
  }
  is (flash1) {
    timerSelect := false.B
    light := true.B
    when (timerDone) { stateReg := space1 }
  }
  is (space1) {
    when (timerDone) { stateReg := flash2 } 
  }
  is (flash2) {
    timerSelect := false.B
    light := true.B
    when (timerDone) { stateReg := space2 }
  }
  is (space2) {
    when (timerDone) { stateReg := flash3 } 
  }
  is (flash3) {
    timerSelect := false.B
    light := true.B
    when (timerDone) { stateReg := off }
  } 
}
```

---

![Figure 9.2: The light flasher split into a Master FSM, a Timer FSM, and a Counter FSM.](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-26_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_3.55.23.png)

---

```scala
val cntReg = RegInit(0.U) 
cntDone := cntReg === 0.U

// Down counter FSM
when(cntLoad) { cntReg := 2.U } 
when(cntDecr) { cntReg := cntReg - 1.U }
```

---

```scala
val off :: flash :: space :: Nil = Enum(3) 
val stateReg = RegInit(off)

val light = WireDefault(false.B) // FSM output

// Timer connection
val timerLoad = WireDefault(false.B) // start timer with a load
val timerSelect = WireDefault(true.B) // select 6 or 4 cycles 
val timerDone = Wire(Bool())
// Counter connection
val cntLoad = WireDefault(false.B)
val cntDecr = WireDefault(false.B)
val cntDone = Wire(Bool())

timerLoad := timerDone

switch(stateReg) { 
  is(off) {
    timerLoad := true.B
    timerSelect := true.B
    cntLoad := true.B
    when (start) { stateReg := flash }
  }
  is (flash) {
    timerSelect := false.B
    light := true.B
    when (timerDone & !cntDone) { stateReg := space } 
    when (timerDone & cntDone) { stateReg := off }
  }
  is (space) {
    cntDecr := timerDone
    when (timerDone) { stateReg := flash } 
  }
}
```

---

# **9.2 State Machine with Datapath**



---

## **9.2.1 Popcount Example**

![Figure 9.3: A state machine with a datapath.](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-26_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_4.01.48.png)

---

![Figure 9.4: State diagram for the popcount FSM.](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-26_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_4.04.28.png)

---

# Pop Count Module

```scala
class PopCount extends Module { 
  val io = IO(new Bundle {
    val dinValid = Input(Bool())
    val dinReady = Output(Bool()) 
    val din = Input(UInt(8.W))
    val popCntValid = Output(Bool()) 
    val popCntReady = Input(Bool()) 
    val popCnt = Output(UInt(4.W))
  })

  val fsm = Module(new PopCountFSM)
  val data = Module(new PopCountDataPath)

  fsm.io.dinValid := io.dinValid 
  io.dinReady := fsm.io.dinReady 
  io.popCntValid := fsm.io.popCntValid 
  fsm.io.popCntReady := io.popCntReady

  data.io.din := io.din 
  io.popCnt := data.io.popCnt 
  data.io.load := fsm.io.load 
  fsm.io.done := data.io.done
}
```

---

## Datapath

![Figure 9.5: Datapath for the popcount circuit.](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-26_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_4.06.20.png)

---

```scala
class PopCountDataPath extends Module { 
  val io = IO(new Bundle {
    val din = Input(UInt(8.W))
    val load = Input(Bool())
    val popCnt = Output(UInt(4.W)) 
    val done = Output(Bool())
  })

  val dataReg = RegInit(0.U(8.W)) 
  val popCntReg = RegInit(0.U(8.W)) 
  val counterReg= RegInit(0.U(4.W))

  dataReg := 0.U ## dataReg(7, 1) 
  popCntReg := popCntReg + dataReg(0)

  val done = counterReg === 0.U 
  when (!done) {
    counterReg := counterReg - 1.U 
  }

  when(io.load) { 
    dataReg := io.din 
    popCntReg := 0.U 
    counterReg := 8.U
  }

  // debug output
  printf("%x %d\n", dataReg, popCntReg)
  io.popCnt := popCntReg
  io.done := done 
}
```

---

## Control

```scala
class PopCountFSM extends Module { 
  val io = IO(new Bundle {
    val dinValid = Input(Bool())
    val dinReady = Output(Bool()) 
    val popCntValid = Output(Bool()) 
    val popCntReady = Input(Bool()) 
    val load = Output(Bool())
    val done = Input(Bool())
  })

  val idle :: count :: done :: Nil = Enum(3)
  val stateReg = RegInit(idle) 

  io.load := false.B

  io.dinReady := false.B 
  io.popCntValid := false.B

  switch(stateReg) { 
    is(idle) {
      io.dinReady := true.B 
      when(io.dinValid) { 
        io.load := true.B 
        stateReg := count
      } 
    }
    is(count) { 
      when(io.done) {
        stateReg := done 
      }
    }
    is(done) {
      io.popCntValid := true.B 
      when(io.popCntReady) {
        stateReg := idle 
      }
    } 
  }
}
```

---

# **9.3 Ready-Valid Interface**



![Figure 9.6: The ready-valid flow control.](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-26_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_4.08.51.png)

---

![Figure 9.7: Data transfer with a ready-valid interface, early ready](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-26_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_3.07.23.png)

---

![Figure 9.8: Data transfer with a ready-valid interface, late ready](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-26_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_3.07.57.png)

---

![Figure 9.9: Single cycle ready/valid and back-to-back transfers](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-26_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_3.08.11.png)

---

```scala
class DecoupledIO[T <: Data](gen: T) extends Bundle { 
  val ready = Input(Bool())
  val valid = Output(Bool())
  val bits = Output(gen)
}
```


