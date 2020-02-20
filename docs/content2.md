class: middleish, center
![ferris](img/ferris.gif)
--

# Enough Rust to be dangerous

---
.chapter[Agenda]
# Agenda

### 1. Recap
0. where are you?
    * follow up questions since last time please
2. using libraries
2. testing
--

### 2. More Rust
1. "Object Orientation"
1. Fearless Concurrency
--

### we build something

---
class: middle
# Recap
### Where are you now?

---
class: middle
.chapter[Recap]
## the standard library

find it at [std.rs](https://std.rs/)

* `std::convert`
* `std::collections`
* `std::error`
* `std::[string, option, result, mem, future]`;

---
class: middle
.chapter[Recap]
## crates

* full registry at [crates.io](https://crates.io)
* categorised overview at [libs.rs](https://lib.rs/)
* find documentation at [docs.rs](https://docs.rs)

---
class: middle
## cargo
* `cargo --list`
* `cargo [doc|test|check|install]`

### extra tools
* `cargo-edit`, `cargo-watch`, `cargo-tree`, `cargo-modules`
* rustfmt `cargo fmt`
* clippy: `cargo clippy` 

---
class: middle
.chapter[Recap]
## testing

* in place
* modules
* tests folder

---
class: middle
.chapter[testing]
### in place
```rust
#[test]
fn parse_property_with_breaks() {
    // ...
    assert_eq!(property(sample_0), Ok((&[][..], expectation)));
}

pub fn property<'a>(i: &'a [u8]) -> IResult<&'a [u8], Property> {
  // ... 
}
```

---
class: middle
.chapter[testing]
### test module

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn parse_with_hande_behind_back() {
        // ..
}
```

---
class: middle
.chapter[Recap]
## project folder structure

<small>
```
├── Cargo.toml
├── LICENSE
├── README.md
├── examples
│   ├── ....
│   ├── urgency.rs
│   └── wait_for_closing.rs
├── src
│   ├── error.rs
│   ├── hints
│   │   ├── mod.rs
│   │   └── tests.rs
│   ├── ....
│   └── lib.rs
└── tests
    ├── conversion.rs
    └── regression.rs
```
</small>

---
class: middle
# More Rust

---
class: middleish
## Object Orientation?
* no classes / 
--
no inheritance
--


- `impl` blocks for associated functions (methods)
--

- **traits**
- **generics & constraints**
- **trait objects**

---
.chapter[Traits]
### Interview Question

#### You want to control two different things with one remote.
How?
--

```rust
#[derive(Debug, Default)]
pub struct Stereo {
    volume: u16,
}
```
--
```rust
impl Stereo {
    fn volume_up(&mut self) {
        self.volume = cmp::min(self.volume + 1, 100);
    }
    fn volume_down(&mut self) {
        self.volume = cmp::max(self.volume - 1, 0);
    }
}
```

---
.chapter[Traits]
### Interview Question

```rust
#[derive(Debug, Default)]
pub struct HotTub {
    bubbles_on: bool,
    heat_on: bool,
}
```
--
```rust
impl HotTub {
    fn enable_bubbles(&mut self) {
        self.bubbles_on = true
    }
    fn disable_bubbles(&mut self) {
        self.bubbles_on = false
    }
    fn set_temperature(&mut self) {
        self.heat_on = true
    }
    fn turn_off_heat(&mut self) {
        self.heat_on = false
    }
}
```

---
.chapter[Traits]
### Remote Control
```rust
#[derive(Debug, Default)]
pub struct RemoteControl;
```
--
```rust
impl RemoteControl {
    pub fn on(device) {

    }
    pub fn off(device) {
        
    }
}
```

---
.chapter[Traits]
### Remote Control
```rust
#[derive(Debug, Default)]
pub struct RemoteControl;
```
```rust
impl RemoteControl {
    pub fn on   (device:       ) {

    }
    pub fn off   (device:       ) {
        
    }
}
```

---
.chapter[Traits]
### Remote Control
```rust
#[derive(Debug, Default)]
pub struct RemoteControl;
```
```rust
impl RemoteControl {
    pub fn on<T>(device: &mut T) {

    }
    pub fn off<T>(device: &mut T) {

    }
}
```

--

```rust
pub trait RemoteControllable {
    fn on_button(&mut self);
    fn off_button(&mut self);
}
```

---
.chapter[Traits]
### Remote Control
```rust
#[derive(Debug, Default)]
pub struct RemoteControl;
```
```rust
impl RemoteControl {
    pub fn on<T>(device: &mut T) {
        device.on_button_pressed();
    }
    pub fn off<T>(device: &mut T) {
        device.off_button_pressed();
    }
}
```

```rust
pub trait RemoteControllable {
    fn on_button(&mut self);
    fn off_button(&mut self);
}
```

---
.chapter[Traits]
### Remote Control
```rust
#[derive(Debug, Default)]
pub struct RemoteControl;
```
```rust
impl RemoteControl {
    pub fn on<T: RemoteControllable>(device: &mut T) {
        device.on_button_pressed();
    }
    pub fn off<T: RemoteControllable>(device: &mut T) {
        device.off_button_pressed();
    }
}
```

```rust
pub trait RemoteControllable {
    fn on_button(&mut self);
    fn off_button(&mut self);
}
```

---
class: middle
.chapter[Traits]
### Remote Control

```rust
impl RemoteControllable for Stereo {
    fn on_button_pressed(&mut self) {
        self.volume_up();
    }

    fn off_button_pressed(&mut self) {
        self.volume_down();
    }
}
```
```rust
impl RemoteControllable for HotTub {
    fn on_button_pressed(&mut self) {
        self.enable_bubbles();
        self.set_temperature();
    }

    fn off_button_pressed(&mut self) {
        self.turn_off_heat();
        self.disable_bubbles();
    }
}
```
---
class: middle
.chapter[Traits]
### Remote Control

```rust
let mut hot_tub = HotTub::default();
let mut sound_blaster = Stereo::default();

RemoteControl::on(&mut hot_tub);
RemoteControl::on(&mut sound_blaster);
```
---

---
class: middle
## Featless Concurrency
* threads 
* synchronization by communication
* shared state


---
class: middle
.chapter[Fearless concurrency]
### Threads<sup>[*](https://doc.rust-lang.org/stable/book/ch16-01-threads.html)</sup>

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap();
}
```

---
.chapter[Fearless concurrency]
### sync by communication<sup>[*](https://doc.rust-lang.org/stable/book/ch16-02-message-passing.html)</sup>


```rust
use std::sync::mpsc::channel;
use std::thread;

let (sender, receiver) = channel();

// Spawn off an expensive computation
thread::spawn(move|| {
    sender.send(expensive_computation()).unwrap();
});

// Do some useful work for awhile

// Let's see what that answer was
println!("{:?}", receiver.recv().unwrap());
```

---
.chapter[Fearless concurrency]
### shared state<sup>[*](https://doc.rust-lang.org/stable/book/ch16-03-shared-state.html)</sup>

```rust
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];

    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();

            *num += 1;
        });
        handles.push(handle);
    }

```

---
class: middle
### Further
`Sync` and `Send` marker Traits

---
class: middle
# Hands On

