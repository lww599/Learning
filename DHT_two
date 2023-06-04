package main

import (
	"crypto/rand"
	"crypto/sha1"
	"encoding/base64"
	"encoding/hex"
	"fmt"
	"math/big"
)

// Node 节点类
type Node struct {
	nodeID  string
	buckets [160]*Bucket
	keys    map[string][]byte
}

// Bucket 桶类
type Bucket struct {
	ids []string
}

var nodesMap = make(map[string]*Node)

var SetNodes []string
var GetNodes []string

// compareGetMin 获取两值最小距离
func compareGetMin(targetValue, value1, value2 string) string {
	num := new(big.Int)
	num1 := new(big.Int)
	num2 := new(big.Int)
	num.SetString(targetValue, 2)
	num1.SetString(value1, 2)
	num2.SetString(value2, 2)
	// 计算距离
	result1 := new(big.Int)
	result1.Xor(num, num1)
	result2 := new(big.Int)
	result2.Xor(num, num2)

	if result1.Cmp(result2) < 0 {
		return value1
	} else {
		return value2
	}
}

// GetRandom 生成两个随机数，0~2之间
func GetRandom() (int, int) {
	var nums [2]int
	// 随机生成两个不重复的整数
	for i := range nums {
		num, err := rand.Int(rand.Reader, big.NewInt(3))
		if err != nil {
			// 处理错误
			return -1, -1
		}
		nums[i] = int(num.Int64())
	}
	for nums[0] == nums[1] {
		num, err := rand.Int(rand.Reader, big.NewInt(3))
		if err != nil {
			// 处理错误
			fmt.Println(err)
			return -1, -1
		}
		nums[1] = int(num.Int64())
	}
	return nums[0], nums[1]
}

// InsertNode
/**
 * 插入节点
 * 计算插入的节点距离应该加到哪个桶
 * 1.如果是距离最近的桶(即数组最后一个元素),则查看桶有没有满,桶没满就加入,桶满了就分裂(数组中加入新的桶,新桶中永远保存距离最近的n个节点,分裂前满的桶装距离远的节点)
 * 2.如果是非最近的桶,桶满就放弃加入,桶未满就加入
 */
func (s *Node) InsertNode(nodeId string) bool {
	if s.nodeID == nodeId {
		return false
	}
	new_node := Node{nodeID: nodeId}
	result := findBucket(s.nodeID, nodeId)
	if result < 0 {
		return false
	}
	var index int
	index = result
	bucket := s.buckets[index]
	if bucket == nil {
		s.buckets[index] = new(Bucket)
	}
	insertInto(index, new_node.nodeID, s)
	return true
}

// insertInto 插入节点
func insertInto(index int, newNode string, targetNode *Node) {
	bucket := targetNode.buckets[index]
	for _, v := range bucket.ids {
		if v == newNode {
			return
		}
	}
	// 小于三个就更新
	if len(bucket.ids) < 3 {
		bucket.ids = append(bucket.ids, newNode)
	}
}

// findBucket 查询属于第几个桶
func findBucket(selfId, targetId string) int {
	num1 := new(big.Int)
	num2 := new(big.Int)
	num1.SetString(selfId, 2)
	num2.SetString(targetId, 2)

	result := new(big.Int)
	result.Xor(num1, num2)
	return 160 - len(fmt.Sprintf("%b", result))
}

// isDuplicate 查看是否有相同的元素
func isDuplicate(str string, strs []string) bool {
	// 将二进制字符串转换为大整数类型
	num := new(big.Int)
	num.SetString(str, 2)
	// 判断这个大整数是否已经存在
	for _, str := range strs {
		n := new(big.Int)
		n.SetString(str, 2)
		if n.Cmp(num) == 0 {
			return true
		}
	}
	return false
}

// printBucketContents 打印桶中的id
func (s *Bucket) printBucketContents() {
	for _, v := range s.ids {
		fmt.Printf("nodeID = %s \n", v)
	}
}

// SetValue 插入值
func (s *Node) SetValue(key string, value []byte) bool {
	hash := sha1.Sum(value)
	hashStr := hex.EncodeToString(hash[:])
	if key != hashStr {
		return false
	}
	if s.keys[key] != nil {
		return true
	}
	// 将值存入自己的节点中
	s.keys[key] = value
	// 获取距离最近的桶
	keyInt := new(big.Int)
	str, _ := keyInt.SetString(key, 16)
	keyBinary := fmt.Sprintf("%160b", str)
	result := findBucket(s.nodeID, keyBinary)
	var bucket *Bucket
	bucket = s.buckets[result]
	if bucket == nil {
		return false
	}
	// 判断找到的新的节点是否为最近的节点,是则递归,否则不递归
	index1, index2 := checkLen(len(bucket.ids))
	var flag1, flag2 bool
	if index2 != -1 {
		var maxStr string
		minStr := compareGetMin(keyBinary, bucket.ids[index1], bucket.ids[index2])
		if minStr == bucket.ids[index1] {
			maxStr = bucket.ids[index2]
		} else {
			maxStr = bucket.ids[index1]
		}

		updateIndex := isUpdated(keyBinary, SetNodes, minStr)
		if updateIndex == -1 {
			return false
		} else {
			SetNodes[updateIndex] = minStr
			flag1 = nodesMap[minStr].SetValue(key, value)
		}

		updateIndexMax := isUpdated(keyBinary, SetNodes, maxStr)
		if updateIndexMax != -1 {
			SetNodes[updateIndexMax] = maxStr
			flag2 = nodesMap[maxStr].SetValue(key, value)
		}
		if flag1 || flag2 {
			return true
		} else {
			return false
		}
	} else {
		var flag bool
		updateIndex := isUpdated(keyBinary, SetNodes, bucket.ids[0])
		if updateIndex == -1 {
			return false
		} else {
			SetNodes[updateIndex] = bucket.ids[0]
			flag = nodesMap[bucket.ids[0]].SetValue(key, value)
		}
		if flag {
			return true
		} else {
			return false
		}
	}
}

// GetValue 获取key对应的值
func (s *Node) GetValue(key string) []byte {
	if s.keys[key] != nil {
		hash := sha1.Sum(s.keys[key])
		hashStr := hex.EncodeToString(hash[:])
		if key != hashStr {
			return nil
		}
		return s.keys[key]
	}
	// 获取距离最近的桶
	keyInt := new(big.Int)
	str, _ := keyInt.SetString(key, 16)
	keyBinary := fmt.Sprintf("%160b", str)
	result := findBucket(s.nodeID, keyBinary)
	var bucket *Bucket
	bucket = s.buckets[result]
	if bucket == nil {
		return nil
	}
	// 判断找到的新的节点是否为最近的节点,是则递归,否则不递归
	index1, index2 := checkLen(len(bucket.ids))
	if index2 != -1 {
		var maxStr string
		var value1 []byte
		var value2 []byte
		minStr := compareGetMin(keyBinary, bucket.ids[index1], bucket.ids[index2])
		if minStr == bucket.ids[index1] {
			maxStr = bucket.ids[index2]
		} else {
			maxStr = bucket.ids[index1]
		}
		updateIndex := isUpdated(keyBinary, GetNodes, minStr)
		if updateIndex == -1 {
			return nil
		} else {
			GetNodes[updateIndex] = minStr
			value1 = nodesMap[minStr].GetValue(key)
		}

		updateIndexMax := isUpdated(keyBinary, GetNodes, maxStr)
		if updateIndexMax != -1 {
			GetNodes[updateIndexMax] = maxStr
			value2 = nodesMap[maxStr].GetValue(key)
		}

		if value1 == nil && value2 == nil {
			return nil
		} else if value1 != nil && value2 != nil {
			hash1 := sha1.Sum(value1)
			hash1Str := hex.EncodeToString(hash1[:])
			hash2 := sha1.Sum(value2)
			hash2Str := hex.EncodeToString(hash2[:])
			if key != hash1Str && key != hash2Str {
				return nil
			} else {
				if key != string(hash1[:]) {
					return value2
				}
				return value1
			}
		} else {
			var hashStr string
			var value []byte
			if value1 == nil {
				hash := sha1.Sum(value2)
				hashStr = hex.EncodeToString(hash[:])
				value = value2
			} else {
				hash := sha1.Sum(value1)
				hashStr = hex.EncodeToString(hash[:])
				value = value1
			}
			if key != hashStr {
				return nil
			}
			return value
		}
	} else {
		updateIndex := isUpdated(key, GetNodes, bucket.ids[0])
		if updateIndex == -1 {
			return nil
		}
		GetNodes[updateIndex] = bucket.ids[0]
		value := nodesMap[bucket.ids[0]].GetValue(key)
		if value == nil {
			return nil
		} else {
			hash := sha1.Sum(value)
			hashStr := hex.EncodeToString(hash[:])
			if key != hashStr {
				return nil
			}
			return value
		}
	}
}

// checkLen 检查节点间距离
func checkLen(len int) (int, int) {
	if len > 2 {
		return GetRandom()
	} else if len == 2 {
		return 0, 1
	} else {
		return 0, -1
	}
}

// isUpdated 更新
func isUpdated(targetValue string, nodes []string, compare string) int {
	targetBinary := new(big.Int)
	targetBinary.SetString(targetValue, 2)
	compareBinary := new(big.Int)
	compareBinary.SetString(compare, 2)
	minValue := compareGetMin(targetValue, nodes[0], nodes[1])
	if minValue == nodes[0] {
		maxValueBinary := new(big.Int)
		maxValueBinary.SetString(nodes[1], 2)
		resultMaxValue := new(big.Int)
		resultMaxValue.Xor(targetBinary, maxValueBinary)
		resultCom := new(big.Int)
		resultCom.Xor(targetBinary, compareBinary)
		if resultCom.Cmp(resultMaxValue) < 0 {
			return 1
		}
	} else {
		maxValueBinary := new(big.Int)
		maxValueBinary.SetString(nodes[0], 2)
		resultMaxValue := new(big.Int)
		resultMaxValue.Xor(targetBinary, maxValueBinary)
		resultCom := new(big.Int)
		resultCom.Xor(targetBinary, compareBinary)
		if resultCom.Cmp(resultMaxValue) < 0 {
			return 0
		}
	}
	return -1
}

// inverse 反转
func inverse(value string) string {
	byteArray := []byte(value)
	for i, v := range byteArray {
		if v == '0' {
			byteArray[i] = '1'
		} else {
			byteArray[i] = '0'
		}
	}
	return string(byteArray)
}

// testValue 测试值
func test() {
	// 生成100个节点,并完成网络的构建
	var strs []string
	for len(strs) < 100 {
		max := new(big.Int).Sub(new(big.Int).Lsh(big.NewInt(1), 160), big.NewInt(1))
		// 生成一个160位的随机二进制字符串
		num, _ := rand.Int(rand.Reader, max)
		str := fmt.Sprintf("%0160b", num)
		// 检查这个二进制字符串是否已经存在
		if !isDuplicate(str, strs) {
			strs = append(strs, str)
		}
	}

	var nodes []*Node
	// 初始化
	for _, v := range strs {
		node := Node{nodeID: v}
		nodes = append(nodes, &node)
		nodesMap[v] = &node
		node.keys = make(map[string][]byte)
	}
	for i, v := range nodes {
		for j, k := range strs {
			if i == j {
				continue
			}
			v.InsertNode(k)
		}
	}

	// 生成200个随机字符串,并生成对应的hash
	var randomStrs []string
	var hashes []string
	for i := 0; i < 200; i++ {
		// 随机生成长度为8的字符串
		length := 8
		bytes := make([]byte, length)
		// 从随机源中读取指定长度的随机字节序列
		rand.Read(bytes)
		str := base64.URLEncoding.EncodeToString(bytes)
		randomStrs = append(randomStrs, str)
		hash := sha1.Sum([]byte(str))
		hashStr := hex.EncodeToString(hash[:])
		hashes = append(hashes, hashStr)
	}

	// 寻找一个随机节点
	num, _ := rand.Int(rand.Reader, big.NewInt(100))
	for i, v := range randomStrs {
		hashInverse := inverse(hashes[i])
		if len(SetNodes) == 0 {
			SetNodes = append(SetNodes, hashInverse, hashInverse)
		} else {
			SetNodes[0] = hashInverse
			SetNodes[1] = hashInverse
		}
		nodes[num.Int64()].SetValue(hashes[i], []byte(v))
	}

	// 生成100个随机数
	var nums []int
	var isExist [200]bool
	for len(nums) <= 100 {
		num1, _ := rand.Int(rand.Reader, big.NewInt(200))
		if !isExist[num1.Int64()] {
			nums = append(nums, int(num1.Int64()))
			isExist[num1.Int64()] = true
		}
	}
	for _, v := range nums {
		num2, _ := rand.Int(rand.Reader, big.NewInt(100))
		hashInverse := inverse(hashes[v])
		if len(GetNodes) == 0 {
			GetNodes = append(GetNodes, hashInverse, hashInverse)
		} else {
			GetNodes[0] = hashInverse
			GetNodes[1] = hashInverse
		}

		value := nodesMap[nodes[num2.Int64()].nodeID].GetValue(hashes[v])
		fmt.Println("key: ", hashes[v])
		if value != nil {
			fmt.Println("value: ", string(value))
		} else {
			fmt.Println("can not find value")
		}
		fmt.Println("")
	}
}

func main() {
	nodesMap = make(map[string]*Node)
	test()
}
