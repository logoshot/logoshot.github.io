---
title: "An Rust Example in codeforces"
date: 2021-12-08T22:49:30+08:00
draft: true
---

use rust in its own way
from [cf1612E's solution](https://codeforces.com/contest/1612/problem/E)

``` rust
#![allow(dead_code, unused_macros, unused_imports)]
use std::{
    cell::{Cell, RefCell, UnsafeCell}, 
    cmp::{Ordering, Reverse, max, min}, 
    collections::{
        BTreeMap, 
        BTreeSet, 
        BinaryHeap, 
        HashMap, 
        HashSet, 
        VecDeque, 
        hash_map::{DefaultHasher, RandomState}},
    error::Error, 
    fmt::{Display, Write as FmtWrite}, 
    hash::{BuildHasher, Hash, Hasher}, 
    io::{BufWriter, Read, Stdin, Stdout, Write}, 
    iter::{FromIterator, Peekable}, 
    mem::swap, 
    ops::*, 
    process::exit, 
    rc::Rc, 
    str::{FromStr, from_utf8_unchecked}, 
    time::{Duration, Instant}};
 
const IO_BUF_SIZE: usize = 1 << 16;
type Input = Scanner<Stdin>;
type Output = BufWriter<Stdout>;
fn _init_input() -> Input { Scanner::new(std::io::stdin()) }
fn _init_output() -> Output { BufWriter::with_capacity(IO_BUF_SIZE, std::io::stdout()) }
 
#[repr(transparent)] struct Unsync<T>(T);
unsafe impl<T> Sync for Unsync<T> {}
 
type BadLazy<T> = Unsync<UnsafeCell<Option<T>>>;
impl<T> BadLazy<T> {
    const fn new() -> Self { Self(UnsafeCell::new(None)) }
}
 
static INPUT: BadLazy<Input> = BadLazy::new();
static OUTPUT: BadLazy<Output> = BadLazy::new();
 
fn inp<F: FnOnce(&mut Input) -> R, R>(f: F) -> R {
    unsafe { f((&mut *INPUT.0.get()).get_or_insert_with(_init_input)) }
}
fn out<F: FnOnce(&mut Output) -> R, R>(f: F) -> R {
    unsafe { f((&mut *OUTPUT.0.get()).get_or_insert_with(_init_output)) }
}
 
macro_rules! read {
    () => { read() };
    ($t: ty) => { read::<$t>() };
    ($t: ty, $($tt: ty),*) => { (read::<$t>(), $(read::<$tt>(),)*) };
    [$t: ty; $n: expr] => { read_vec::<$t>($n) };
}
macro_rules! println { 
    () => { out(|x| { let _ = writeln!(x); }) };
    ($exp: expr) => { out(|x| { let _ = writeln!(x, "{}", $exp); }) }; 
    ($fmt: expr, $($arg : tt )*) => { out(|x| { let _ = writeln!(x, $fmt, $($arg)*); }) }
}
macro_rules! print { 
    ($exp: expr) => { out(|x| { let _ = write!(x, "{}", $exp); }) }; 
    ($fmt: expr, $($arg : tt )*) => { out(|x| { let _ = write!(x, $fmt, $($arg)*); }) }
}
 
fn out_flush() { out(|x| { let _ = x.flush(); }); }
 
fn input_is_eof() -> bool { inp(|x| x.eof()) }
fn read_byte() -> u8 { inp(|x| x.byte()) }
fn read_bytes_no_skip(n: usize) -> Vec<u8> { inp(|x| x.bytes_no_skip(n)) }
fn read_bytes(n: usize) -> Vec<u8> { inp(|x| x.bytes(n)) }
fn read_bytes2(n: usize, m: usize) -> Vec<Vec<u8>> { inp(|x| x.bytes2(n, m)) }
fn read_token() -> Vec<u8> { inp(|x| x.token_bytes()) }
fn read_token_str() -> String { unsafe { String::from_utf8_unchecked(read_token()) } }
fn read_line() -> Vec<u8> { inp(|x| x.line_bytes()) }
fn read_line_str() -> String { unsafe { String::from_utf8_unchecked(read_line()) } }
fn read<T: FromStr>() -> T { read_token_str().parse::<T>().ok().expect("failed parse") }
fn read_vec<T: FromStr>(n: usize) -> Vec<T> { (0..n).map(|_| read()).collect() }
fn read_vec2<T: FromStr>(n: usize, m: usize) -> Vec<Vec<T>> { (0..n).map(|_| read_vec(m)).collect() }
 
struct Scanner<R: Read> {
    src: R,
    _buf: Vec<u8>,
    _pt: usize, // pointer
    _rd: usize, // bytes read
}
 
#[allow(dead_code)]
impl<R: Read> Scanner<R> {
    fn new(src: R) -> Scanner<R> {
        Scanner { src, _buf: vec![0; IO_BUF_SIZE], _pt: 1, _rd: 1 }
    }
 
    fn _check_buf(&mut self) {
        if self._pt == self._rd {
            self._rd = self.src.read(&mut self._buf).unwrap_or(0);
            self._pt = (self._rd == 0) as usize;
        }
    }
 
    // returns true if end of file
    fn eof(&mut self) -> bool {
        self._check_buf();
        self._rd == 0
    }
 
    // filters \r, returns \0 if eof
    fn byte(&mut self) -> u8 {
        loop {
            self._check_buf();
            if self._rd == 0 { return 0; }
            let res = self._buf[self._pt];
            self._pt += 1;
            if res != b'\r' { return res; }
        }
    }
 
    fn bytes_no_skip(&mut self, n: usize) -> Vec<u8> { (0..n).map(|_| self.byte()).collect() }
    fn bytes(&mut self, n: usize) -> Vec<u8> {
        let res = self.bytes_no_skip(n);
        self.byte();
        res
    }
    fn bytes2(&mut self, n: usize, m: usize) -> Vec<Vec<u8>> { (0..n).map(|_| self.bytes(m)).collect() }
 
    fn token_bytes(&mut self) -> Vec<u8> {
        let mut res = Vec::new();
        let mut c = self.byte();
        while c <= b' ' {
            if c == b'\0' { return res; }
            c = self.byte();
        }
        loop {
            res.push(c);
            c = self.byte();
            if c <= b' ' { return res; }
        }
    }
 
    fn line_bytes(&mut self) -> Vec<u8> {
        let mut res = Vec::new();
        let mut c = self.byte();
        while c != b'\n' && c != b'\0' {
            res.push(c);
            c = self.byte();
        }
        res
    }
}
 
trait JoinToStr { 
    fn join_to_str(self, sep: &str) -> String;
    fn concat_to_str(self) -> String;
}
impl<T: Display, I: Iterator<Item = T>> JoinToStr for I { 
    fn join_to_str(mut self, sep: &str) -> String {
        match self.next() {
            Some(first) => {
                let mut res = first.to_string();
                while let Some(item) = self.next() {
                    res.push_str(sep);
                    res.push_str(&item.to_string());
                }
                res
            }
            None => { String::new() }
        }
    }
 
    fn concat_to_str(self) -> String {
        let mut res = String::new();
        for item in self { res.push_str(&item.to_string()); }
        res
    }
}
trait AsStr { fn as_str(&self) -> &str; }
impl AsStr for [u8] { fn as_str(&self) -> &str {std::str::from_utf8(self).expect("attempt to convert non-UTF8 byte string.")} }
 
macro_rules! veci {
    ($n:expr , $i:ident : $gen:expr) => {{
        let _veci_n = $n;
        let mut _veci_list = Vec::with_capacity(_veci_n);
        for $i in 0.._veci_n {
            _veci_list.push($gen);
        }
        _veci_list
    }};
    ($n:expr , $gen:expr) => { veci!($n, _veci_: $gen) }
}
 
fn abs_diff<T: Sub<Output = T> + PartialOrd>(x: T, y: T) -> T {
    if x < y { y - x } else { x - y }
}
 
trait CommonNumExt {
    fn div_ceil(self, b: Self) -> Self;
    fn div_floor(self, b: Self) -> Self;
    fn gcd(self, b: Self) -> Self;
    fn highest_one(self) -> Self;
    fn lowest_one(self) -> Self;
}
 
macro_rules! impl_common_num_ext {
    ($($ix:tt = $ux:tt),*) => {
        $(
            impl CommonNumExt for $ux {
                fn div_ceil(self, b: Self) -> Self {
                    let q = self / b; let r = self % b;
                    if r != 0 { q + 1 } else { q }
                }
                fn div_floor(self, b: Self) -> Self { self / b }
                fn gcd(self, mut b: Self) -> Self {
                    let mut a = self; while a != 0 { swap(&mut a, &mut b); a %= b; }
                    b
                }
                #[inline] fn highest_one(self) -> Self { 
                    if self == 0 { 0 } else { const ONE: $ux = 1; ONE.rotate_right(1) >> self.leading_zeros() } 
                }
                #[inline] fn lowest_one(self) -> Self { self & self.wrapping_neg() }
            }
 
            impl CommonNumExt for $ix {
                fn div_ceil(self, b: Self) -> Self {
                    let q = self / b; let r = self % b;
                    if self ^ b >= 0 && r != 0 { q + 1 } else { q }
                }
                fn div_floor(self, b: Self) -> Self { 
                    let q = self / b; let r = self % b;
                    if self ^ b < 0 && r != 0 { q - 1 } else { q }
                }
                fn gcd(self, mut b: Self) -> Self {
                    let mut a = self; while a != 0 { swap(&mut a, &mut b); a %= b; }
                    b.abs()
                }
                #[inline] fn highest_one(self) -> Self { (self as $ux).highest_one() as _ }
                #[inline] fn lowest_one(self) -> Self { self & self.wrapping_neg() }
            }
        )*
    }
}
impl_common_num_ext!(i8 = u8, i16 = u16, i32 = u32, i64 = u64, i128 = u128, isize = usize);
 
trait ChMaxMin<T> {
    fn set_max(&mut self, v: T) -> bool;
    fn set_min(&mut self, v: T) -> bool;
}
impl<T: PartialOrd> ChMaxMin<T> for T {
    fn set_max(&mut self, v: T) -> bool { if v > *self { *self = v; true } else { false } }
    fn set_min(&mut self, v: T) -> bool { if v < *self { *self = v; true } else { false } }
}
 
// * end commons * //
 
 
#[allow(non_snake_case, non_upper_case_globals)]
fn main() {
    let num_cases: usize = 1;//read();
 
    for _case_num in 1..=num_cases {
        let n = read!(usize);
 
        let A = veci!{n, read!(usize, usize)};
        let M = A.iter().map(|x| x.0).max().unwrap();
        let K = A.iter().map(|x| x.1).max().unwrap();
 
        let mut ans = vec![];
        let mut score = (0, 1);
        for t in 1..=min(M, K) {
            let mut S = veci!{M+1, i: (i, 0)};
            for &(m, k) in &A {
                S[m].1 += min(k, t);
            }
 
            S.select_nth_unstable_by_key(t-1, |x| Reverse(x.1));
            let s = S[..t].iter().map(|x| x.1).sum::<usize>();
 
            if s * score.1 > score.0 * t {
                score = (s, t);
                ans = S[..t].iter().map(|x| x.0).collect()
            }
        }
 
        println!(ans.len());
        println!(ans.iter().join_to_str(" "));
    }
 
    out_flush();
}
```

