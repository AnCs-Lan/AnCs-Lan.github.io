---
title: rust日记：从堆实现到Dijkstra
description: 用rust写了一个堆，然后复刻了一下单源最短路
date: 2026-01-16 22:39:00
tags:
   - Rust
   - 算法
categories:
   - Rust
---
# Rust学习日记
## Heap初版:
　　在初学了一部分rust语法后，偶然想起来了以前打得很丑的板子，于是决定时隔多年重新复刻（复习）一下，希望写好看一点
　　于是在经历了一番折腾后写出了一个相对比较差劲的代码:  
### 总体代码
```Rust
struct Heap<T: Copy + Ord> {
    heap: Vec<T>,
    size: usize,
}

impl<T: Copy + Ord> Heap<T> {
    fn up(&mut self, idx: usize) {
        if idx == 0 {
            return;
        }
        let fa = (idx - 1) >> 1;
        if self.heap[fa] < self.heap[idx] {
            self.heap.swap(fa, idx);
            self.up(fa);
        }
    }

    fn down(&mut self, idx: usize) {
        let mut next: usize = idx;
        if self.size >= ((idx << 1) + 2) && self.heap[idx] < self.heap[(idx << 1) + 1] {
            next = (idx << 1) + 1;
        }
        if self.size >= ((idx << 1) + 3) && self.heap[next] < self.heap[(idx << 1) + 2] {
            next = (idx << 1) + 2;
        }
        if next != idx {
            self.heap.swap(idx, next);
            self.down(next);
        }
    }

    fn build_heap(&mut self) {
        if self.heap.is_empty() {
            return;
        }
        let last_fa = self.heap.len() >> 1;
        for i in (0..last_fa).rev() {
            self.down(i);
        }
    }

    pub fn new() -> Self {
        Self {
            heap: Vec::new(),
            size: 0,
        }
    }

    pub fn from_vec(input: Vec<T>) -> Self {
        let mut heap = Self {
            size: input.len(),
            heap: input,
        };
        heap.build_heap();
        heap
    }

    pub fn push(&mut self, num: T) {
        self.heap.push(num);
        self.size += 1;
        self.up(self.size - 1);
    }

    pub fn front(&self) -> T {
        self.heap[0]
    }

    pub fn pop(&mut self) {
        self.heap.swap(0, self.size - 1);
        self.size -= 1;
        self.down(0);
    }

    pub fn is_empty(&self) -> bool {
        if self.size == 0 {
            return true;
        }
        false
    }
}
```
　　代码本身就有很多问题,比如我把一些队列的函数命名混淆在了一块,同时还缺乏了查看长度的功能~~当然,我当时根本不知道还有这么个功能~~  
### new函数
　　在rust中，`new`函数的作用有些类似于 **C++** 中`class`的构造函数，主要职能就是构建一个结构体变量给调用者  
```Rust
    pub fn new() -> Self {
        Self {
            heap: Vec::new(),
            size: 0,
        }
    }

```
　　其中`Self`就是对于结构体本身的指代，在这里就相当于`Heap`，这样无论结构体改名叫什么，`Self`都可以发挥其作用，然后不打`;`就将这个表达式的返回值直接作为函数返回值返回回去了  
　　不要把`Self`和`self`搞混了，后者所指代的是调用函数者本身  

### 向上调整与向下调整：
　　个人感觉堆这个数据结构最核心的两个函数就是**向下调整**和**向上调整**了，其他实现都围绕其展开  
```Rust
    fn up(&mut self, idx: usize) {
        if idx == 0 {
            return;
        }
        let fa = (idx - 1) >> 1;
        if self.heap[fa] < self.heap[idx] {
            self.heap.swap(fa, idx);
            self.up(fa);
        }
    }

    fn down(&mut self, idx: usize) {
        let mut next: usize = idx;
        if self.size >= ((idx << 1) + 2) && self.heap[idx] < self.heap[(idx << 1) + 1] {
            next = (idx << 1) + 1;
        }
        if self.size >= ((idx << 1) + 3) && self.heap[next] < self.heap[(idx << 1) + 2] {
            next = (idx << 1) + 2;
        }
        if next != idx {
            self.heap.swap(idx, next);
            self.down(next);
        }
    }
```
#### 向上调整：
　　简单来说，**向上调整**就是把当前节点和其父节点进行比较，我们以大根堆为例，父节点的元素在满足堆结构的情况下应该是比子节点元素大的  
　　所以如果当前节点大于了父节点，就应该把它和父节点交换，而我们不能确保其父节点的父节点就大于该节点，所以我们还要继续跟踪  
　　因而当交换发生后，应该再去对父节点向上调整，要是也交换了，那重复之前的操作，也就是保持追踪某节点直到它符合堆结构  
　　当然，仅仅是向上调整自然是不能保证堆结构的，所以这个操作通常是在整体已经满足堆结构后，在底部插入了一个新元素时执行的  
#### 向下调整：
　　类比**向上调整**，还是以大根堆为例，所谓向下调整其实也就是把当前节点和其两个字节点进行比较，如果不满足堆结构，就把相对更大的那个子节点换上来，这样就保证了这三个节点是符合堆结构的  
　　但是交换后不能保证其字节点的字节点就小于当前节点，所以同理的要进行追踪，并复读上面的操作  
　　这样一来，假如其两个字节点本来是符合堆结构的，我们就将这个形状视作三个三角形，显然第一次调整后，上三角和没有被交换的那个三角是符合堆结构的  
　　那么当我们继续追踪该节点并继续判断是否交换并执行后，被交换的那个三角也就满足堆结构了，以此类推，就可以保证当前节点的整个子树是堆结构的  
　　所以继续类推，只要我们一直在一个下方满足堆结构的位置进行向下调整，就可以让下方节点逐渐全部满足堆结构，此时再继续向上移动，就可以让整个树都满足堆结构  
### push函数：
　　那么再来看看push部分，就很容易理解了：
```Rust
    pub fn push(&mut self, num: T) {
        self.heap.push(num);
        self.size += 1;
        self.up(self.size - 1);
    }
```
　　就像上面说的，如果树一开始是满足堆结构的，我们每次插入一个元素自然就只需要对该元素**向上调整**  
　　然后很显然，如果树一开始是空的，那其自然是符合堆结构的，所以我们从空树开始压入元素到底部（由于使用完全二叉树存储，每次新元素只需要插入到末尾就刚好是整个树的末尾），并一步步**向上调整**，就能让树保持堆结构  
　　所以只要不断压入并向上调整，就能让树一直保持堆结构从而实现维护与插入操作  
### pop函数：
　　一个数据结构通常要实现**增删查改**，四个操作，但是堆的普通情况是不能改的，而且查也只能查第一个元素，所以就不多赘述，我们直接看删除操作  
```Rust
    pub fn pop(&mut self) {
        self.heap.swap(0, self.size - 1);
        self.size -= 1;
        self.down(0);
    }
```
　　当然，堆的删除其实通常也是只能删除第一个元素然后把其他元素推到上面来  
　　而我们的实现方式，其实就是把最后一个元素与第一个元素交换位置，然后对交换后的第一个元素进行**向下调整**，就像上面说的，毕竟其他元素都没发生改变，自然是符合堆结构的，所以只要把当前的第一个元素进行**向下调整**，就能维护整个树的堆结构  
　　这就模拟出了堆内的其他元素挤上来的样子  
　　然后再把堆表示大小的数据-1，表示删除末尾元素~~这其实是普通数组的写法，我的脑子还留在Ｃ语言~~  
　　当然，其实这个函数写得有很严重的问题，导致一个堆其实只能一次性使用，后面会进行修改  
### from_vec函数:
　　最开始的想法其实是读入数据然后压入堆内,毕竟我从前一直是这么做的,但是当时那天看了视频还知道了一个把树调整为堆的方法,只不过一开始没当回事,直到后来问了AI才意识到,如果我把一个数列压入堆内,其实更好的方案是直接调整数列  
　　于是,`from_vec`函数就诞生了,~~虽然中间还写了一个没必要的`build`函数~~  
```Rust
    fn build_heap(&mut self) {
        if self.heap.is_empty() {
            return;
        }
        let last_fa = self.heap.len() >> 1;
        for i in (0..last_fa).rev() {
            self.down(i);
        }
    }

    pub fn from_vec(input: Vec<T>) -> Self {
        let mut heap = Self {
            size: input.len(),
            heap: input,
        };
        heap.build_heap();
        heap
    }
```
　　总体的逻辑就是从最后一个父节点开始反向遍历,然后对每一个遍历到的节点进行向下调整,这样可以保证每次遍历到的节点其下方都是符合堆结构的(即其子节点<=(或者>=)当前节点,叶子天然满足这一点),同时这也就让当前节点向下调整后就能符合堆结构  
#### 堆排序
　　这样一轮下来,用O(n)的时间复杂度就完成了堆化,此时如果把每一个节点都pop出来,按顺序存入一个数组(需要O(nlogn)的时间复杂度),就完成了**堆排序**  
### Rust的实现过程中遇到的问题
　　扯了一大堆的数据结构本身，然而这其实是Rust学习笔记~~我去不早说~~  
　　前面一通解释的核心原因是我自己也快忘了（）  
　　所以接下来进入正题：  
#### 所有权不便给出
　　Rust的特色，不得不品鉴的一环，由于在写代码的过中，我所设想的**查**环节，是把数据整个返回回来，因为我在日常使用堆的时候都是边查边扔的，如果所有权不转移，借用就会对删除造成掣肘  
　　但是将`Vec`的单个元素所有权给出，无论从整体运作还是局部实现都是不太可行的，所以我当时的解决方案是：加一个**trait**  
　　我在结构体的**泛型**限制中，加入了一个`Copy`的trait，让**查**这个步骤在执行的时候可以返回这个数据的`Copy`  
　　这是一个权宜之计，实际上不算是个好的处理方式，因为更苛刻的限制会降低泛用性，但这个问题后面再说  
#### 泛型定义的错误
　　这要分两个部分来说  
　　第一点是`impl<T: Ord + Copy> Heap<T>{}`中的两个泛型，我最开始写的其实是`impl Heap<T>`，但是实际上`impl`对于泛型的定义是在`impl`后面完成的，而非结构体名称旁边，在结构体名称旁边写的其实是表示调用了该泛型  
　　而第二点也出在上面那个，由于不熟悉定义规则，当我尝试把`Copy`这个**trait**加上去的时候，我写的其实是`<T: Ord, Copy>`，是该不对报错去查了资料才知道应该用`+`连接，不管怎么说还是记录一下  
#### 数组下标问题
　　既然我的堆的存储是基于完全二叉树实现的，那么二叉树的线性存储和节点编号就会与数组下标关联了，而我过往的数组使用习惯都是静态数组  
　　故而图求省事便利，是以初始下标为1作为假设来写的，但是Rust本身又是一门严苛的编程语言，导致我所习惯的静态数组是不方便的，而动态数组要想把初始下标定义为1，在创建使用和管理维护上都是时间和空间的不便与浪费  
　　为此我最终还是选择了`0-based`写法，这就导致需要对几个编号计算的地方做出调整  
　　假设左字节点，右字节点，父节点分别是:lc, rc, fa．那么lc = fa * 2 + 1; rc = fa * 2 + 2; fa = (lc-1)/2 或 fa = (rc-1)/2  
　　于是又花费了点精力去对计算表达式做了修改  
#### usized的限制
　　总听Rust安全，但是具体的体现的确是在使用的过程中才能体会到的，而`usize`就算是其中之一  
　　在Rust的规定里，下标索引的类型必须是`usize`类型，这也就保证了数组的下标不会出现负数，但是与C的不同在于，C的非负是在对二进制内容的解释上实现的，即无符号位  
　　但是Rust更加稳妥，它严格限制`usize`不能在使用过程中出现负数，即一旦小于0就会`panic`然后终止程序运行，这就直接要求我不能按照以前的逻辑：加一个特判，来解决，而且由于从前从来不使用非负类型，其实我的特判本身也有问题  
　　所以当程序反复异常终止后我才找到了原因：我的程序在运行过程中企图把**0下标**进行减法运算，导致程序异常终止，于是我就在调用相关调整函数的时候就提前判断是非为零  
　　不过其实在`pop`函数中的特判是大于１，因为只有一个节点同样也是不需要调整的，可以算是一个微乎其微的优化  
## 优化版heap
### 总代码:
　　作为初版倒也算差强人意，但是始终有所不足，所以又进一步进行了优化，也让它更加符合Rust的习惯:  
```Rust
struct Heap<T: Ord> {
    heap: Vec<T>,
}

impl<T: Ord> Heap<T> {
    fn sift_up(&mut self, mut idx: usize) {
        while idx > 0 {
            let fa = (idx - 1) >> 1;
            if self.heap[fa] < self.heap[idx] {
                self.heap.swap(fa, idx);
                idx = fa;
            } else {
                break;
            }
        }
    }

    fn sift_down(&mut self, mut idx: usize) {
        let len = self.heap.len();
        loop {
            let mut next: usize = idx;
            let left_idx = (idx << 1) + 1;
            let right_idx = (idx << 1) + 2;
            if left_idx < len && self.heap[idx] < self.heap[left_idx] {
                next = left_idx;
            }
            if right_idx < len && self.heap[next] < self.heap[right_idx] {
                next = right_idx;
            }
            if next != idx {
                self.heap.swap(idx, next);
                idx = next;
            } else {
                break;
            }
        }
    }

    pub fn new() -> Self {
        Self { heap: Vec::new() }
    }

    pub fn from_vec(input: Vec<T>) -> Self {
        let mut heap = Self { heap: input };
        for idx in (0..(heap.heap.len() >> 1)).rev() {
            heap.sift_down(idx);
        }
        heap
    }

    pub fn push(&mut self, num: T) {
        self.heap.push(num);
        self.sift_up(self.heap.len() - 1);
    }

    pub fn peek(&self) -> Option<&T> {
        self.heap.first()
    }

    pub fn pop(&mut self) -> Option<T> {
        if self.heap.is_empty() {
            return None;
        }
        let idx = self.heap.len() - 1;
        self.heap.swap(0, idx);
        let item = self.heap.pop();
        if self.heap.len() > 1 {
            self.sift_down(0);
        }
        item
    }

    pub fn len(&self) -> usize {
        self.heap.len()
    }

    pub fn is_empty(&self) -> bool {
        self.heap.is_empty()
    }
}
```
### 具体优化点
　　这一版的代码在逻辑上进行了一些精简，并提高了一些效率和泛用性：
#### 泛型约束的优化
　　就像上面所说，最初为了防止堆顶数据弹出后就无法使用或者说被引用后无法弹出或是修改，我的解决方案是把泛型的约束加上了一个`Copy`，但是显然降低了泛用性  
　　直到后来我才意识到：我分明可以在弹出数据时就把那个数据传递出来，也就是执行`Move`操作，这样其实算得上两全其美，毕竟虽然这要求我如果想要完全获得堆顶元素，就必须弹出堆顶元素  
　　但是其实无伤大雅，毕竟堆这种数据结构如果想要看其他元素，本来就是要弹出堆顶的，而如果不需要查看其他元素，那引用堆顶其实也足够  
　　所以现在的泛型约束仅有:`<T: Ord>`，这个就的确不能删除了，毕竟无法比较大小的元素本来也就没有利用堆数据结构的意义  
#### 边界检查修正
　　在我初版本的`down`函数中写出了个非常奇怪的东西：  
```Rust
        if self.size >= ((idx << 1) + 2) && self.heap[idx] < self.heap[(idx << 1) + 1] {
            next = (idx << 1) + 1;
        }
        if self.size >= ((idx << 1) + 3) && self.heap[next] < self.heap[(idx << 1) + 2] {
            next = (idx << 1) + 2;
        }
```
　　这是一个历史遗留问题，还是由于惯性思维，我最初的设想是`1-based`函数，所以`size`其实没有包含0下标，因而在下标运算的过程中，边界检查写的是`>=`，而后来转为`0-based`后，并没有把比较符号修改了，而是把size执行了一个-1操作  
　　但是我也说了，当时并没有意识到在Rust中如果`usize`类型的元素执行`0-1`这种操作就会直接报错，于是再后来再一次检查的时候又把`-1`移项到了右边  
　　总之，原来的两个边界检查其实是逻辑混乱的低效且冗余的产物  
　　所以后来就修改为了:
```Rust
            let left_idx = (idx << 1) + 1;
            let right_idx = (idx << 1) + 2;
            if left_idx < len && self.heap[idx] < self.heap[left_idx] {
                next = left_idx;
            }
            if right_idx < len && self.heap[next] < self.heap[right_idx] {
                next = right_idx;
            }
```
　　让逻辑更加清晰而且提高了代码的可读性  
#### 内存管理的修正
　　这其实是一个历史遗留问题，在过去的代码中，我本来就习惯于使用静态数组，因而即使在Rust中半强制地使用了Vec类型，依然是旧习难改  
　　比如那个`size`数据就是一个典型，在`Vec`类型中本来就有一个用于查看大小的关联函数`len()`，而我并没有意识到，这也是我上一版代码没有查询长度的功能的原因，我根本没想起来还有这么个东西  
　　而这主要是造成了一个资源的浪费，并不是最严重的问题  
　　真正的大问题是，出于静态数组遗留的习惯，我并没有把末尾在逻辑上被删除的元素真正地执行删除操作，事实上`Vec`类型自己有`pop`函数，可以弹出末尾元素  
　　这与`push`函数运作的逻辑联合在一起，就会发现其实有很大的问题：`push`函数执行后压入的元素被排在了已删除元素的后方，根本无法被使用！  
#### 接口安全性
```Rust
    pub fn pop(&mut self) -> Option<T> {
        if self.heap.is_empty() {
            return None;
        }
        let idx = self.heap.len() - 1;
        self.heap.swap(0, idx);
        let item = self.heap.pop();
        if self.heap.len() > 1 {
            self.sift_down(0);
        }
        item
    }
```
　　我在第一条说过，现在的`pop`运作逻辑是把弹出的元素返回，但是在Rust中，其实执行这样一个操作并不规范，因而函数的返回类型变为了一个枚举类型`Option`  
　　同时也在开头加入了一个空堆返回`None`的特判，既提高了逻辑的严密性也增强了安全性，同时也更加符合Rust规范  
　　同理的：
```Rust
    pub fn peek(&self) -> Option<&T> {
        self.heap.first()
    }
```
　　首先是把`front`改名为了`peek`，因为这个命名本身就是我记忆错乱的产物，然后对于查询第一个元素的操作从直接下标索引变更为了`Vec`自带的`first`函数，也就自然而然地把获取变为了借用  
　　但是`first`的返回内容是一个`Option`枚举类型，这也印证了刚才说这样的API接口更加符合Rust规范的说法，那么我们就没必要解包了，直接进一步把这个返回内容继续用`peek`返回给调用者就好  
#### 递归转迭代
　　在我初版本的代码中，`up`和`down`都是通过递归实现的，这也符合我的代码习惯，但是事实上，反复调用函数对于时间和空间都是一种压迫，所以在优化版本中，两个部分都改为了迭代实现  
```Rust
    fn sift_up(&mut self, mut idx: usize) {
        while idx > 0 {
            let fa = (idx - 1) >> 1;
            if self.heap[fa] < self.heap[idx] {
                self.heap.swap(fa, idx);
                idx = fa;
            } else {
                break;
            }
        }
    }

    fn sift_down(&mut self, mut idx: usize) {
        let len = self.heap.len();
        loop {
            let mut next: usize = idx;
            let left_idx = (idx << 1) + 1;
            let right_idx = (idx << 1) + 2;
            if left_idx < len && self.heap[idx] < self.heap[left_idx] {
                next = left_idx;
            }
            if right_idx < len && self.heap[next] < self.heap[right_idx] {
                next = right_idx;
            }
            if next != idx {
                self.heap.swap(idx, next);
                idx = next;
            } else {
                break;
            }
        }
    }
```
　　然后就是把两个函数的名字稍作了修改，更加符合堆数据结构的命名习惯  
#### 对于一个地方精简化
　　虽然初版代码中实现了判空的函数，但实际上我并没有想起来`Vec`函数自带判空函数，为此还特地写了一个用`size`判断的方法  
　　当然，基于当时的我并不习惯于直接用函数的返回内容进一步作为接口的返回内容，估计就算用了，我也还是会自己去写个`if`然后自己手动写return true or false  
```Rust
    pub fn is_empty(&self) -> bool {
        self.heap.is_empty()
    }
```
#### 小Tips
　　在C语言中，乘除二的用位运算实现可以提高代码运行效率，但是在Rust中，两者会被编译为相同的内容，效率等价，所以写哪个都无所谓  
## Rust内置的Heap封装
### 函数调用
　　自己手动实现堆的目的有三：
　　　　1. 练习Rust语言  
　　　　2. 复习堆数据结构  
　　　　3. 以备自带内容无法满足需求的情况  
　　但是一般情况下还是更加适宜使用内置的内容，因为其经过了更加激进的优化，当然，也是保证了安全的前提下  
　　Rust的内置函数在大致逻辑，泛型约束，调用方法和我的手动实现是一致的，被封装在了`std::collections::BinaryHeap`模块，而且默认是大根堆  
　　这一点和 `C++` 是类似的  
### 小根堆实现
　　但是在Rust里，小根堆的实现不是写个`Less`  
　　Rust在`std::cmp::Reverse`模块中封装了一个`Reverse`结构体，可以反转内部值的比较逻辑  
　　只要在调用大根堆插入元素的时候，写`.push(Reverse(...))`就好  
## 初版Dijkstra
### 总体代码
　　当时第一次把堆的优化写出来后，脑子里第一个想法就是把`Dijkstra`在Rust实现一次，于是又是很长一段时间的折腾，写了第一版的代码：  
　　~~其实在此之前还有一版，但是一个跑都跑不动的代码还是不贴出来丢人现眼了~~  
```Rust
struct Dijkstra {
    adj: Vec<Vec<(i32, usize)>>,
    pub dist: Vec<i32>,
}

impl Dijkstra {
  pub fn new(n: usize) -> Self {
        const INF: i32 = 0x3f3f3f3f;
        Self {
            adj: vec![Vec::new(); n],
            dist: vec![INF; n],
        }
    }

    pub fn add(&mut self, u: usize, vertex: usize, val: i32) {
        self.adj[u].push((val, vertex));
    }

    pub fn dijkstra(&mut self, start: usize) {
        let mut heap = std::collections::BinaryHeap::<(Reverse<i32>, usize)>::new();
        self.dist[start] = 0;
        heap.push((Reverse(self.dist[start]), start));
        while let Some((Reverse(d), u)) = heap.pop() {
            if d > self.dist[u] {
                continue;
            }
            for &(val, v) in &self.adj[u] {
                if self.dist[v] > val + d {
                    self.dist[v] = val + d;
                    heap.push((Reverse(self.dist[v]), v));
                }
            }
        }
    }
}
```
### 与不可用版本的对比
　　虽说没打算完整贴出来不可用版本的代码，但是对比还是要写一些，警示后人这一块（）  
#### 定义混乱
　　毕竟刚刚学到如何去修改堆的排序规则，于是初次实践就出现了不少的问题：  
##### 类型不匹配
　　把`Reverse(...)`返回的内容类型当成了原来的类型  
　　所以当时写了`BinaryHeap::<i32, usize>::()`，而下面的压入却写了`heap.push((Reverse(val), vertex))`，导致不匹配  
##### 调用与类型用混
　　错误地把调用函数和类型定义混杂在了一起  
　　也就是本来该定义类型为`Reverse<...>`被我写了个`Reverse(...)`，完全就让代码没法跑了．．．  
##### 类型混用，导致结构体定义混乱
　　开始的想法比较质朴，认为堆内数据要用`Reverse`来改，所以就打算把结构体本身的数据类型内容也该为用`Reverse`包裹的状态  
　　于是乎写出了这样的结构体:  
```Rust
struct Dijkstra {
    adj: Vec<Vec<(Revsese<i32>, usize)>>,
    pub dist: Vec<Reverse<i32>>,
}
```
　　虽然说也不是什么大问题,理论上来说，最多时资源浪费和代码冗余，然而既然我是这么定义的，存入数据的时候写的自然就变成了：  
　　`self.adj.push((Reverse(val), vertex))`，到这里依然只是一个冗长＋资源浪费的问题，但是后面我从`adj`里面取出元素后又这样插入了堆：  
　　`heap.push((Reverse(val), v))`，这就导致这个数据其实被**Reverse**了两次，至于后果是什么我也不确定，乐观点可能是又转回来了，但是代码就没跑动，我也不知道结局  
　　而且与此同时，无论堆里面还是`Vec`里面，值元素都是**Reverse结构体**，以至于后面取出来没法相加，这也是代码跑不动的原因之一~~我勒个超级石山代码~~  
　　~~当然，实际上这个问题可以通过多层解包解决，但是我当时是不会~~  
### 解包冗长
　　由于刚刚从C转过来，对于Rust的多层解包不熟悉，循环使用也比较生硬，导致最初仿照着从前的代码写出了一个很冗长的产物：  
```Rust
        while !heap.is_empty() {
            let peek = heap
                .pop()
                .map(|(_, idx)| idx)
                .expect("Can't pop an empty heap!");
```
　　本来的想法是说在堆非空的情况下对堆顶元素弹出并且解包，在同时在解包同时把边权忽略只留下编号  
　　这很符合从前的C语言代码使用习惯，那时候还沾沾自喜地觉得自己的处理很秒（）  
　　直到AI给出了现在这个版本的代码，我大吃一惊：还有这种操作？  
```Rust
        while let Some((Reverse(d), u)) = heap.pop() {
```
　　是的，仅仅用了一行就完成了我上面的所有操作：判断是否为空并对枚举类型`Option`解包，同时对元组解包  
　　甚至完成了我当时没想到的解包：对`Reverse`结构体解包  
　　这样一下子就取出了所需要的元素的初始状态，实在是太优雅了  
### 初始化冗长
　　在我的逻辑中，对结构体内容的初始化就是定义完成后进行resize之类的方式扩容以及赋值，如果是`C++`的话我就会在构造函数里面写这个  
　　所以我就理所当然地写了这样的初始化方式:
```Rust
    pub fn new(n: usize) -> Self {
        let mut new = Self {
            adj: Vec::<Vec<(Reverse(i32), usize)>>::new(),
            dis: Vec::<i32>::new(),
        };
        new.adj.resize(n, Vec::<(Reverse(i32), usize)>::new());
        new.dis.resize(n, 0x3f3f3f3f);
        new
    }
```
　　基本和`C++`的构造方式类似，然而我后来才知道`Vec`的宏就可以做到：
```Rust
  pub fn new(n: usize) -> Self {
        const INF: i32 = 0x3f3f3f3f;
        Self {
            adj: vec![Vec::new(); n],
            dist: vec![INF; n],
        }
    }
```
　　就，简介，大气，优雅  
　　语法糖！小子！  
　　当然，数据类型也可以定义，但是这里就交给类型推断了  
### 简单优化（或许不算优化）
　　这一点就不是Rust特供了，只不过我以前从来没想过可以这么干，还是记录一下  
　　从我个人的使用经验来说，想来是写一个`bool`类型的数组来标记某节点是否使用过，而事实上，`Dijkstra`的核心逻辑就决定了：当一个点被更新过，那么这个点的最短路径就已经被找到了  
　　毕竟`Dijkstra`不处理负边权问题  
　　所以就可以实现一个懒删除：
```Rust
        while let Some((Reverse(d), u)) = heap.pop() {
            if d > self.dist[u] {
                continue;
            }
```
## Dijkstra的简单优化
　　其实上面那个版本已经是很多优化的结果了，因此接下来的优化主要是一些修修补补，就不把总代码贴出来了  
### 数据类型简单修改
　　AI的建议是把`i32`换为`u64`  
　　这一点其实可以视情况而定，但是的确可以选择把`i32`改为`u32`或者`u64`  
    毕竟`Dijkstra`主要还是处理非负边权的问题，所以使用无符号类型的确算是轻微优化  
　　而究竟使用32还是64其实该看数据大小来定毕竟就是`int`和`long long`的区别罢了  
　　当然，还有一个小细节，Rust中已经定义了最大值常量，写`i32::MAX`或者`u64::MAX`之类的就好  
### 名称修改和数据定义
　　我最开始也确实纠结用`Dijkstra`定义一个结构体的名字很怪，但是当时感觉毕竟主要在写`dijkstra`算法，最后也没动  
　　然而实际上这个结构体应该用来存图，而算法该视作一个关联函数，这才是最合理的理解  
　　既然如此，`dist`就不必定义在结构体里面，而是可以在算法里面定义然后返回整个`Vec`就好，其实这主要是为以前没有这样的习惯，不怎么把一个较大的内容返回  
　　但是这样显然是更加合理的操作方式，这样一来既在名称上更加说得过去，调用起来也更加灵活，可以连续对多个点求单源最短路而不必初始化并覆盖上一次的结果  

## 结语
　　总的来说，这是一次不错的实践，我在这一次试验里体会和学习到了不少新东西，获益良多，也对Rust的理解更深入了一点点  
　　其实这一次还涉及到了一些别的优化和实践，但是那些与本次的主题关联不算大，还是在别的地方讨论吧  
