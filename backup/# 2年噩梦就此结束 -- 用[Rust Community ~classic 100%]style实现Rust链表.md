# 困扰了我两年的Rust链表难题在此划上句号(喵～!)
myself:
![Rexim is Cute]("https://avatars.githubusercontent.com/u/115876698?v=4")
## 本人难度评价
若用c/c++,python,java实现都很简单,即使是c/c++手动释放可控的堆内存也行(C++智能指针yyds) -> c++11引入
而c实现又足够轻量
Java和pyhon自带gc功能而且在现在能胜任很大的数据流冲击,gc间隔造成的性能问题也不断解决,java和kotlin在这些年无畏并发有着很优雅的解决办法
用Rust难度会指数上升,Rust是偏函数式编程的,很多时候是像Haskell那样晦涩难懂的,我本人是命令行式编程和面向对象之后学的函数式编程喵～！可爱捏
Rust里的所有权,迭代器,引用与借用,错误处理,泛型，生命周期，组织源码test等让人烦恼,但是在我的不断练习和磨练，复习函数式编程成功实现了喵～！
## 项目地址:![可爱喵,欢迎参考](https://github.com/Poss-Studio/Rust_LinkList/blob/main/src/main.rs)

``` rust
use std::fmt;

// ---------- 节点定义 ----------
struct Node<T> {
    val: T,
    next: Option<Box<Node<T>>>,
}

impl<T> Node<T> {
    fn new(val: T) -> Self {
        Node { val, next: None }
    }
}

// ---------- 单链表 ----------
pub struct LinkedList<T> {
    head: Option<Box<Node<T>>>,
    len: usize,
}

impl<T> LinkedList<T> {
    /// 创建空链表
    pub fn new() -> Self {
        LinkedList { head: None, len: 0 }
    }

    /// 返回链表长度
    pub fn len(&self) -> usize {
        self.len
    }

    /// 判断是否为空
    pub fn is_empty(&self) -> bool {
        self.len == 0
    }

    // ---------- 头部操作 ----------
    /// 头部插入
    pub fn push_front(&mut self, val: T) {
        let new_node = Box::new(Node {
            val,
            next: self.head.take(),
        });
        self.head = Some(new_node);
        self.len += 1;
    }

    /// 头部弹出
    pub fn pop_front(&mut self) -> Option<T> {
        self.head.take().map(|node| {
            self.head = node.next;
            self.len -= 1;
            node.val
        })
    }

    /// 查看头部元素（不可变）
    pub fn peek_front(&self) -> Option<&T> {
        self.head.as_ref().map(|node| &node.val)
    }

    /// 查看头部元素（可变）
    pub fn peek_front_mut(&mut self) -> Option<&mut T> {
        self.head.as_mut().map(|node| &mut node.val)
    }

    // ---------- 尾部操作 ----------
    /// 尾部插入
    pub fn push_back(&mut self, val: T) {
        let new_node = Box::new(Node { val, next: None });

        if self.head.is_none() {
            self.head = Some(new_node);
        } else {
            let mut cur = self.head.as_mut().unwrap();
            while let Some(ref mut next) = cur.next {
                cur = next;
            }
            cur.next = Some(new_node);
        }
        self.len += 1;
    }

    /// 尾部弹出
    pub fn pop_back(&mut self) -> Option<T> {
        if self.head.is_none() {
            return None;
        }
        // 只有一个节点
        if self.head.as_ref().unwrap().next.is_none() {
            return self.pop_front();
        }

        let mut cur = self.head.as_mut().unwrap();
        while let Some(next) = cur.next.as_mut() {
            if next.next.is_none() {
                // 取出尾节点
                let tail = cur.next.take();
                self.len -= 1;
                return tail.map(|boxed| boxed.val);
            }
            cur = cur.next.as_mut().unwrap();
        }
        None
    }

    /// 查看尾部元素（不可变）
    pub fn peek_back(&self) -> Option<&T> {
        let mut cur = self.head.as_ref();
        while let Some(node) = cur {
            if node.next.is_none() {
                return Some(&node.val);
            }
            cur = node.next.as_ref();
        }
        None
    }

    /// 查看尾部元素（可变）
    pub fn peek_back_mut(&mut self) -> Option<&mut T> {
        let mut cur = self.head.as_mut();
        while let Some(node) = cur {
            if node.next.is_none() {
                return Some(&mut node.val);
            }
            cur = node.next.as_mut();
        }
        None
    }

    // ---------- 按值查找、删除、更新 ----------
    /// 查找值是否存在（需要 T: PartialEq）
    pub fn contains(&self, val: &T) -> bool
    where
        T: PartialEq,
    {
        let mut cur = self.head.as_ref();
        while let Some(node) = cur {
            if &node.val == val {
                return true;
            }
            cur = node.next.as_ref();
        }
        false
    }

    /// 删除第一个匹配的值（需要 T: PartialEq）
    pub fn remove(&mut self, val: &T) -> bool
    where
        T: PartialEq,
    {
        // 处理头节点
        if let Some(node) = self.head.as_mut() {
            if &node.val == val {
                self.head = node.next.take();
                self.len -= 1;
                return true;
            }
        }

        let mut cur = self.head.as_mut();
        while let Some(node) = cur {
            if let Some(ref mut next_node) = node.next {
                if &next_node.val == val {
                    node.next = next_node.next.take();
                    self.len -= 1;
                    return true;
                }
            }
            cur = node.next.as_mut();
        }
        false
    }

    /// 更新第一个匹配的值（需要 T: PartialEq）
    pub fn update(&mut self, old_val: &T, new_val: T) -> bool
    where
        T: PartialEq,
    {
        let mut cur = self.head.as_mut();
        while let Some(node) = cur {
            if &node.val == old_val {
                node.val = new_val;
                return true;
            }
            cur = node.next.as_mut();
        }
        false
    }

    // ---------- 按索引操作 ----------
    /// 按索引获取元素（不可变）
    pub fn get(&self, index: usize) -> Option<&T> {
        if index >= self.len {
            return None;
        }
        let mut cur = self.head.as_ref();
        for _ in 0..index {
            cur = cur?.next.as_ref();
        }
        cur.map(|node| &node.val)
    }

    /// 按索引获取元素（可变）
    pub fn get_mut(&mut self, index: usize) -> Option<&mut T> {
        if index >= self.len {
            return None;
        }
        let mut cur = self.head.as_mut();
        for _ in 0..index {
            cur = cur?.next.as_mut();
        }
        cur.map(|node| &mut node.val)
    }

    /// 在指定索引插入（0 为头部，len 为尾部）
    pub fn insert_at(&mut self, index: usize, val: T) -> bool {
        if index > self.len {
            return false;
        }
        if index == 0 {
            self.push_front(val);
            return true;
        }

        let mut cur = self.head.as_mut();
        for _ in 0..(index - 1) {
            match cur {
                Some(node) => cur = node.next.as_mut(),
                None => return false, // 不会发生
            }
        }

        if let Some(prev) = cur {
            let new_node = Box::new(Node {
                val,
                next: prev.next.take(),
            });
            prev.next = Some(new_node);
            self.len += 1;
            true
        } else {
            false
        }
    }

    /// 按索引删除，返回被删除的值
    pub fn remove_at(&mut self, index: usize) -> Option<T> {
        if index >= self.len {
            return None;
        }
        if index == 0 {
            return self.pop_front();
        }

        let mut cur = self.head.as_mut();
        for _ in 0..(index - 1) {
            cur = cur?.next.as_mut();
        }

        if let Some(prev) = cur {
            if let Some(mut node) = prev.next.take() {
                prev.next = node.next.take();
                self.len -= 1;
                return Some(node.val);
            }
        }
        None
    }

    // ---------- 迭代器 ----------
    pub fn iter(&self) -> Iter<T> {
        Iter {
            next: self.head.as_deref(),
        }
    }

    pub fn iter_mut(&mut self) -> IterMut<T> {
        IterMut {
            next: self.head.as_deref_mut(),
        }
    }
}

// ---------- 不可变迭代器 ----------
pub struct Iter<'a, T> {
    next: Option<&'a Node<T>>,
}

impl<'a, T> Iterator for Iter<'a, T> {
    type Item = &'a T;

    fn next(&mut self) -> Option<Self::Item> {
        self.next.map(|node| {
            self.next = node.next.as_deref();
            &node.val
        })
    }
}

// ---------- 可变迭代器 ----------
pub struct IterMut<'a, T> {
    next: Option<&'a mut Node<T>>,
}

impl<'a, T> Iterator for IterMut<'a, T> {
    type Item = &'a mut T;

    fn next(&mut self) -> Option<Self::Item> {
        self.next.take().map(|node| {
            self.next = node.next.as_deref_mut();
            &mut node.val
        })
    }
}

// ---------- 所有权迭代器 ----------
impl<T> IntoIterator for LinkedList<T> {
    type Item = T;
    type IntoIter = IntoIter<T>;

    fn into_iter(self) -> Self::IntoIter {
        IntoIter { list: self }
    }
}

pub struct IntoIter<T> {
    list: LinkedList<T>,
}

impl<T> Iterator for IntoIter<T> {
    type Item = T;

    fn next(&mut self) -> Option<Self::Item> {
        self.list.pop_front()
    }
}

// ---------- 实现 Display ----------
impl<T: fmt::Display> fmt::Display for LinkedList<T> {
    fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
        let mut parts = Vec::new();
        for val in self.iter() {
            parts.push(format!("{}", val));
        }
        write!(f, "[{}]", parts.join(" -> "))
    }
}

// ---------- Drop：手动实现防止长链表栈溢出 ----------
impl<T> Drop for LinkedList<T> {
    fn drop(&mut self) {
        let mut cur = self.head.take();
        while let Some(mut node) = cur {
            cur = node.next.take();
        }
    }
}

// ---------- 测试 ----------
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_push_pop_front() {
        let mut list = LinkedList::new();
        list.push_front(1);
        list.push_front(2);
        assert_eq!(list.pop_front(), Some(2));
        assert_eq!(list.pop_front(), Some(1));
        assert_eq!(list.pop_front(), None);
    }

    #[test]
    fn test_push_pop_back() {
        let mut list = LinkedList::new();
        list.push_back(1);
        list.push_back(2);
        list.push_back(3);
        assert_eq!(list.pop_back(), Some(3));
        assert_eq!(list.pop_back(), Some(2));
        assert_eq!(list.pop_back(), Some(1));
        assert_eq!(list.pop_back(), None);
    }

    #[test]
    fn test_remove() {
        let mut list = LinkedList::new();
        list.push_back(1);
        list.push_back(2);
        list.push_back(3);
        assert!(list.remove(&2));
        assert_eq!(list.iter().collect::<Vec<_>>(), vec![&1, &3]);
        assert!(!list.remove(&4));
    }

    #[test]
    fn test_update() {
        let mut list = LinkedList::new();
        list.push_back(1);
        list.push_back(2);
        list.update(&2, 99);
        assert_eq!(list.get(1), Some(&99));
    }

    #[test]
    fn test_insert_at() {
        let mut list = LinkedList::new();
        list.push_back(1);
        list.push_back(3);
        list.insert_at(1, 2);
        assert_eq!(list.iter().collect::<Vec<_>>(), vec![&1, &2, &3]);
        list.insert_at(0, 0);
        assert_eq!(list.iter().collect::<Vec<_>>(), vec![&0, &1, &2, &3]);
        list.insert_at(4, 4);
        assert_eq!(list.iter().collect::<Vec<_>>(), vec![&0, &1, &2, &3, &4]);
    }

    #[test]
    fn test_remove_at() {
        let mut list = LinkedList::new();
        list.push_back(1);
        list.push_back(2);
        list.push_back(3);
        assert_eq!(list.remove_at(1), Some(2));
        assert_eq!(list.iter().collect::<Vec<_>>(), vec![&1, &3]);
        assert_eq!(list.remove_at(0), Some(1));
        assert_eq!(list.iter().collect::<Vec<_>>(), vec![&3]);
        assert_eq!(list.remove_at(0), Some(3));
        assert!(list.is_empty());
    }

    #[test]
    fn test_peek() {
        let mut list = LinkedList::new();
        list.push_back(1);
        list.push_back(2);
        assert_eq!(list.peek_front(), Some(&1));
        assert_eq!(list.peek_back(), Some(&2));
        *list.peek_front_mut().unwrap() = 9;
        assert_eq!(list.peek_front(), Some(&9));
    }

    #[test]
    fn test_into_iter() {
        let mut list = LinkedList::new();
        list.push_back(1);
        list.push_back(2);
        let collected: Vec<i32> = list.into_iter().collect();
        assert_eq!(collected, vec![1, 2]);
    }
}

fn main() {
    let mut list = LinkedList::new();

    // 插入
    list.push_back(10);
    list.push_back(20);
    list.push_front(5);
    println!("链表: {}", list); // [5 -> 10 -> 20]

    // 更新
    list.update(&20, 99);
    println!("更新 20 -> 99 后: {}", list); // [5 -> 10 -> 99]

    // 删除
    list.remove(&10);
    println!("删除 10 后: {}", list); // [5 -> 99]

    // 按索引插入
    list.insert_at(1, 88);
    println!("在索引 1 插入 88: {}", list); // [5 -> 88 -> 99]

    // 按索引删除
    list.remove_at(0);
    println!("删除索引 0 后: {}", list); // [88 -> 99]

    // 迭代
    for val in list.iter() {
        println!("iter: {}", val);
    }
}
```
# Author :Poss-Rexim
where can find  me?
![喵~!以后就是朋友了](ddt1472582022@outlook.com)