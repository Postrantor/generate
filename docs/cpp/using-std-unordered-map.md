---
tip: translate by baidu@2023-10-25 08:36:16
---
---
metaTitle: "C++ | Using std::unordered_map"
description: "Declaration and Usage, Some Basic Functions"
---

# Using std::unordered_map



std::unordered_map is just an associative container. It works on keys and their maps. Key as the names goes, helps to have uniqueness in the map. While the mapped value is just a content that is associated with the key. The data types of this key and map can be any of the predefined data type or user-defined.

> std:：unordered_map只是一个关联容器。它适用于关键帧及其贴图。名称的关键点有助于在地图中具有唯一性。而映射的值只是与键关联的内容。此键和映射的数据类型可以是任何预定义的数据类型或用户定义的数据类型。



## Declaration and Usage



As already mentioned you can declare an unordered map of any type. Let's have a unordered map named first with string and integer type.

> 如前所述，您可以声明任何类型的无序映射。让我们先用字符串和整数类型命名一个无序映射。

```cpp
unordered_map<string, int> first; //declaration of the map 
first["One"] = 1; // [] operator used to insert the value 
first["Two"] = 2;
first["Three"] = 3;
first["Four"] = 4;
first["Five"] = 5;

pair <string,int> bar = make_pair("Nine", 9); //make a pair of same type
first.insert(bar); //can also use insert to feed the values

```



## Some Basic Functions


```cpp
unordered_map<data_type, data_type> variable_name; //declaration

variable_name[key_value] = mapped_value; //inserting values

variable_name.find(key_value); //returns iterator to the key value

variable_name.begin(); // iterator to the first element

variable_name.end(); // iterator to the last + 1 element

```



#### Remarks



As the name goes, the elements in unordered map are not stored in sorted sequence. They are stored according to their hash values and hence, usage of unordered map has many benefits such as it only takes O(1) to search any item from it. It is also faster than other map containers. It is also visible from the example that it is very easy to implement as the operator ( [] ) helps us to directly access the mapped value.

> 顾名思义，无序映射中的元素不是按排序顺序存储的。它们是根据哈希值存储的，因此，使用无序映射有很多好处，比如只需要O（1）就可以从中搜索任何项目。它也比其他映射容器更快。从示例中还可以看出，由于运算符（[]）帮助我们直接访问映射的值，因此实现起来非常容易。

