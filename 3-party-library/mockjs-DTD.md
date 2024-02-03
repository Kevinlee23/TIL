### mock - DTD

生成随机数据的范式

#### 属性值为 String

`'name|min-max': string` 通过重复 string 生成字符串，重复次数大于 min 小于 max
`'name|count': string` 通过重复 string 生成字符串，重复次数等于 count

#### Number

`'name|+1': number` 属性值自动加 1，初始值为 number
`'name|min-max': number` 生成一个大于等于 min、小于等于 max 的整数，属性值 number 只是用来确定类型
`'name|min-max.dmin-dmax': number` 生成一个浮点数，整数部分大于等于 min、小于等于 max，小数部分保留 dmin 到 dmax 位
`'name|digit1.digit2: number` 生成一个浮点数，整数部分等于 digit1, 小数部分保留 digit2 位

#### Boolean

`'name|1': boolean` 随机生成一个布尔值，结果为 true 和 false 的概率一样
`'name|min-max': value` 结果为 true 的概率为 min / (min+max), 为 false 的概率为 max / (min+max)

#### Object

`'name|count': object` 从属性值 object 中随机选取 count 个属性
`'name|min-max': object` 从属性值 object 中随机选取 min 到 max 个属性

#### Array

`'name|1': array` 从属性值 array 中随机选取 1 个元素，作为最终值
`'name|+1': array` 从属性值 array 中顺序选取 1 个元素，作为最终值
`'name|min-max': array` 通过重复属性值 array 生成一个新数组，重复次数大于等于 min，小于等于 max
`'name|count': array` 通过重复属性值 array 生成一个新数组，重复次数为 count

#### RegExp

`'name': regexp` 通过正则表达式反向生成匹配字符串
