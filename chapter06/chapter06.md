---
theme: gaia
paginate: true
backgroundColor: #fff
marp: true
---

# **Chapter 6** : Sequential Circuit



### 6.1 Register

### 6.2 Counter

### 6.3 Shift Register

### 6.4 Memory

---

# **6.1 Register**

![bg right:30% 100%](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-24_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.17.28.png)

jdflksdjflks jlfj kldj

```scala
val q = Reg(UInt(4.W)) //Register Declaration
q := d //Next-state Logic
```

jdflk sdjflks  jlfj kldj

```scala
val q = RegNext(d`) //Functional Abstraction
```

---

## 6.1.1 Register with Sync Reset

![bg right:40% 100%](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-24_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.19.33.png?text=A)

Adssadsadasdsa

```scala
val q = RegInit(0.U(4.W)) //Initial Value

q := data //Next State Logic
```

sdsadsadasd

```scala
val q = RegNext(data, 0.U(4.W)) //Functional Abstraction
```



---

## 6.1.1 Register with Sync Reset

![bg fit](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-24_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.19.51.png)

---

## 6.1.2 Register with Async Reset





---

## 6.1.2 Register with Async Reset





---

## 6.1.3 Register with an Enable Signal

![bg right:30% 100% fit](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-24_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.20.47.png)

```scala
val enableReg = Reg(UInt(4.W)) //no reset

when (enable) { 
  enableReg := inVal
}
```

```scala
val enableReg2 = RegEnable(inVal, enable)
```

---

## 6.1.3 Register with an Enable Signal

![bg fit](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-24_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.21.09.png)



---

## 6.1.4 Register with an Enable and Reset Signal 

```scala
val resetEnableReg = RegInit(0.U(4.W)) //with initial value

when (enable) { 
  resetEnableReg := inVal
}
```

```scala
val resetEnableReg2 = RegEnable(inVal, 0.U(4.W), enable)
```

```scala
val risingEdge = din & !RegNext(din)
```

---

# **6.2 Counters**

![bg right:40% 100% fit](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-24_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.23.58.png)

```scala
val cntReg = RegInit(0.U(4.W))

cntReg := cntReg + 1.U
```

---

## Counter with Event Signal

![bg right:40% 100% fit](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-24_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.24.13.png)

```scala
val cntEventsReg = RegInit(0.U(4.W)) 
when(event) {
  cntEventsReg := cntEventsReg + 1.U 
}
```

- 4비트 레지스터가 있는 free running 카운터는 0에서 15까지 계산한 다음 다시 0으로 래핑
- 레지스터는 `0.W`로 reset 됨

---

## **6.2.1-a Up Counter**

**Up-Counter**

```scala
val cntReg = RegInit(0.U(8.W))

cntReg := cntReg + 1.U 
when(cntReg === N) {
  cntReg := 0.U 
}
```



```scala
val cntReg = RegInit(0.U(8.W))

cntReg := Mux(cntReg === N, 0.U, cntReg + 1.U)
```

---

## **6.2.1-b Down Counter**

**Down-Counter**

```scala
val cntReg = RegInit(N)

cntReg := cntReg - 1.U 
when(cntReg === 0.U) {
  cntReg := N 
}
```

```scala
val cntReg = RegInit(N)

cntReg := RegMux(cntReg === 0.U, N, cntReg - 1.U)
```

---

## **6.2.1-c Functional Abstaction**

```scala
// This function returns a counter
def genCounter(n: Int) = {
  val cntReg = RegInit(0.U(8.W))
  cntReg := Mux(cntReg === n.U, 0.U, cntReg + 1.U) 
  cntReg //함수의 반환 값
}

---

// now we can easily create many counters
val count10 = genCounter (10) 
val count99 = genCounter (99)
```

---

## **6.2.2 Generating Timing with Counters**

![bg vertical](./image/fake_image.png)

![bg height:300px](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-24_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.27.15.png)

```scala
val tickCounterReg = RegInit(0.U(32.W)) 
val tick = tickCounterReg === (N-1).U

tickCounterReg := tickCounterReg + 1.U 
when (tick) {
  tickCounterReg := 0.U 
}                                                                                             
```

---

## **6.2.2 Generating Timing with Counters**

![bg vertical](./image/fake_image.png)

![bg height:300px](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-24_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.28.17.png)

```scala
val lowFrequCntReg = RegInit(0.U(4.W)) 
when (tick) {
  lowFrequCntReg := lowFrequCntReg + 1.U 
}
```

---

## **6.2.3 The Nerd Counter**

```scala
val MAX = (N - 2).S(8.W) 
val cntReg = RegInit(MAX) 
io.tick := false.B

cntReg := cntReg - 1.S 
when(cntReg(7)) {
  cntReg := MAX
  io.tick := true.B 
}
```

---

## **6.2.4 A Timer**

![bg right:45% 100%](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-24_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.30.26.png)



```scala
val cntReg = RegInit(0.U(8.W)) 
val done = cntReg === 0.U

val next = WireDefault(0.U) 
when (load) {
  next := din
} .elsewhen (!done) {
  next := cntReg - 1.U 
}
cntReg := next
```

---

## **6.2.5 Pulse-Width Modulation**

```scala
def pwm(nrCycles: Int, din: UInt) = {
  val cntReg = RegInit(0.U(unsignedBitLength(nrCycles-1).W)) 
  cntReg := Mux(cntReg === (nrCycles-1).U, 0.U, cntReg + 1.U) 
  din > cntReg
}

val din = 3.U
val dout = pwm(10, din)                                                                 
```



![bg vertical](./image/fake_image.png)

![bg height:300px](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-24_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_9.31.35.png)

---

```scala
val FREQ = 100000000 // a 100 MHz clock input 
val MAX = FREQ/1000 // 1 kHz

val modulationReg = RegInit(0.U(32.W))

val upReg = RegInit(true.B)

when (modulationReg < FREQ.U && upReg) { 
  modulationReg := modulationReg + 1.U
} .elsewhen (modulationReg === FREQ.U && upReg) { 
  upReg := false.B
} .elsewhen (modulationReg > 0.U && !upReg) { 
  modulationReg := modulationReg - 1.U
} .otherwise { // 0 
  upReg := true.B
}

// divide modReg by 1024 (about the 1 kHz)
val sig = pwm(MAX, modulationReg >> 10)
```

---

# **6.3 Shift Registers**

![bg vertical](./image/fake_image.png)

![bg height:300px](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-26_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_2.15.18.png)

```scala
val shiftReg = Reg(UInt(4.W)) 
shiftReg := Cat(shiftReg(2, 0), din) 
val dout = shiftReg (3)
```

---

## **6.3.1 Shift Register with Parallel Output**

![bg vertical](./image/fake_image.png)

![bg height:300px](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-26_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_2.16.40.png)

```scala
val outReg = RegInit(0.U(4.W)) 
outReg := Cat(serIn, outReg(3, 1)) 
val q = outReg
```

---

## **6.3.2 Shift Register with Parallel Load**

![bg vertical](./image/fake_image.png)

![bg height:300px](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-26_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_2.16.52.png)

```scala
val loadReg = RegInit(0.U(4.W)) 
when (load) {
  loadReg := d 
} otherwise {
  loadReg := Cat(0.U, loadReg(3, 1))
}
val serOut = loadReg (0)                                                                                  
```

---

# **6.4 Memory**

![bg right:50% 90%](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-26_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_2.18.31.png)



```scala
class Memory() extends Module { 
  val io = IO(new Bundle {
    val rdAddr = Input(UInt(10.W)) 
    val rdData = Output(UInt(8.W)) 
    val wrEna = Input(Bool())
    val wrData = Input(UInt(8.W)) 
    val wrAddr = Input(UInt(10.W))
  })
  val mem = SyncReadMem(1024, UInt(8.W)) 
 
  io.rdData := mem.read(io.rdAddr)
  
  when(io.wrEna) { 
    mem.write(io.wrAddr, io.wrData)
  } 
}
```

---

# **6.4 Memory**

![bg right:30% 90%](./image/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2022-06-26_%E1%84%8B%E1%85%A9%E1%84%8C%E1%85%A5%E1%86%AB_2.21.10.png)

```scala
class ForwardingMemory() extends Module { 
  val io = IO(new Bundle {
    val rdAddr = Input(UInt(10.W))
    val rdData = Output(UInt(8.W))
    val wrEna = Input(Bool())
    val wrData = Input(UInt(8.W))
    val wrAddr = Input(UInt(10.W))
  })
  val mem = SyncReadMem(1024, UInt(8.W))

  val wrDataReg = RegNext(io.wrData)
  val doForwardReg = RegNext(io.wrAddr === io.rdAddr && io.wrEna)
  
  val memData = mem.read(io.rdAddr)

  when(io.wrEna) { 
    mem.write(io.wrAddr, io.wrData)
  }
  
  io.rdData := Mux(doForwardReg , wrDataReg , memData) 
}
```

---

# **6.5 Exercise**

마지막 연습에서 7-세그먼트 인코더를 사용하고 4비트 카운터를 입력으로 추가하여 디스플레이를 0에서 F로 전환합니다. 이 카운터를 FPGA 보드의 클록에 직접 연결하면 16개의 숫자가 모두 겹친 것을 볼 수 있습니다( 7개의 세그먼트가 모두 켜집니다). 따라서 계산 속도를 늦출 필요가 있습니다. 500밀리초마다 단일 주기 틱 신호를 생성할 수 있는 다른 카운터를 만듭니다. 이 신호를 4비트 카운터에 대한 활성화 신호로 사용합니다.

생성기 함수로 PWM 파형을 구성하고 함수(삼각형 또는 사인 함수)로 임계값을 설정합니다. 위아래로 세어 삼각함수를 만들 수 있습니다. 몇 줄의 Scala 코드로 생성할 수 있는 조회 테이블을 사용하는 sinus 함수(섹션 10.3 참조). 변조된 PWM 기능을 사용하여 FPGA 보드에서 LED를 구동합니다. PWM 신호의 주파수는 얼마입니까? 드라이버가 실행하는 주파수는 무엇입니까?

디지털 디자인은 종종 종이에 회로로 스케치됩니다. 모든 세부 정보를 표시할 필요는 없습니다. 이 책의 그림과 같이 블록 다이어그램을 사용합니다. 회로의 도식적 표현과 끌 설명 사이를 유창하게 번역할 수 있는 것은 중요한 기술입니다. 다음 회로에 대한 블록 다이어그램을 스케치하십시오.

---

```scala
val dout = WireDefault(0.U)

switch(sel) {
  is(0.U) { dout := 0.U } 
  is(1.U) { dout := 11.U } 
  is(2.U) { dout := 22.U } 
  is(3.U) { dout := 33.U } 
  is(4.U) { dout := 44.U } 
  is(5.U) { dout := 55.U }
}
```

---

다음은 레지스터를 포함하는 조금 더 복잡한 회로입니다.

```scala
val regAcc = RegInit(0.U(8.W))

switch(sel) {
  is(0.U) { regAcc := regAcc}
  is(1.U) { regAcc := 0.U}
  is(2.U) { regAcc := regAcc + din} 
  is(3.U) { regAcc := regAcc - din}
}
```