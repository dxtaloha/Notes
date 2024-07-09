## 注意事项

```cpp
//1、引用也是一个指向引用对象的地址，但是它对程序员透明，因此不能把它传递给指针，可以直接像使用对象一样使用它，更方便。因此就把他当作一个普通的对象看待就好，他就是对象的别名，在具体使用时也是这样。
//而指针中也是一个指向对象的地址，因为需要用*来解引用来使用对象。所谓解引用（解析引用中的地址=解析指针中的地址），就是指它是对程序员不透明的引用而已。

//不能把指针传递给引用
int *point; point->void fun_name(int& rename_point){} //错误
//也不能把引用传递给指针
int& point; point->void fun_name(int* rename_point){} //错误
//但是可以把指针解引用传递给引用（不存在引用传递给指针解引用的情况，参数不会存在指针解引用，只存在指针）
int* point; *point->void fun_name(int& rename_point){} //正确
//也可以把指针传递给指针引用，即此时rename_point就是point找个指针的别名
int* point; point->void fun_name(int*& rename_point){} //正确



int& point; point->void fun_name(int rename_point){} //传递形参；
int* point; point->void fun_name(int* rename_point){}//虽然传递的是形参，但是是形参指针，指向的对象是同一个
int point; point->void fun_name(int& rename_point){} //传递实参；
int* point; *point->void fun_name(int& rename_point){} //传递实参；


//2、指针的创建
//创建指针时，要么把一个已经存在的对象赋给它，要么它必须new或者malloc一个对象
//如果赋值的已经存在的对象，且该对象在栈里被创建，那么不需要释放内存；否则其它任何情况都必须要释放内存

//指针这个针是被创建于栈中的（即存地址的这块内存是在栈里，因此不需要关心指针的释放，只需要关系指针指向对象内存的释放）
```



## 1、vector

```cpp
//创建
vector<int> vec_name;
vector<char> vec_name;
vector<string> vec_name;
vector<class> vec_name;
vector<vector<int>> vec_name; //二维创建,一二维都是空
vector<vector<int>> vec_name(a); //一维为a个类型为vector<int>元素，二维量为空

//创建时初始化
vector<class> vec_name(a); //a个class类型值为0的数组
vector<class> vec_name(a, b); //a个class类型值为b的数组
vector<vector<class>> vec_name(a， vector<class>(b)); //a个vector<class>类型的元素，每个vector<class>中有b个class类型的元素，默认全为0
vector<int> vec_name = {0, 1, 2, 3} //赋初值
vector<vector<int>> vec_name = {{0, 1, 2}, {5, 6, 3}, {7, 2, 4}}; //二维数组赋初值

//更改长度
//vector是可以动态更改长度的，但是动态更改可能在运行时比较慢，所以可以更改数组默认长度
//每当数组元素超过当前capacity+1后，就会自动将capacity翻倍
vec_name.resize(a); //将数组重新更改长度为a


//查
vec_name[i];
vec_name.at(i);
vec_name[i][j]; //二维
vec_name.at(i).at(j); //所有的二维数组都是这样调用

vec_name.size(); //返回元素数
vec_name.front(); //返回第一个元素值
vec_name.back(); //返回最后一个元素值
vec_name.begin(); //返回第一个元素的地址
vec_name.end(); //返回最后一个元素之后的地址
vec_name.empty(); //判空，空返回1(true)，非空返回0(false)
vec_name.capacity(); //返回数组当前的最大长度
vec_name.max_size(); //返回每种类型vector的极限capacity


//删
vec_name.clear(); //清空
vec_name.pop_back(); //删除最后一个元素
vec_name.erase(a); //删除下标为a的数
vec_name.erase(vec_name.begin() + k); //删除下标为k的数，所有后续元素前移，返回下一个位置的下标（即为原先k+2位置的数的下标，现在为k+1）
vec_name.ease(a, b); //删除下标a到b的元素，不包括b
vec_name.erase(vec_name.begin() + k, vec_name.end() - l);//删除下标从k到l的元素，不包括下标l



//插
vec_name.push_back(a); //在尾部插入数a
vec_name.insert(a, b); //将b插入到下标a
vec_name.insert(vec_name.begin() + k, a); //将a插入到下标k(从0计数，实际a是第为k+1个元素)，k顺位后移为下标k+1
vec_name.insert(a, b, c); //将c个b依次插入到下标a处
vec_name.insert(vec_name.begin() + k, a, b); //将b个a依次插入到下标k处

```