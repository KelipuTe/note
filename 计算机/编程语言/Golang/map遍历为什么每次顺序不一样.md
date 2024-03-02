## map遍历为什么每次顺序不一样

go 版本是 go1.20。源码都在 /src/runtime/map.go 里面。

```
func TestFunction(p7t *testing.T) {
	m := map[int]int{}

	m[11] = 11
	m[22] = 22
	m[33] = 33
	m[44] = 44
	m[55] = 55
	m[66] = 66
	m[77] = 77
	m[88] = 88
	m[99] = 99

	for {
		log.Printf("map;")
		for k, v := range m {
			log.Printf("k=%d;v=%d;", k, v)
		}
	}
}
```

测试代码，初始化一个map，然后塞 9 个元素，9 个元素可以让 map 里面有 2 个桶。方便观察程序运行的过程。

```
const (
	bucketCntBits = 3
	bucketCnt     = 1 << bucketCntBits //每个桶最多8个元素
}
```

```
// A header for a Go map.
type hmap struct {
	count     int //map中键值对数量
	B         uint8  //map里有2的B次方个桶
	buckets    unsafe.Pointer // 所有的桶，放在一个数组里面
	其他参数不是本篇问题的重点
}
```

```
//每个桶的结构
type bmap struct {
	tophash [bucketCnt]uint8
}
```

上面的 bmap 结构是静态结构，在编译过程中 runtime.bmap 会拓展成下面这样的结构。
不过这并不是本篇问题的重点。

```
type bmap struct{
    topbits  [8]uint8
    keys     [8]keytype
    values   [8]valuetype
    其他参数不是本篇问题的重点
}
```

每次用 for range 遍历 map 的时候。
一开始会调用 func mapiterinit(t *maptype, h *hmap, it *hiter) 这个方法。
mapiterinit() 会进行一些初始化操作，主要是初始化 it *hiter 这个变量，
然后调用 func mapiternext(it *hiter) 这个方法确定每次遍历的键值对是哪一对。

```
func mapiterinit(t *maptype, h *hmap, it *hiter) {
	......

	//下面这段代码会决定这次遍历从哪个桶的哪个键值对开始
	var r uintptr
	if h.B > 31-bucketCntBits {
		r = uintptr(fastrand64())
	} else {
		r = uintptr(fastrand())
	}
	it.startBucket = r & bucketMask(h.B) //随机数和桶的数量作与运算
	it.offset = uint8(r >> h.B & (bucketCnt - 1)) //随机数右移，然后和桶的长度作与运算

	// iterator state
	it.bucket = it.startBucket
	
	//bucket（也就是 startBucket）决定从哪个桶开始，offset 决定从哪个键值对开始。

	......

	mapiternext(it) //这次遍历第一个返回的键值对
}
```

```
func mapiternext(it *hiter) {
	......
	bucket := it.bucket
	......
	i := it.i
	......

next:
	if b == nil {
		if bucket == it.startBucket && it.wrapped {
			//当前遍历的桶等于这次遍历开始的桶，而且 wrapped 为 true
			//表示遍历一圈又转回来了，map 里的键值对全都遍历完了，遍历结束
			it.key = nil
			it.elem = nil
			return
		}
		......
		bucket++ //当前遍历的桶+1，startBucket 一上来就会 + 1
		if bucket == bucketShift(it.B) {
		    //数组下标是 0 到 2^B-1 ，这里相等说明下标已经溢出了
		    //也就是说已经遍历到数组的最后一个桶了，转回去到第一个桶
			bucket = 0
			it.wrapped = true
		}
		i = 0
	}
	for ; i < bucketCnt; i++ {
		offi := (i + it.offset) & (bucketCnt - 1) //计算本次遍历的键值对在 tophash 中的下标
		if isEmpty(b.tophash[offi]) || b.tophash[offi] == evacuatedEmpty {
		    //tophash 里对应下标的位置非空表示这里有键值对，否则就是没有，直接跳过
			continue
		}
		k := add(unsafe.Pointer(b), dataOffset+uintptr(offi)*uintptr(t.keysize)) 
		//这里看起来像计算内存偏移量，算的应该是键的地址
		if t.indirectkey() {
			k = *((*unsafe.Pointer)(k))
		}
		e := add(unsafe.Pointer(b), dataOffset+bucketCnt*uintptr(t.keysize)+uintptr(offi)*uintptr(t.elemsize))
		//这里算的应该是值的地址
		......
		if (b.tophash[offi] != evacuatedX && b.tophash[offi] != evacuatedY) ||
			!(t.reflexivekey() || t.key.equal(k, k)) {
			//本次遍历要找的键
			it.key = k
			if t.indirectelem() {
				e = *((*unsafe.Pointer)(e))
			}
			it.elem = e //本次遍历要找的值
		} else {
			//这里先跳过，不是本篇问题的重点
		}
		it.bucket = bucket //改 it 的 bucket
		......
		it.i = i + 1 //i 加 1
		......
		return
	}
	b = b.overflow(t) //这个桶遍历完了，我也不知道是不是置空的意思
	i = 0
	goto next //上面改完 b，这里 goto 回去，就会触发上面的 b == nil，开始遍历下一个桶
}
```

然后我们回到上面的测试代码。这玩意每次测试桶都是两个桶没错，但是每个桶里的元素不一样。我这里用了其中某一次测试的数据。

map结构是怎么出来的我没摸清楚。这里是先 debug 遍历一遍，然后用 debug 的数据倒推出 map 里面的桶结构。

**见图：**

图是个示意图，而且没给出来 tophash 的数据，tophash 在 debug 的时候可以看得到数据的。

倒推出 map 里面的桶结构之后，遍历顺序在每次 for range 的一开始调用 mapiterinit() 的时候就可以定下来了。

下面是两次遍历的过程和参数变化。

过程1

**见图：**

遍历结果：

```
2024/02/25 16:48:10 k=99;v=99;
2024/02/25 16:49:31 k=44;v=44;
2024/02/25 16:50:13 k=55;v=55;
2024/02/25 16:50:28 k=66;v=66;
2024/02/25 16:52:01 k=88;v=88;
2024/02/25 16:56:08 k=11;v=11;
2024/02/25 16:56:11 k=22;v=22;
2024/02/25 16:56:12 k=33;v=33;
2024/02/25 16:56:14 k=77;v=77;
```

过程2

**见图：**

遍历结果：

```
2024/02/25 17:01:15 k=11;v=11;
2024/02/25 17:01:19 k=22;v=22;
2024/02/25 17:01:25 k=33;v=33;
2024/02/25 17:01:34 k=77;v=77;
2024/02/25 17:03:02 k=44;v=44;
2024/02/25 17:03:05 k=55;v=55;
2024/02/25 17:03:09 k=66;v=66;
2024/02/25 17:03:12 k=88;v=88;
2024/02/25 17:03:21 k=99;v=99;
```
